---
name: conventional-commits
description: Use when generating commit messages. Produces messages following Conventional Commits specification with proper types, scopes, and format.
---

# Conventional Commits

## Formato

```
<type>(<scope>): <subject>

<body>

<footer>
```

## Tipos

- **feat**: Nueva funcionalidad.
- **fix**: Correccion de bug.
- **docs**: Cambios en documentacion.
- **style**: Cambios que no afectan la logica (formato, espacios).
- **refactor**: Cambio que no corrige bugs ni anade features.
- **perf**: Mejora de rendimiento.
- **test**: Anadir o corregir tests.
- **chore**: Cambios en build, CI, dependencias.
- **ci**: Cambios en configuracion de CI/CD.
- **build**: Cambios en sistema de build.

## Reglas

- Subject en imperativo presente.
- Sin mayuscula inicial (excepto nombres propios).
- Sin punto final.
- Maximo 72 caracteres en subject.
- Scope opcional, describe el modulo afectado.
- Body explica el "que" y el "por que", no el "como".
- Footer para breaking changes o referencias a issues.

## Buenas practicas

- Un commit, una responsabilidad.
- No mezclar cambios no relacionados.
- El mensaje debe ser comprensible sin ver el diff.
