---
name: code-review-principles
description: Use when reviewing code for quality. Provides general principles for code review covering reuse, simplicity, maintainability, consistency, duplication, architecture, and overall quality. Language-agnostic.
---

# Code Review Principles

## Principios

### Reutilizacion

- Existe codigo que ya resuelve esto?
- Se estan utilizando utilidades y funciones existentes?

### Simplicidad

- Es la solucion mas simple posible?
- Se puede entender en una lectura?

### Mantenibilidad

- Sera facil de modificar en el futuro?
- Las dependencias estan claras?

### Consistencia

- Sigue los patrones del proyecto?
- Respeta las convenciones de nombre y estilo?

### Duplicacion

- Hay logica duplicada?
- Se puede extraer en una abstraccion comun?

### Arquitectura

- Respeta los limites de los modulos?
- Las dependencias van en la direccion correcta?

### Calidad general

- El manejo de errores es adecuado?
- Los nombres son claros y descriptivos?
- Hay tests para los cambios?
