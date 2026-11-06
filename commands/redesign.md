---
description: "Redesign: audita y mejora una UI existente sin romper funcionalidad. Ej: /redesign pagina de productos"
agent: build
---

Rediseña sin romper funcionalidad: $ARGUMENTS

`redesign-existing-projects` esta retirada upstream; este flujo la
reemplaza con `impeccable` + `design-taste-frontend`.

1. Diagnostico: delega a `frontend` en modo solo lectura para cargar
   `impeccable` y ejecutar `critique` y `audit` sobre el objetivo. Guarda
   los hallazgos priorizados.
2. Rediseño: delega a `frontend` para corregir los hallazgos cargando
   `design-taste-frontend` e `impeccable`, respetando la marca, el
   `DESIGN.md` del proyecto, el contenido, las rutas y el comportamiento
   existentes.
3. Sigue tu pipeline normal: validacion, review, verify y Cierre. En
   `Cambios` relaciona cada cambio con el hallazgo que resuelve.

Si alguna de las dos skills no esta disponible, continua con la que haya y
dejalo en `Limitaciones`; si no hay ninguna, detente y reportalo.
