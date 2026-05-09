# 🏗️ Estructura Modular Propuesta

Redesign del código actual en módulos independientes, reutilizables y mantenibles.

---

## 📊 Árbol de Módulos

```
whatsapp-parser/
├── js/
│   ├── core/
│   │   ├── chat-parser.js          ← Motor principal de parsing
│   │   └── message-builder.js      ← Constructor de objetos mensaje
│   │
│   ├── security/
│   │   ├── sanitizer.js            ← escapeHtml, escapar atributos
│   │   ├── base64-validator.js     ← Validación de MIME y base64
│   │   └── file-validator.js       ← Validación de archivos
│   │
│   ├── normalization/
│   │   ├── text-normalizer.js      ← BOM, CRLF, espacios invisibles
│   │   ├── date-normalizer.js      ← Convertir a ISO 8601
│   │   └── emoji-normalizer.js     ← Manejar ZWJ sequences
│   │
│   ├── io/
│   │   ├── zip-loader.js           ← Cargar y extraer ZIP
│   │   ├── zip-validator.js        ← Validar ZIP (tamaño, estructura)
│   │   └── file-processor.js       ← Procesar archivos individuales
│   │
│   ├── performance/
│   │   ├── chunked-processor.js    ← Procesamiento por chunks
│   │   ├── chat.worker.js          ← Web Worker
│   │   └── progress-tracker.js     ← Tracking de progreso
│   │
│   ├── metadata/
│   │   ├── stats-calculator.js     ← Calcular estadísticas
│   │   ├── participant-detector.js ← Detectar participantes
│   │   └── attachment-analyzer.js  ← Analizar adjuntos
│   │
│   ├── ui/
│   │   ├── renderer.js             ← Renderizar mensajes (seguro)
│   │   ├── error-reporter.js       ← Reportes de error/warning
│   │   └── virtual-scroll.js       ← Scroll virtual
│   │
│   └── index.js                    ← Punto de entrada, orquestación
│
├── tests/
│   ├── fixtures/
│   │   └── whatsapp-samples.js     ← Muestras reales
│   ├── security.test.js
│   ├── parser.test.js
│   ├── normalization.test.js
│   └── performance.test.js
│
└── docs/
    ├── API.md
    ├── SECURITY.md
    └── CHANGELOG.md
```

---

## 🔄 Flujo de Datos

```
┌─────────────┐
│  ZIP Input  │
└──────┬──────┘
       │
       ▼
  ┌─────────────────────┐
  │ ZIP Validator       │ ← Validar tamaño, estructura, corrupción
  │ - zip-validator.js  │
  └──────┬──────────────┘
         │
         ▼
  ┌──────────────────────┐
  │ File Processor       │ ← Extraer archivos, validar MIME
  │ - file-processor.js  │
  └──────┬───────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ Text Normalizer          │ ← Eliminar BOM, CRLF, invisibles
  │ - text-normalizer.js     │
  └──────┬───────────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ Chat Parser              │ ← Parsear líneas, detectar estructura
  │ - chat-parser.js         │
  └──────┬───────────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ Message Builder          │ ← Construir objetos mensaje
  │ - message-builder.js     │
  └──────┬───────────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ Stats Calculator         │ ← Calcular métricas
  │ - stats-calculator.js    │
  └──────┬───────────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ Secure Renderer          │ ← Escapar HTML, sanitizar
  │ - renderer.js            │
  └──────┬───────────────────┘
         │
         ▼
  ┌──────────────────────────┐
  │ HTML Output              │
  └──────────────────────────┘
```

---

## 📦 Módulos Detallados

### 1. **core/chat-parser.js**
**Responsabilidad:** Parsear líneas de texto y detectar estructura

**Métodos:**
```javascript
class ChatParser {
    parse(text)                    // Parse completo, retorna array de mensajes
    tryParseLine(line, lineNum)    // Parse de una línea
    isAttachmentLine(line)         // Detecta si es adjunto
    getDateRegexes()               // Array de regex por formato
}
```

**Entradas:** Texto crudo del chat  
**Salidas:** Array de objetos {fecha, hora, remitente, texto, adjuntos}  
**Errores:** Array de errores por línea

---

### 2. **security/sanitizer.js**
**Responsabilidad:** Escapar y sanitizar texto para HTML

**Métodos:**
```javascript
function escapeHtml(str)                  // Escapa < > " ' &
function sanitizeAttribute(attr, value)   // Valida atributos (src, href)
function sanitizeFilename(filename)       // Limpia nombres de archivo
function sanitizeMessageText(text)        // Sanitiza contenido de mensaje
```

---

### 3. **normalization/text-normalizer.js**
**Responsabilidad:** Normalizar caracteres especiales

**Métodos:**
```javascript
function normalizeInvisibleChars(text)    // Elimina BOM, ZWNJ, etc
function normalizeLineEndings(text)       // CRLF → LF
function normalizeUnicode(text)           // NFC normalization
function removeControlChars(text)         // Elimina \u0000-\u001F
```

---

### 4. **io/zip-validator.js**
**Responsabilidad:** Validar integridad y seguridad del ZIP

**Métodos:**
```javascript
async validateZipFile(file)              // Validación completa
getMimeTypeFromSignature(bytes)          // Magic bytes
detectZipBomb(entries)                   // Detecta Zip of Death
validatePath(path)                       // Previene path traversal
```

---

### 5. **ui/renderer.js**
**Responsabilidad:** Renderizar HTML de forma segura

**Métodos:**
```javascript
renderMessage(msg)                       // Renderiza un mensaje
renderAdjunto(file)                      // Renderiza un adjunto
renderDaySeparator(date)                 // Separador de día
renderStats(stats)                       // Panel de estadísticas
```

---

## 🔗 Dependencias Entre Módulos

```
text-normalizer.js
    ↓
chat-parser.js
    ↓
message-builder.js  ←  sanitizer.js
    ↓                       ↓
stats-calculator.js    base64-validator.js
    ↓                       ↓
renderer.js  ←────────────────┘
```

**Regla:** Cada módulo solo conoce de los que están abajo en la cadena.

---

## ✅ Criterios de Modularización

✔️ **Baja acoplamiento:** Cada módulo es independiente  
✔️ **Alta cohesión:** Funciones relacionadas en el mismo módulo  
✔️ **Responsabilidad única:** Un módulo = un propósito  
✔️ **Reutilizable:** Puede usarse en otros proyectos  
✔️ **Testeable:** Fácil de escribir tests  
✔️ **Documentado:** Comentarios y tipos claros  

---

## 📚 Patrones de Diseño

### 1. **Chain of Responsibility**
Varios validadores (ZIP, MIME, tamaño) en cadena.

### 2. **Strategy**
Varios regex para diferentes formatos de fecha.

### 3. **Builder**
Construir objetos mensaje paso a paso.

### 4. **Observer**
Progress tracker notifica al UI.

### 5. **Worker Thread**
Web Worker para operaciones pesadas.

---

## 🧪 Testing

Cada módulo tiene tests independientes:

```javascript
// tests/parser.test.js
describe('ChatParser', () => {
    it('should parse Android format', () => { ... });
    it('should handle multiline messages', () => { ... });
});

// tests/security.test.js
describe('Sanitizer', () => {
    it('should escape XSS payload', () => { ... });
    it('should validate base64', () => { ... });
});
```

---

## 🚀 Ventajas de esta Arquitectura

✅ **Fácil de mantener:** Cambios aislados a un módulo  
✅ **Fácil de testear:** Tests independientes  
✅ **Fácil de escalar:** Agregar nuevas features sin tocar código existente  
✅ **Fácil de reutilizar:** Módulos en otros proyectos  
✅ **Fácil de debuggear:** Errores aislados a un módulo  
✅ **Fácil de colaborar:** Múltiples devs sin conflictos  

---

**Última actualización:** 2026-05-09
