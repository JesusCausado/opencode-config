---
name: implementation-history
description: Use when validating an Implementation History before or after development. Verifies objective, scope, criteria, dependencies, and completion status.
---

# Implementation History

## Validacion ANTES del desarrollo

Verificar que la Historia de Implementacion contiene:

- **Objetivo**: Que se busca lograr, claro y conciso.
- **Alcance**: Que incluye y que NO incluye.
- **Criterios de aceptacion**: Condiciones medibles de completitud.
- **Dependencias**: Modulos, servicios o cambios que deben existir.
- **Riesgos**: Problemas potenciales identificados.

Si falta alguno de estos elementos, la Historia NO esta lista para desarrollarse.

## Validacion DESPUES del desarrollo

Verificar:

- **Criterios cumplidos**: Cada criterio de aceptacion tiene evidencia.
- **Validaciones ejecutadas**: Compilacion, tests, lint, etc.
- **Pendientes**: Trabajo no completado documentado.
- **Riesgos materializados**: Problemas encontrados y como se resolvieron.

## Resultado

- Si todo es correcto: Historia cerrada, lista para commit.
- Si hay pendientes: Documentar claramente que falta y por que.
