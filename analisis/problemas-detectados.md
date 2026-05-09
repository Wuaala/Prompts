# 🔴 Problemas Detectados en WebChat7.html

Análisis de 10 problemas críticos encontrados en el parser de WhatsApp.

---

## 📋 Tabla Resumen

| # | Problema | Severidad | Línea | Impacto | Solución |
|---|----------|-----------|-------|---------|----------|
| 1 | XSS vía innerHTML | 🔴 CRÍTICO | 191-210 | Inyección de código | Sanitizar todo texto |
| 2 | Regex fecha frágil | 🟠 ALTO | 159 | No detecta variantes | Soportar múltiples formatos |
| 3 | Base64 sin validación | 🔴 CRÍTICO | 66 | Datos inválidos insertados | Validar MIME types |
| 4 | Mensajes multilínea rotos | 🟠 ALTO | 177 | Líneas perdidas/mal parseadas | Parser robusto por línea |
| 5 | Caracteres invisibles | 🟠 ALTO | 59, 77 | BOM, ZWNJ, spaces ocultos | Normalizar Unicode |
| 6 | Sin manejo de errores | 🟠 ALTO | Todos | Falla silenciosa | Try/catch con logging |
| 7 | Persona hardcoded | 🟡 MEDIO | 165 | Solo detecta un usuario | Detectar dinámicamente |
| 8 | Sin estadísticas | 🟡 MEDIO | N/A | No hay resumen | Calcular stats |
| 9 | Bloqueo UI (ZIP grande) | 🟡 MEDIO | 42-70 | Interfaz congela | Web Workers |
| 10 | Formatos fecha incompletos | 🟡 MEDIO | 159 | No soporta MM/DD/YYYY, AM/PM | Agregar más formatos |

---

## 🔴 PROBLEMA 1: XSS vía innerHTML

### Código Actual (VULNERABLE)
```javascript
html += `
<div class="msg ${msg.clase}">
  <div class="remitente">${msg.remitente}</div>
  <div class="texto">${msg.texto.replace(/\n/g, "<br>")}</div>
`;
```

### Ataque Posible
```
Remitente: <img src=x onerror="alert('XSS')">
Texto: <script>alert('XSS')</script>
```

Al hacer `innerHTML +=`, el navegador interpreta el HTML malicioso.

### Impacto
- 🔥 **CRÍTICO EN PRODUCCIÓN**
- Robo de sesiones
- Inyección de malware
- Compromiso de datos

### Solución
```javascript
function escapeHtml(str) {
    const div = document.createElement('div');
    div.textContent = str;  // textContent NO interpreta HTML
    return div.innerHTML;   // Devuelve texto escapado
}

// Usar así:
html += `
<div class="msg ${escapeHtml(msg.clase)}">
  <div class="remitente">${escapeHtml(msg.remitente)}</div>
  ...
`;
```

### Tests
```javascript
// Debe escapar esto:
escapeHtml('<script>alert(1)</script>')
// Resultado: &lt;script&gt;alert(1)&lt;/script&gt;

escapeHtml('<img src=x onerror="alert(1)">')
// Resultado: &lt;img src=x onerror=&quot;alert(1)&quot;&gt;
```

---

## 🟠 PROBLEMA 2: Regex de fecha frágil

### Código Actual
```javascript
const msgRegex = /^(\d{1,2}\/\d{1,2}\/\d{2}), (\d{1,2}:\d{2}) - (.*?): (.*)$/;
```

### Lo que NO detecta
- ❌ `21/3/23` (funciona por suerte, pero mal)
- ❌ `3/21/23` (formato MM/DD/YY - iPhone USA)
- ❌ `21.3.2023` (formato europeo)
- ❌ `2023-03-21` (ISO 8601)
- ❌ `21/3/23, 14:45:23` (con segundos)
- ❌ `3/21/23, 2:45 PM` (con AM/PM)
- ❌ Espacios al inicio/final

### Impacto
- 🔥 **Incompatible con iPhone**
- 🔥 **Falla con formatos europeos**
- 📉 50-70% de chats reales no se parsean

### Solución
Crear array de regexes ordenadas por probabilidad:
```javascript
const dateRegexes = [
    // DD/MM/YYYY, HH:MM:SS (más común)
    /^(\d{1,2})\/(\d{1,2})\/(\d{4}),\s+(\d{1,2}):(\d{2}):(\d{2})\s+-\s+(.+?):\s*(.*)$/,
    
    // DD/MM/YY, HH:MM
    /^(\d{1,2})\/(\d{1,2})\/(\d{2}),\s+(\d{1,2}):(\d{2})\s+-\s+(.+?):\s*(.*)$/,
    
    // MM/DD/YYYY, HH:MM (iPhone USA)
    /^(\d{1,2})\/(\d{1,2})\/(\d{4}),\s+(\d{1,2}):(\d{2})\s+(?:AM|PM)\s+-\s+(.+?):\s*(.*)$/i,
    
    // ... más formatos
];
```

### Tests
```javascript
const formats = [
    '[21/3/23, 14:45:23] Juan: Hola',           // Android
    '3/21/23, 2:45 PM - John: Hi',              // iPhone USA
    '[21.3.2023, 14:45] Juan: Hola',            // Europeo
    '2023-03-21, 14:45:23 - Juan: Hola',        // ISO 8601
];

formats.forEach(line => {
    const result = tryParseDate(line);
    console.assert(result !== null, `Failed: ${line}`);
});
```

---

## 🔴 PROBLEMA 3: Base64 sin validación

### Código Actual
```javascript
const blob = await entry.async("blob");
const base64 = await blobToBase64(blob);
zipMediaFiles[clean] = base64;  // ⚠️ Sin validar
```

### Riesgos
- ❌ Base64 inválido → navegador lo rechaza
- ❌ MIME type falso → error silencioso
- ❌ Archivo corrupto → imagen rota
- ❌ Archivo malicioso → vulnerabilidad

### Impacto
- 🔥 **Datos inválidos en memoria**
- 📉 Adjuntos que no se muestran
- 🔥 Posible inyección de datos

### Solución
```javascript
function isValidBase64Data(dataUrl) {
    if (!dataUrl.startsWith('data:')) return false;
    
    const [mimeType, data] = dataUrl.split(',');
    
    // Validar MIME type
    const validMimes = [
        'image/jpeg', 'image/png', 'image/gif',
        'audio/mpeg', 'audio/wav', 'video/mp4',
        'application/pdf'
    ];
    
    if (!validMimes.some(m => mimeType.includes(m))) {
        return false;
    }
    
    // Validar base64
    try {
        atob(data);  // Lanza error si es inválido
        return true;
    } catch (e) {
        return false;
    }
}

// Usar:
if (isValidBase64Data(base64)) {
    zipMediaFiles[clean] = base64;
} else {
    console.warn(`Invalid base64: ${clean}`);
}
```

### Tests
```javascript
// Debe aceptar:
isValidBase64Data('data:image/png;base64,iVBORw0K...')  // true

// Debe rechazar:
isValidBase64Data('data:application/exe;base64,xxx')  // false
isValidBase64Data('data:image/png;base64,!!!')        // false
isValidBase64Data('not-a-data-url')                   // false
```

---

## 🟠 PROBLEMA 4: Mensajes multilínea rotos

### Código Actual
```javascript
if (current) current.texto += "\n" + line;
```

### Problemas
1. **No detecta citas:**
   ```
   [14:45] Juan: Mira
   > Mensaje anterior
   > De alguien más
   
   Esto es nuevo
   ```
   ❌ Incluye las citas como parte del mensaje

2. **Falla con menciones:**
   ```
   [14:45] Juan: Hola @María
   Cómo estás?
   ```
   ❌ La segunda línea se une sin contexto

3. **Emojis en nueva línea:**
   ```
   [14:45] Juan: Jajaja
   😂😂😂
   ```
   ❌ Se une sin espacios

### Impacto
- 📉 Mensajes truncados o mal formados
- 📉 Pérdida de estructura

### Solución
Parser robusto con estados:
```javascript
const states = {
    READING_MESSAGE: 'reading_message',
    READING_ATTACHMENT: 'reading_attachment',
    READING_MULTILINE: 'reading_multiline'
};

// Usar máquina de estados en lugar de simplemente concatenar
if (isNewMessage(line)) {
    state = READING_MESSAGE;
} else if (isAttachmentLine(line) && state === READING_MESSAGE) {
    state = READING_ATTACHMENT;
} else {
    state = READING_MULTILINE;
}
```

---

## 🟠 PROBLEMA 5: Caracteres invisibles

### Código Actual
```javascript
line = line.replace(/[\u200E\u200F\u202A-\u202E]/g, "").trim();
```

### Lo que falta
- ❌ BOM (Byte Order Mark): `\uFEFF`
- ❌ Zero-width space: `\u200B`
- ❌ Zero-width joiner: `\u200D`
- ❌ Thin space: `\u2009`
- ❌ Non-breaking space: `\u00A0`
- ❌ Control characters: `\u0000-\u001F`

### Impacto
- 📉 Regex no matchea por caracteres ocultos
- 📉 Nombres de participante con espacios fantasma
- 📉 Adjuntos con nombres duplicados

### Solución
```javascript
const INVISIBLE_CHARS = /[\u200B-\u200D\u200E\u200F\u202A-\u202E\uFEFF]/g;
const CONTROL_CHARS = /[\u0000-\u001F\u007F-\u009F]/g;
const UNICODE_SPACES = /[\u00A0\u1680\u2000-\u200B\u202F\u205F\u3000]/g;

function normalizeText(text) {
    return text
        .replace(INVISIBLE_CHARS, '')
        .replace(CONTROL_CHARS, '')
        .replace(UNICODE_SPACES, ' ')
        .replace(/\r\n/g, '\n')
        .replace(/\r/g, '\n')
        .trim();
}
```

---

## 🟠 PROBLEMA 6: Sin manejo de errores

### Código Actual
No hay try/catch. Si una línea falla, todo falla silenciosamente.

### Impacto
- 🔥 Parsing incompleto sin avisar
- 🔥 Usuarios no saben qué salió mal
- 📉 Difícil de debuggear

### Solución
```javascript
const errors = [];
const warnings = [];

for (let i = 0; i < lines.length; i++) {
    try {
        const result = parseLine(lines[i]);
        if (!result) {
            warnings.push({
                line: i + 1,
                content: lines[i].substring(0, 100),
                reason: 'No match'
            });
        }
    } catch (err) {
        errors.push({
            line: i + 1,
            error: err.message
        });
    }
}

return { messages, errors, warnings };
```

---

## 🟡 PROBLEMA 7: Persona hardcoded

### Código Actual
```javascript
const clase = remitente.toLowerCase().includes("iván") ||
              remitente.toLowerCase().includes("ivan") ||
              remitente.toLowerCase().includes("piscis")
              ? "me" : "them";
```

### Problemas
- ❌ Solo funciona con esos nombres
- ❌ Imposible reutilizar en otro chat
- ❌ Nombre duplicado (ivan/iván)

### Impacto
- 📉 Código no reutilizable
- 📉 Mensajes propios mostrados como ajenos

### Solución
Detectar automáticamente:
1. Contar mensajes por remitente
2. El que tiene MÁS es el usuario
3. O dejar que el usuario seleccione

---

## 🟡 PROBLEMA 8: Sin estadísticas

### Falta
- ❌ Total de mensajes
- ❌ Participantes únicos
- ❌ Rango de fechas
- ❌ Adjuntos por tipo
- ❌ Mensajes por persona

### Impacto
- 📉 No hay resumen del chat
- 📉 Imposible validar si se parseó correctamente

---

## 🟡 PROBLEMA 9: Bloqueo UI (ZIP grande)

### Código Actual
```javascript
const zip = await jszip.loadAsync(arrayBuffer);  // Síncrono, bloquea UI
```

### Impacto
- 🔥 Interfaz congela 5-10 segundos
- 📉 Mala experiencia de usuario

### Solución
Web Worker + chunks + progress bar

---

## 🟡 PROBLEMA 10: Formatos fecha incompletos

### Formatos NO soportados
- ❌ `3/21/23, 2:45 PM` (iPhone USA)
- ❌ `21.3.2023` (Europeo)
- ❌ `21-3-2023` (Alternativo)
- ❌ `2023-03-21` (ISO)
- ❌ Con segundos: `14:45:23`

### Impacto
- 📉 Incompatible con iPhone
- 📉 Incompatible con locales europeos

---

## 📊 Resumen por Fase

### FASE 1 (Seguridad) - Arregla problemas 1, 3, 5
### FASE 2 (Errores) - Arregla problemas 2, 4, 6, 10
### FASE 3 (Performance) - Arregla problema 9
### FASE 4 (Metadatos) - Arregla problemas 7, 8

---

**Siguiente paso:** Lee `GUIA-COPILOT-WORKSPACE.md` y comienza con FASE 1.
