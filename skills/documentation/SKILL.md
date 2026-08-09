---
name: documentation
description: Use when determining whether documentation needs updating after changes. Covers README, ADR, technical docs, functional docs, and architectural changes. Does not generate unnecessary documentation.
---

# Documentation

## Principio

Solo documentar cuando aporta valor. La documentacion debe reflejar fielmente el estado actual del codigo.

## Cuando actualizar

### Siempre

- Cambios arquitectonicos.
- Nuevos endpoints o APIs.
- Variables de entorno nuevas o modificadas.
- Cambios en requisitos de instalacion o despliegue.
- Breaking changes.

### Cuando corresponda

- README: si cambia el proposito, setup o uso del proyecto.
- ADR: si se toma una decision arquitectonica relevante.
- Documentacion tecnica: si cambia la estructura interna.
- Documentacion funcional: si cambia el comportamiento del usuario.

### Nunca

- Documentar codigo que se explica por si mismo.
- Generar documentacion "por si acaso".
- Duplicar informacion que ya existe en otro lugar.

## Validacion

Antes de finalizar, verificar:

- La documentacion coincide con el codigo actual.
- No hay referencias a funcionalidades eliminadas.
- No hay ejemplos incorrectos.
