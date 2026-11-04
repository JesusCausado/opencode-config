---
name: implementation-history
description: Use when validating an Implementation History before or after development. Verifies objective, scope, criteria, gates, dependencies, and completion status.
---

# Implementation History

## Validacion ANTES del desarrollo

Verificar que la Historia de Implementacion contiene:

- **Objetivo**: Que se busca lograr, claro y conciso.
- **Alcance**: Que incluye y que NO incluye.
- **Criterios de aceptacion**: Condiciones medibles de completitud.
- **Gates**: Security, Review y Verify obligatorios u opcionales con
  motivo. Verify siempre obligatorio.
- **Dependencias**: Modulos, servicios o cambios que deben existir.
- **Riesgos**: Problemas potenciales identificados.

Si falta alguno de estos elementos, la Historia NO esta lista para
desarrollarse.

## Plantilla minima

```markdown
# Historia de Implementacion: <titulo>

## Objetivo
<1-3 lineas>

## Alcance
- Incluye: <lista breve>
- No incluye: <lista breve o "Ninguna">

## Criterios de aceptacion
- [ ] <criterio medible>
- [ ] <criterio medible>

## Gates
- security: <obligatorio|opcional> | motivo: <...|N/A>
- review: <obligatorio|opcional> | motivo: <...|N/A>
- verify: obligatorio

## Dependencias
- <modulo, servicio o cambio previo; "Ninguna" si aplica>

## Riesgos
- <riesgo> | mitigacion: <opcion>

## Estrategia
<2-5 lineas>

## Handoff a Build
- Archivos candidatos: <ruta> (certeza: alta|media|baja)
- Validacion esperada: <comandos y criterios>
- Restricciones: <no negociables>
- Decisiones pendientes: <lista vacia si no hay>
```

## Clasificacion y gates

Toda Historia debe declarar su clasificacion:

- **Dominios afectados**: frontend, backend, database, ninguno o varios.
- **Criticidad**: baja, media, alta.
- **Security requerido**: si/no + motivo (auth, secretos, PII, deps, red).
- **Review requerido**: si para logica critica o APIs publicas; opcional
  para cambios pequenos que cubra `verify`.
- **Verify requerido**: siempre.

## Validacion DESPUES del desarrollo

Verificar:

- **Criterios cumplidos**: Cada criterio de aceptacion tiene evidencia.
- **Gates cumplidos**: Security/Review/Verify con veredicto y evidencia.
- **Validaciones ejecutadas**: Compilacion, tests, lint, etc.
- **Pendientes**: Trabajo no completado documentado.
- **Riesgos materializados**: Problemas encontrados y como se resolvieron.

## Bucle de remediacion

Si Review o Verify devuelven `CAMBIOS_REQUERIDOS` o `RECHAZADO`:

1. Build corrige los hallazgos.
2. Re-ejecuta Review si la correccion toco logica critica.
3. Re-ejecuta Verify.
4. Maximo 3 ciclos; el cuarto fallo consecutivo declara `BLOQUEADO`.

## Resultado

- Si todo es correcto: Historia cerrada, lista para commit.
- Si hay pendientes: Documentar claramente que falta y por que.
