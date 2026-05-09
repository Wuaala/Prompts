# 🔄 Diagrama: Flujo de Parsing

Visualización del flujo de datos desde ZIP hasta HTML renderizado.

---

## 📊 Flujo Principal

```
┌─────────────────────────────────────────────────────────────┐
│                   ZIP File Input                            │
└─────────────┬───────────────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────────────────────────────┐
│              ZIP Validation                                 │
│  ✓ Size check (500MB max)                                  │
│  ✓ Structure validation                                    │
│  ✓ Corruption detection                                    │
│  ✓ Compression ratio check (ZIP bomb detection)            │
└─────────────┬───────────────────────────────────────────────┘
              │
         ❌ INVALID                                             │
         ├──► Error Report ✗                           │ VALID
         │                                               │
         └◄──────────────────────────────────────────┐  │
                                                       │  ▼
                              ┌──────────────────────────────────────┐
                              │      Extract Files from ZIP          │
                              │  ✓ Extract .txt chat                 │
                              │  ✓ Extract media files               │
                              │  ✓ Convert to base64                 │
                              └──────────────┬───────────────────────┘
                                             │
                                    ❌ ERROR │ ✓ OK
                                    ├────┐   │
                                    │    │   ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Text Normalization       │
                                    │    │  │  ✓ Remove BOM             │
                                    │    │  │  ✓ Remove invisible chars │
                                    │    │  │  ✓ Normalize line endings │
                                    │    │  │  ✓ NFC normalization      │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    │               ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │   Chat Parsing            │
                                    │    │  │  Per-line processing:      │
                                    │    │  │  • Detect message line     │
                                    │    │  │  • Extract: date, time,    │
                                    │    │  │             sender, text   │
                                    │    │  │  • Detect attachments      │
                                    │    │  │  • Handle multiline        │
                                    │    │  │  • Log errors/warnings     │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                            PARSE ERROR  │    ❌ LINE ERROR (skip, log)
                                    │    │    ├──► Add to error list
                                    │    │    │     Continue parsing
                                    │    │    │
                                    │    │    ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Message Object           │
                                    │    │  │  {                         │
                                    │    │  │   date, time,              │
                                    │    │  │   sender, text,            │
                                    │    │  │   attachments[]            │
                                    │    │  │  }                         │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    │               ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Sanitization             │
                                    │    │  │  ✓ Escape HTML entities    │
                                    │    │  │  ✓ Sanitize filename       │
                                    │    │  │  ✓ Validate base64 MIME    │
                                    │    │  │  ✓ Check file sizes        │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    │    ❌ INVALID │ ✓ VALID
                                    │    │    ├──► Skip file
                                    │    │    │     Log warning
                                    │    │    │
                                    │    │    ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Statistics Calculation   │
                                    │    │  │  • Message count           │
                                    │    │  │  • Participants            │
                                    │    │  │  • Date range              │
                                    │    │  │  • Attachment breakdown    │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    │               ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Rendering (Secure)       │
                                    │    │  │  ✓ Escape all text output  │
                                    │    │  │  ✓ Validate attributes     │
                                    │    │  │  ✓ Build HTML safely       │
                                    │    │  │  ✓ Group by date           │
                                    │    │  │  ✓ Add media with checks   │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    │               ▼
                                    │    │  ┌────────────────────────────┐
                                    │    │  │  Error/Warning Report     │
                                    │    │  │  ✓ Parse success rate      │
                                    │    │  │  ✓ List of errors by line  │
                                    │    │  │  ✓ Skipped files           │
                                    │    │  │  ✓ Warnings                │
                                    │    │  └────────────┬──────────────┘
                                    │    │               │
                                    │    └───────────────┤
                                    │                    │
                                    │                    ▼
                                    │   ┌────────────────────────────────┐
                                    │   │   Final HTML Output            │
                                    │   │  ┌──────────────────────────┐   │
                                    │   │  │ Error/Warning Panel      │   │
                                    │   │  │ (if errors)              │   │
                                    │   │  └──────────────────────────┘   │
                                    │   │  ┌──────────────────────────┐   │
                                    │   │  │ Statistics Panel         │   │
                                    │   │  └──────────────────────────┘   │
                                    │   │  ┌──────────────────────────┐   │
                                    │   │  │ Messages                 │   │
                                    │   │  │ - Grouped by date        │   │
                                    │   │  │ - Styled messages        │   │
                                    │   │  │ - Media embedded         │   │
                                    │   │  └──────────────────────────┘   │
                                    │   └────────────────────────────────┘
                                    │
                                    └──► Done ✓
```

---

## 🔴 Puntos Críticos de Error

```
┌─ ZIP Loading
│  └─ File corrupted
│     └─ Log error, return empty
│
├─ Text Extraction
│  └─ Invalid UTF-8
│     └─ Log warning, continue
│
├─ Line Parsing
│  └─ Regex no match
│     ├─ Orphan line → log warning
│     └─ Try next regex
│
├─ Attachment Detection
│  └─ Invalid base64
│     └─ Skip file, log warning
│
└─ HTML Rendering
   └─ XSS payload detected
      └─ Escape and render safe
```

---

## ⚙️ Validación de Entrada en Cada Paso

```
┌──────────────────────────────┐
│ ZIP Input                    │
│ • Is file?                   │
│ • Is valid ZIP signature?    │
│ • < 500MB?                   │
└──────────────────────────────┘
         │
         ▼
┌──────────────────────────────┐
│ Text Extraction              │
│ • Valid UTF-8?               │
│ • Has .txt file?             │
│ • Not empty?                 │
└──────────────────────────────┘
         │
         ▼
┌──────────────────────────────┐
│ Line Parsing                 │
│ • Valid date format?         │
│ • Valid sender name?         │
│ • Valid text content?        │
└──────────────────────────────┘
         │
         ▼
┌──────────────────────────────┐
│ Attachment Processing        │
│ • Is known extension?        │
│ • Valid MIME type?           │
│ • Valid base64?              │
│ • < 100MB?                   │
└──────────────────────────────┘
         │
         ▼
┌──────────────────────────────┐
│ HTML Generation              │
│ • XSS sanitized?             │
│ • Attributes escaped?        │
│ • No javascript: URLs?       │
└──────────────────────────────┘
```

---

## 📊 Ejemplo: Parsing de Línea

```
Input: "[21/3/23, 14:45:23] Juan: Hola mundo"

┌────────────────────────────────────────┐
│ Step 1: Normalize                      │
│ "[21/3/23, 14:45:23] Juan: Hola mundo" │
│ (remove BOM, invisible chars)           │
└────────────────────────────────────────┘
             ▼
┌────────────────────────────────────────┐
│ Step 2: Regex Match                    │
│ Try regex #1: DD/MM/YYYY format        │
│ ✓ MATCH                                │
│ Groups: [21, 3, 23, 14, 45, 23,        │
│          Juan, Hola mundo]             │
└────────────────────────────────────────┘
             ▼
┌────────────────────────────────────────┐
│ Step 3: Validate Fields                │
│ • Date 21/3/23 → valid? ✓              │
│ • Time 14:45:23 → valid? ✓             │
│ • Sender "Juan" → valid? ✓             │
│ • Text "Hola mundo" → valid? ✓         │
└────────────────────────────────────────┘
             ▼
┌────────────────────────────────────────┐
│ Step 4: Normalize Date                 │
│ 21/3/23 → 2023-03-21                  │
│ (convert to ISO 8601)                  │
└────────────────────────────────────────┘
             ▼
┌────────────────────────────────────────┐
│ Step 5: Sanitize                       │
│ • Escape HTML in sender: Juan          │
│ • Escape HTML in text: Hola mundo      │
│ • Check for XSS: none found ✓          │
└────────────────────────────────────────┘
             ▼
┌────────────────────────────────────────┐
│ Message Object                         │
│ {                                      │
│   dateStr: "2023-03-21",               │
│   time: "14:45:23",                    │
│   remitente: "Juan",                   │
│   texto: "Hola mundo",                 │
│   adjuntos: [],                        │
│   lineNumber: 1                        │
│ }                                      │
└────────────────────────────────────────┘
```

---

## 🔐 Flujo de Seguridad

```
┌─ Input Validation
│  ├─ ZIP signature check
│  ├─ File size limits
│  └─ UTF-8 validation
│
├─ Processing Validation
│  ├─ Regex matching
│  ├─ Date validation
│  ├─ MIME type checking
│  └─ Base64 validation
│
├─ Output Sanitization
│  ├─ HTML entity escaping
│  ├─ Attribute validation
│  ├─ XSS payload detection
│  └─ URL scheme validation
│
└─ Error Handling
   ├─ Try/catch on critical operations
   ├─ Error logging
   ├─ Continue on non-fatal errors
   └─ Report summary to user
```

---

## ⏱️ Timeline Estimado

```
┌─────────────────────────────────────┐
│ ZIP Loading & Validation: 0.5s      │
├─────────────────────────────────────┤
│ Text Extraction: 0.1s               │
├─────────────────────────────────────┤
│ Normalization: 0.2s                 │
├─────────────────────────────────────┤
│ Parsing (10k lines): 1.5s           │
├─────────────────────────────────────┤
│ Sanitization: 0.3s                  │
├─────────────────────────────────────┤
│ Stats Calculation: 0.2s             │
├─────────────────────────────────────┤
│ HTML Rendering: 0.2s                │
├─────────────────────────────────────┤
│ Total: ~3.0s (goal: <2s)            │
└─────────────────────────────────────┘

⚡ Optimization targets:
  • Use Web Worker for parsing
  • Chunked processing
  • Virtual scrolling for rendering
  • Lazy-load attachments
```

---

**Última actualización:** 2026-05-09
