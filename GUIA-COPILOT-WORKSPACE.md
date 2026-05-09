# 📖 Guía: Cómo usar Wuaala-Prompts con Copilot Workspace

## ¿Qué es GitHub Copilot Workspace?

Copilot Workspace es una interfaz donde puedes:
- 💬 Conversar con Copilot sobre tu código
- 🔄 Hacer que genere cambios en ramas nuevas
- ✅ Revisar, aceptar o rechazar cambios
- 🚀 Hacer commit directamente desde la interfaz

## 🚀 Flujo Paso a Paso

### Paso 1: Abre Copilot Workspace

1. Ve a tu repositorio en GitHub
2. Presiona `.` (punto) en el teclado
3. O ve a `https://github.com/codespaces/new` y selecciona el repo

### Paso 2: Abre este repositorio de prompts

En la terminal:
```bash
git clone https://github.com/Wuaala/prompts.git
cd prompts
```

Verás la estructura:
```
prompts/
├── analisis/
├── arquitectura/
├── prompts/
└── README.md
```

### Paso 3: Lee el análisis

Antes de cualquier cambio, lee:

```bash
# Para entender qué está roto
cat analisis/problemas-detectados.md

# Para entender las vulnerabilidades
cat analisis/seguridad-analysis.md

# Para ver la estrategia completa
cat arquitectura/roadmap-v1.0.md
```

### Paso 4: Copia un prompt

Ejemplo con FASE 1 (Seguridad):

```bash
cat prompts/FASE-1-seguridad-xss.md
```

Copia todo el contenido.

### Paso 5: Pega en Copilot Chat

1. En Copilot Workspace, abre el Chat (icono de chat en la izquierda)
2. Pega el prompt completo
3. Copilot analizará y te dirá qué va a hacer

### Paso 6: Revisa el plan

Copilot te mostrará:
- ✅ Archivos que va a crear
- ✅ Cambios que va a hacer
- ✅ Commit message que va a usar

### Paso 7: Acepta o ajusta

- Puedes decir "más detalles en X"
- Puedes decir "también agrega Y"
- Puedes rechazar cambios

### Paso 8: Genera el código

Copilot creará los archivos. Verás:
- Rama nueva creada
- Cambios listados
- Preview de los archivos

### Paso 9: Haz commit

```bash
git add .
git commit -m "[message sugerido]"
git push
```

O deja que Copilot haga commit automáticamente.

---

## 📋 Estructura de los Prompts

Cada prompt contiene:

```markdown
# FASE X: [Nombre]

## 🎯 Objetivo
Qué implementar

## 📊 Análisis
Por qué se necesita

## 🔧 Tareas
Qué crear exactamente

## 📝 Ejemplo de código
Referencia (opcional)

## ✅ Tests
Cómo validar

## 💾 Commit Message
Mensaje sugerido
```

---

## 🎯 Flujo Recomendado

### Semana 1: FASE 1 - Seguridad

```
1. Lee analisis/seguridad-analysis.md (15 min)
2. Copia prompts/FASE-1-seguridad-xss.md
3. Pega en Copilot Chat
4. Revisa el plan (5 min)
5. Acepta cambios
6. Tests y validación (30 min)
7. Git push
```

### Semana 2: FASE 2 - Manejo de Errores

```
1. Lee analisis/problemas-detectados.md (15 min)
2. Copia prompts/FASE-2-manejo-errores.md
3. Pega en Copilot Chat
4. ... (mismo flujo)
```

### Semana 3: FASE 3 - Performance

```
1. Lee arquitectura/diagrama-flujo-parsing.md (15 min)
2. Copia prompts/FASE-3-performance.md
3. Pega en Copilot Chat
4. ... (mismo flujo)
```

### Semana 4: FASE 4 - Metadatos

```
1. Lee arquitectura/estructura-modular.md (15 min)
2. Copia prompts/FASE-4-metadatos.md
3. Pega en Copilot Chat
4. ... (mismo flujo)
```

---

## 💡 Consejos Prácticos

### Si quieres ajustar un prompt:

```
"En lugar de X, hazlo así: Y"
```

Copilot regenerará el código.

### Si quieres que agregue algo:

```
"Además, agrega Z con los siguientes requisitos: ..."
```

### Si quieres crear una rama nueva:

```
git checkout -b feature/nombre-del-feature
```

Copilot automáticamente crea branches como `copilot-*`.

### Para ver el diff:

```bash
git diff main
```

### Para hacer squash de commits:

```bash
git rebase -i main
```

---

## 🔍 Troubleshooting

### "Copilot no genera código"

1. Verifica que estés en Copilot Workspace (no en Codespaces normal)
2. Cierra y abre el chat nuevamente
3. Copia el prompt completo (sin el # del título)

### "El código generado no es lo que esperaba"

1. Rechaza el cambio
2. Pide a Copilot "Más detalle en la estructura"
3. Regenera

### "Git push falla"

```bash
# Verifica tu token
git remote -v

# O usa HTTPS
git remote set-url origin https://github.com/Wuaala/prompts.git
```

---

## 📚 Referencias

- [GitHub Copilot Workspace Docs](https://github.com/features/copilot/workspace)
- [GitHub CLI](https://cli.github.com/)
- [Git Workflow](https://guides.github.com/introduction/flow/)

---

## ✅ Checklist Antes de Empezar

- [ ] Tengo acceso a GitHub Copilot
- [ ] Tengo Copilot Workspace habilitado
- [ ] He clonado este repositorio
- [ ] He leído `README.md`
- [ ] Entiendo la estructura de carpetas

---

**¡Listo para empezar! 🚀**

Comienza con: `cat analisis/problemas-detectados.md`
