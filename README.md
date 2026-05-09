# 📚 Wuaala-Prompts

Repositorio centralizado de **prompts, análisis y arquitectura** para proyectos de desarrollo usando GitHub Copilot Workspace.

## 📁 Estructura del Repositorio

```
prompts/
├── README.md                          (Este archivo)
├── .gitignore
├── prompts/                           (Prompts reutilizables por fase)
│   ├── FASE-1-seguridad-xss.md
│   ├── FASE-2-manejo-errores.md
│   ├── FASE-3-performance.md
│   ├── FASE-4-metadatos.md
│   └── README.md                      (Guía de uso)
├── analisis/                          (Análisis completos del proyecto)
│   ├── parser-analysis.md             (Análisis del parser actual)
│   ├── zip-handling-analysis.md       (Análisis de manejo de ZIP)
│   ├── ui-rendering-analysis.md       (Análisis del rendering seguro)
│   ├── seguridad-analysis.md          (Análisis de vulnerabilidades)
│   ├── problemas-detectados.md        (Listado de 10 problemas críticos)
│   └── README.md                      (Índice de análisis)
├── arquitectura/                      (Diseño y estructura)
│   ├── estructura-modular.md          (Propuesta de modularización)
│   ├── roadmap-v1.0.md               (Plan comercial completo)
│   ├── diagrama-flujo-parsing.md      (Flujo de parsing)
│   ├── diagrama-seguridad.md          (Capas de seguridad)
│   ├��─ commits-sugeridos.md           (Listado de commits por fase)
│   └── README.md                      (Índice de arquitectura)
└── GUIA-COPILOT-WORKSPACE.md          (Cómo usar estos recursos)
```

## 🚀 Uso Rápido con Copilot Workspace

### Opción 1: Usar un prompt de fase completa

1. Abre [GitHub Copilot Workspace](https://github.com/copilot/workspace)
2. Clona este repositorio o carga el prompt desde `prompts/FASE-X-*.md`
3. Pega el contenido del prompt en Copilot Workspace
4. Copilot generará el código en una rama nueva

**Ejemplo:**
```bash
# Copia el contenido de prompts/FASE-1-seguridad-xss.md
# Pégalo en Copilot Workspace
# Copilot creará una rama con el código de seguridad
```

### Opción 2: Usar análisis para entender el proyecto

1. Lee `analisis/problemas-detectados.md` para ver qué está roto
2. Consulta `analisis/seguridad-analysis.md` para entender vulnerabilidades
3. Usa `arquitectura/estructura-modular.md` como referencia

### Opción 3: Seguir el roadmap paso a paso

1. Abre `arquitectura/roadmap-v1.0.md`
2. Sigue cada fase en orden
3. Usa los prompts correspondientes de `prompts/`

## 📋 Contenido por Carpeta

### `prompts/`
Prompts listos para copiar-pegar en Copilot Workspace. Cada uno es **independiente** y contiene:
- Descripción clara de qué implementar
- Análisis de problemas específicos
- Código a generar
- Tests a escribir
- Commit message sugerido

### `analisis/`
Análisis profundos **sin código**, solo documentación:
- Problemas detectados en el código actual
- Explicación de vulnerabilidades
- Impacto de cada problema
- Estrategia de solución

### `arquitectura/`
Diseño y planificación:
- Estructura modular propuesta
- Flujos de datos
- Capas de seguridad
- Roadmap de desarrollo
- Commit messages sugeridos

## 🎯 Flujo de Trabajo Recomendado

### Semana 1: FASE 1 (Seguridad)
```
1. Lee: analisis/seguridad-analysis.md
2. Usa: prompts/FASE-1-seguridad-xss.md
3. Genera: Código de sanitización
4. Tests: Incluidos en el prompt
```

### Semana 2: FASE 2 (Errores)
```
1. Lee: analisis/problemas-detectados.md
2. Usa: prompts/FASE-2-manejo-errores.md
3. Genera: Parser robusto
4. Tests: Incluidos en el prompt
```

### Semana 3: FASE 3 (Performance)
```
1. Lee: arquitectura/diagrama-flujo-parsing.md
2. Usa: prompts/FASE-3-performance.md
3. Genera: Web Workers + chunking
4. Tests: Incluidos en el prompt
```

### Semana 4: FASE 4 (Metadatos)
```
1. Lee: arquitectura/estructura-modular.md
2. Usa: prompts/FASE-4-metadatos.md
3. Genera: Estadísticas + UI
4. Tests: Incluidos en el prompt
```

## 🔐 Seguridad

Este repositorio **no contiene**:
- Claves privadas
- Datos sensibles
- Código vulnerable

Contiene **solo documentación y prompts seguros**.

## 📞 Soporte

¿Necesitas ayuda?
1. Consulta `GUIA-COPILOT-WORKSPACE.md`
2. Revisa el `README.md` de cada carpeta
3. Copia un prompt y ajustalo a tus necesidades

## 📝 Licencia

Este repositorio es **abierto y reutilizable** para cualquier proyecto.

---

**Última actualización:** 2026-05-09  
**Versión:** 1.0  
**Estado:** Estructura lista para implementación
