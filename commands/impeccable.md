---
description: "Impeccable: /impeccable <comando> <objetivo>. Ej: /impeccable audit pagina de precios. Sin argumentos lista los comandos."
agent: build
---

Usa la skill `impeccable` con esta solicitud: $ARGUMENTS

1. Si `impeccable` no esta en `<available_skills>`, detente y reporta que
   hay que instalarla en el proyecto (`npx impeccable install`).
2. Si no hay argumentos, carga la skill y lista sus comandos agrupados con
   una linea de proposito cada uno. No modifiques nada.
3. Comandos de evaluacion (`critique`, `audit`, `doctor`): delega a
   `frontend` indicando que es solo lectura; devuelve los hallazgos con
   `archivo:linea` y severidad. No edites.
4. Comandos que modifican (`polish`, `bolder`, `quieter`, `distill`,
   `harden`, `typeset`, `layout`, `colorize`, `animate`, `adapt`,
   `optimize`, `clarify`, `delight`, `onboard`, `overdrive`, `shape`,
   etc.): sigue tu pipeline normal. Delega a `frontend` indicando el
   comando de impeccable a aplicar y el objetivo; luego review/verify.
5. `init` / `teach`: normalmente entrevista al usuario, pero en este
   pipeline nadie puede preguntar. Si `PRODUCT.md` ya existe, solo revisa
   huecos y reportalos. Si no existe, delega a `frontend` para escribirlo
   con lo que se pueda inferir del proyecto, marcando cada dato no
   confirmado como `[PENDIENTE: pregunta]`, y lista esas preguntas en el
   Cierre. Nunca inventes usuarios, posicionamiento ni marca.
6. `document`, `extract`: generan `DESIGN.md` o tokens a partir del codigo
   existente; delega a `frontend` y reporta los archivos creados.
