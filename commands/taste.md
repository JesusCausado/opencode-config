---
description: "Taste: construye o reestiliza UI con design-taste-frontend (anti-AI-slop). Ej: /taste hero de la landing"
agent: build
---

Aplica la skill `design-taste-frontend` a: $ARGUMENTS

1. Si `design-taste-frontend` no esta en `<available_skills>`, detente y
   reporta que hay que instalarla
   (`npx skills add https://github.com/Leonxlnx/taste-skill --skill design-taste-frontend`).
2. Delega a `frontend` indicando que cargue `design-taste-frontend` e
   `impeccable` (si existe) antes de editar, respetando la marca y el
   `DESIGN.md` del proyecto por encima de las skills.
3. Sigue tu pipeline normal: validacion, review, verify y Cierre.
