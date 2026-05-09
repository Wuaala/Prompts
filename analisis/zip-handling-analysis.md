# 📦 Análisis: Manejo de ZIP y Carga de Archivos

Análisis profundo del módulo de carga y procesamiento de archivos ZIP.

---

## 🎯 Resumen

**Riesgo:** 🟠 ALTO + 🟡 MEDIO  
**Áreas:** 5 (Validación, Tamaño, Corrupción, Path traversal, Duplicados)  
**Líneas afectadas:** 42-70  

---

## 🔴 PROBLEMA 1: Validación MIME tipo real

### Situación actual
El código confía en la extensión de archivo:
```javascript
const isMedia =
    lower.endsWith(".jpg") || lower.endsWith(".jpeg") ||
    lower.endsWith(".png") || ...
```

### Riesgo
```
1. Archivo: "photo.jpg"
   Contenido real: ejecutable .exe
   → Se carga como base64 sin validar

2. Archivo: "document.pdf"
   Contenido real: malware ZIP
   → Se procesa como media confiable
```

### Solución: zip-validator.js

```javascript
// Firmas de archivos (magic bytes)
const FILE_SIGNATURES = {
    jpg: [0xFF, 0xD8, 0xFF],
    png: [0x89, 0x50, 0x4E, 0x47],
    gif: [0x47, 0x49, 0x46],
    mp4: [0x00, 0x00, 0x00, 0x20, 0x66, 0x74, 0x79, 0x70],
    pdf: [0x25, 0x50, 0x44, 0x46],
    zip: [0x50, 0x4B, 0x03, 0x04],
};

function getMimeTypeFromSignature(bytes) {
    // Convertir primeros bytes a hex
    const signature = Array.from(bytes.slice(0, 8))
        .map(b => b.toString(16).padStart(2, '0'))
        .join('');
    
    // Comparar con firmas conocidas
    for (const [type, sig] of Object.entries(FILE_SIGNATURES)) {
        const sigHex = sig.map(b => b.toString(16).padStart(2, '0')).join('');
        if (signature.startsWith(sigHex)) {
            return type;
        }
    }
    
    return null;
}

function validateFileContent(blob, expectedType) {
    return new Promise((resolve) => {
        const reader = new FileReader();
        
        reader.onload = (event) => {
            const array = new Uint8Array(event.target.result);
            const detectedType = getMimeTypeFromSignature(array);
            
            // El tipo detectado debe coincidir con lo esperado
            const isValid = detectedType === expectedType ||
                           (expectedType === 'jpeg' && detectedType === 'jpg');
            
            resolve(isValid);
        };
        
        reader.readAsArrayBuffer(blob.slice(0, 16)); // Solo primeros 16 bytes
    });
}
```

### Test
```javascript
// Mock blob con firma de PNG
const pngBytes = new Uint8Array([0x89, 0x50, 0x4E, 0x47, 0x0D, 0x0A, 0x1A, 0x0A]);
const pngBlob = new Blob([pngBytes]);

const isValid = await validateFileContent(pngBlob, 'png');
console.assert(isValid === true, 'Valid PNG rejected');

// Mock blob con firma falsa
const fakeBlob = new Blob([new Uint8Array([0xFF, 0xFF, 0xFF])]);
const isFake = await validateFileContent(fakeBlob, 'jpg');
console.assert(isFake === false, 'Invalid signature accepted');
```

---

## 🟠 PROBLEMA 2: Límite de tamaño sin validar

### Situación actual
```javascript
// Sin límites establecidos
const arrayBuffer = await file.arrayBuffer();
const zip = await jszip.loadAsync(arrayBuffer);
```

### Riesgos

**ZIP Bomb (Zip of Death):**
```
1. Archivo: 1MB comprimido
2. Contenido: 1GB descomprimido
3. Resultado: Memoria agotada, UI congela
```

**Archivo gigante:**
```
1. Usuario sube ZIP de 5GB
2. Navegador intenta cargar todo en RAM
3. Crash de la pestaña
```

### Solución

```javascript
const LIMITS = {
    MAX_ZIP_SIZE: 500 * 1024 * 1024,           // 500MB
    MAX_FILE_SIZE: 100 * 1024 * 1024,          // 100MB por archivo
    MAX_UNCOMPRESSED_SIZE: 1000 * 1024 * 1024, // 1GB total descomprimido
    COMPRESSION_RATIO_THRESHOLD: 0.1,          // Alerta si ratio < 10%
};

class ZipValidator {
    static async validateZipFile(file) {
        const errors = [];
        
        // Validar tamaño comprimido
        if (file.size > LIMITS.MAX_ZIP_SIZE) {
            errors.push(`ZIP demasiado grande: ${this.formatSize(file.size)} (máx: ${this.formatSize(LIMITS.MAX_ZIP_SIZE)})`);
            return { valid: false, errors };
        }
        
        // Cargar y validar estructura
        const jszip = new JSZip();
        try {
            const arrayBuffer = await file.arrayBuffer();
            const zip = await jszip.loadAsync(arrayBuffer);
            
            let totalUncompressed = 0;
            
            for (const path in zip.files) {
                const entry = zip.files[path];
                
                // Validar tamaño individual
                if (entry._data.uncompressedSize > LIMITS.MAX_FILE_SIZE) {
                    errors.push(`Archivo ${path} demasiado grande: ${this.formatSize(entry._data.uncompressedSize)}`);
                }
                
                // Detectar ZIP bomb por ratio de compresión
                const compressedSize = entry._data.compressedSize;
                const uncompressedSize = entry._data.uncompressedSize;
                const ratio = compressedSize / uncompressedSize;
                
                if (ratio < LIMITS.COMPRESSION_RATIO_THRESHOLD && uncompressedSize > 10 * 1024 * 1024) {
                    errors.push(`⚠️ Posible ZIP bomb en ${path}: ratio ${(ratio * 100).toFixed(1)}%`);
                }
                
                totalUncompressed += uncompressedSize;
            }
            
            // Validar tamaño total descomprimido
            if (totalUncompressed > LIMITS.MAX_UNCOMPRESSED_SIZE) {
                errors.push(`Contenido descomprimido demasiado grande: ${this.formatSize(totalUncompressed)}`);
            }
            
        } catch (err) {
            errors.push(`ZIP corrupto o inválido: ${err.message}`);
        }
        
        return { valid: errors.length === 0, errors };
    }
    
    static formatSize(bytes) {
        const units = ['B', 'KB', 'MB', 'GB'];
        let size = bytes;
        let unitIndex = 0;
        
        while (size >= 1024 && unitIndex < units.length - 1) {
            size /= 1024;
            unitIndex++;
        }
        
        return `${size.toFixed(2)} ${units[unitIndex]}`;
    }
}
```

---

## 🟠 PROBLEMA 3: Carga de ZIP sin detectar corrupción

### Situación actual
No hay manejo de errores si el ZIP es inválido:
```javascript
const zip = await jszip.loadAsync(arrayBuffer); // Si falla, todo falla
```

### Solución

```javascript
async loadZipSafely(file) {
    const jszip = new JSZip();
    
    try {
        const arrayBuffer = await file.arrayBuffer();
        const zip = await jszip.loadAsync(arrayBuffer);
        return { success: true, zip };
    } catch (error) {
        return {
            success: false,
            error: {
                type: 'INVALID_ZIP',
                message: 'El archivo ZIP no es válido o está corrupto',
                details: error.message
            }
        };
    }
}
```

---

## 🟠 PROBLEMA 4: Path Traversal Attack

### Riesgo
```
Archivo en ZIP: "../../../etc/passwd"
O: "..\..\windows\system32\config\sam"

Al procesar sin validar, se podrían acceder rutas fuera del ZIP.
```

### Solución

```javascript
function sanitizePath(path) {
    // Resolver path relativo
    const normalized = path.replace(/\\/g, '/');
    
    // Rechazar path traversal
    if (normalized.startsWith('..') || 
        normalized.includes('/../') ||
        normalized.includes('..\\')) {
        return null;
    }
    
    // Rechazar rutas absolutas
    if (normalized.startsWith('/')) {
        return normalized.substring(1);
    }
    
    return normalized;
}

// En el loop de archivos:
for (const path in zip.files) {
    const safePath = sanitizePath(path);
    
    if (!safePath) {
        console.warn(`Path traversal attempt blocked: ${path}`);
        continue;
    }
    
    // Procesar con safePath
}
```

---

## 🟡 PROBLEMA 5: Detección de duplicados

### Situación actual
Si hay 2 archivos con el mismo nombre (diferente case o carpeta), se sobrescribe:
```
IMG_001.jpg (desde carpeta1/)
img_001.jpg (desde carpeta2/)

→ Se sobrescribe uno al otro
```

### Solución

```javascript
class ZipProcessor {
    constructor() {
        this.fileMap = new Map();
        this.duplicates = [];
    }
    
    trackFile(filename, blob) {
        const key = filename.toLowerCase();
        
        if (this.fileMap.has(key)) {
            this.duplicates.push({
                filename,
                original: this.fileMap.get(key),
                action: 'skipped'
            });
            return false; // No procesar
        }
        
        this.fileMap.set(key, filename);
        return true; // Procesar
    }
    
    getDuplicateReport() {
        return this.duplicates;
    }
}
```

---

## 📊 Performance: Bloqueo de UI

### Problema
```javascript
const zip = await jszip.loadAsync(arrayBuffer); // Síncrono, bloquea UI
```

### Solución: Web Worker
Ver `FASE-3-performance.md`

---

## ✅ Checklist de Validación ZIP

- [ ] Validar MIME type con magic bytes
- [ ] Limitar tamaño ZIP (500MB máx)
- [ ] Limitar archivo individual (100MB máx)
- [ ] Detectar ZIP bombs (ratio compresión)
- [ ] Validar estructura ZIP
- [ ] Detectar path traversal
- [ ] Registrar errores sin fallar
- [ ] Detectar duplicados
- [ ] Usar Web Worker para grandes archivos
- [ ] Mostrar progreso al usuario

---

## 📝 Implementación: Fase 3

Este análisis se implementa en **FASE-3-performance.md** con el módulo `zip-validator.js`

---

**Última actualización:** 2026-05-09
