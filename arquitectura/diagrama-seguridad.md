# 🔒 Diagrama: Capas de Seguridad

Visualización de las capas de defensa contra vulnerabilidades.

---

## 🛡️ Modelo de Defensa en Profundidad

```
┌───────────────────────────────────────────────────────────────┐
│                    ATACANTE                                  │
│         (Intenta inyectar código malicioso)                  │
└────────────────────┬────────────────────────────────────────┘
                     │
     ┌───────────────┼───────────────┐
     │ Diferentes Vectores de Ataque │
     └───────────────┼───────────────┘
                     │
        ┌────────────┼────────────┐
        ▼            ▼            ▼
   ┌────────┐  ┌─────────┐  ┌──────────┐
   │ ZIP    │  │ HTML    │  │ URL      │
   │ Bomb   │  │ XSS     │  │ Injection│
   └────────┘  └─────────┘  └──────────┘
        │            │            │
        └────────────┼────────────┘
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║                  CAPA 1: INPUT VALIDATION                    ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ ZIP signature verification (magic bytes)                  ║
║  ✓ File size limits (500MB ZIP, 100MB/file)                 ║
║  ✓ Compression ratio check (ZIP bomb detection)             ║
║  ✓ Path traversal prevention                                ║
║  ✓ UTF-8 encoding validation                                ║
╚═══════════════════════════════════════════════════════════════╝
                     │ (Files extracted)
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║              CAPA 2: NORMALIZATION & PARSING                 ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ Remove BOM (U+FEFF)                                       ║
║  ✓ Remove invisible characters (ZWNJ, ZWS, etc)             ║
║  ✓ Normalize line endings (CRLF → LF)                       ║
║  ✓ NFC Unicode normalization                                ║
║  ✓ Multi-regex parsing (multiple date formats)             ║
║  ✓ Line-by-line error catching                              ║
║  ✓ Continue on parse failures                               ║
╚═══════════════════════════════════════════════════════════════╝
                     │ (Parsed messages)
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║           CAPA 3: FILE & MIME VALIDATION                     ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ Magic byte verification (actual file type)               ║
║  ✓ MIME type whitelist                                       ║
║  ✓ Extension validation                                       ║
║  ✓ Base64 format validation                                   ║
║  ✓ File size re-check                                        ║
║  ✓ Duplicate file detection                                  ║
╚═══════════════════════════════════════════════════════════════╝
                     │ (Validated files)
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║              CAPA 4: CONTENT SANITIZATION                    ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ HTML entity escaping (<, >, ", ', &)                     ║
║  ✓ Filename sanitization                                      ║
║  ✓ Attribute validation (src, href, etc)                    ║
║  ✓ Protocol validation (javascript:, data:, etc)            ║
║  ✓ Length limits on text fields                              ║
║  ✓ Whitelist of safe HTML tags                               ║
╚═══════════════════════════════════════════════════════════════╝
                     │ (Sanitized content)
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║               CAPA 5: SECURE RENDERING                       ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ Use textContent instead of innerHTML                      ║
║  ✓ Element creation with safe attributes                     ║
║  ✓ CSP headers (Content-Security-Policy)                    ║
║  ✓ No inline JavaScript                                      ║
║  ✓ No eval() or Function()                                  ║
║  ✓ rel="noopener noreferrer" on links                      ║
╚═══════════════════════════════════════════════════════════════╝
                     │ (Safe HTML)
                     ▼

╔═══════════════════════════════════════════════════════════════╗
║                 CAPA 6: ERROR HANDLING                        ║
╠═══════════════════════════════════════════════════════════════╣
║  ✓ Try/catch on all critical operations                      ║
║  ✓ Graceful degradation                                      ║
║  ✓ Error logging without exposing internals                  ║
║  ✓ User-friendly error messages                              ║
║  ✓ Never expose stack traces                                 ║
║  ✓ Rate limiting (if deployed online)                        ║
╚═══════════════════════════════════════════════════════════════╝
                     │
                     ▼
              ┌─────────────┐
              │ Safe Output │
              │   (HTML)    │
              └─────────────┘
```

---

## 🔍 Matriz de Amenazas vs Defensas

```
┌────────────────────┬──────────────┬──────────────┬──────────────┐
│ Amenaza            │ Capa 1       │ Capa 3-4     │ Capa 5       │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ XSS via HTML       │              │ Sanitize     │ Escape HTML  │
│ injection          │              │ filename     │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ ZIP Bomb           │ Size check   │              │              │
│                    │ Ratio check  │              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Malicious file     │ Signature    │ MIME check   │ Validate     │
│ upload             │ verify       │ Extension    │ attributes   │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Path Traversal     │ Path check   │              │              │
│ (../)              │ Sanitize     │              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Corrupt ZIP        │ Signature    │              │ Error handle │
│                    │ Structure    │              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ JavaScript URL     │              │ Protocol     │ Validate src │
│ (javascript:)      │              │ check        │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Data URL attack    │              │ MIME list    │ Whitelist    │
│ (data:text/html)   │              │              │              │
├────────────────────┼──────────────┼──────────────┼──────────────┤
│ Invalid UTF-8      │ Encoding     │              │              │
│                    │ check        │              │              │
└────────────────────┴──────────────┴──────────────┴──────────────┘
```

---

## 📋 Checklist de Validación

### Input Validation (CAPA 1)
```
☐ ZIP magic bytes: 50 4B 03 04
☐ File size: ≤ 500MB
☐ Total uncompressed: ≤ 1GB
☐ Compression ratio: ≥ 10% (detect bomb)
☐ Path traversal: reject ../ or ..\
☐ UTF-8 encoding: valid sequences
```

### Sanitization (CAPA 4)
```
☐ HTML entities: & < > " '
☐ Control chars: \u0000-\u001F
☐ Invisible chars: ZWNJ, ZWS, BOM
☐ Filename length: ≤ 255 chars
☐ Attribute protocols: reject javascript:, vbscript:
☐ Data URLs: only image/* MIME types
```

### Rendering (CAPA 5)
```
☐ Use textContent, not innerHTML
☐ CSP headers set
☐ No inline <script> tags
☐ Links have rel="noopener noreferrer"
☐ Form submissions disabled
☐ Event handlers not from user input
```

---

## 🎯 Ejemplos de Ataques Bloqueados

### Ataque 1: XSS Básico
```
Input:  <script>alert('XSS')</script>
Capa 4: Escapa a &lt;script&gt;alert('XSS')&lt;/script&gt;
Capa 5: Renderiza como texto, no ejecuta
Result: ✓ BLOQUEADO
```

### Ataque 2: XSS Atributo
```
Input:  " onerror="alert('XSS')"
Capa 4: Escapa a \" onerror=\"alert('XSS')\"
Capa 5: Atributo valor validado
Result: ✓ BLOQUEADO
```

### Ataque 3: ZIP Bomb
```
Input:  1MB ZIP → 5GB descomprimido
Capa 1: Ratio 0.02% << 10% threshold
        Rechaza: "Posible ZIP bomb"
Result: ✓ BLOQUEADO
```

### Ataque 4: Malicious File
```
Input:  photo.jpg (actual: malware.exe)
Capa 1: Magic bytes no coinciden
Capa 3: MIME type no es image/jpeg
Result: ✓ BLOQUEADO
```

### Ataque 5: Path Traversal
```
Input:  "../../etc/passwd"
Capa 1: Path starts with .. → rechaza
Result: ✓ BLOQUEADO
```

---

## 🔄 Flujo de Error

```
┌──────────────────┐
│ Threat Detected  │
└────────┬─────────┘
         │
         ▼
    ┌─────────┐
    │ Severity│
    └────┬────┘
         │
    ┌────┴──────┐
    │            │
    ▼ CRITICAL   ▼ WARNING
┌────────┐  ┌──────────┐
│ Reject │  │ Log Only │
│ & Stop │  │ Continue │
└────────┘  └──────────┘
```

---

## 📊 Security Test Coverage

```
┌─────────────────────────────────┐
│ Unit Tests (Capa por capa)      │
├─────────────────────────────────┤
│ ✓ Input validation tests        │
│ ✓ Sanitization tests            │
│ ✓ XSS payload tests             │
│ ✓ MIME type tests               │
│ ✓ Path traversal tests          │
│ ✓ Error handling tests          │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Integration Tests               │
├─────────────────────────────────┤
│ ✓ Full pipeline tests           │
│ ✓ Real WhatsApp samples         │
│ ✓ Edge cases                    │
│ ✓ Error recovery                │
└─────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────┐
│ Security Audit (OWASP)          │
├─────────────────────────────────┤
│ ✓ Manual code review            │
│ ✓ Vulnerability scanning        │
│ ✓ Pen testing                   │
│ ✓ User feedback                 │
└─────────────────────────────────┘
```

---

**Última actualización:** 2026-05-09  
**Estado:** Arquitectura definida  
**Próximo paso:** Implementar capas (FASE-1)
