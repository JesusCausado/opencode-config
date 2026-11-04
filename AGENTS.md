# Principios Globales

## Filosofia

Ejecucion autonoma estilo Codex/Claude: el usuario concede el consentimiento
al inicio de la sesion y los agentes completan la tarea sin interrupciones
salvo bloqueos reales. Las preguntas funcionales se agrupan en una sola
solicitud cuando son inevitables.

## Principios operativos

- Analiza antes de modificar.
- Comprende el problema antes de implementar.
- Busca reutilizacion antes de crear.
- Respeta la arquitectura definida por el proyecto.
- Manten los cambios pequenos y enfocados.
- No introduzcas complejidad innecesaria.
- No inventes resultados de validaciones.
- Ejecuta las validaciones posibles antes de finalizar.
- Reporta riesgos y bloqueos encontrados.
- No realices commits ni push sin autorizacion explicita.
- Actua con autonomia local: explora, edita, valida y delega sin
  preguntar por preferencias reversibles.
- Manten visibles el plan y los todos durante la ejecucion.
- Solo pide confirmacion para acciones destructivas, secretos,
  produccion o efectos externos no autorizados.
- Cuando una validacion externa o fisica no sea posible, declaralo en el
  cierre; no inventes resultados.

## Pipeline canonico

```text
Plan -> Build -> especialistas -> Security -> Review -> Verify -> Commit
                          \_____________ remediacion ____________/
```

- **Plan** es el unico punto donde se pregunta al usuario; Build no puede.
- **Build** orquesta la implementacion y los gates.
- Los subagentes ejecutan y reportan; nunca preguntan al usuario.
- **Verify** es obligatorio y precede a **Commit**.
- **Commit** solo se invoca con autorizacion explicita del usuario.

## Permisos y autonomia

- Bash permite todo por defecto; lo destructivo, el historial de git,
  la publicacion, la administracion, los efectos de produccion y los
  wrappers de interprete estan denegados explicitamente.
- Se prefiere `deny` sobre `ask`: un deny no bloquea la sesion y el
  agente busca otra via. Ningun agente del pipeline se queda esperando.
- Build corre sin supervision (`question: deny`): decide lo reversible,
  registra sus decisiones y deja los bloqueos en el Cierre.
- Los subagentes (frontend, backend, database, security, review, verify,
  explore, scout) tienen `question: deny` y reportan bloqueos a Build.

## Gates de calidad

- **Verify** es obligatorio y siempre corre al final del pipeline.
- **Review** es obligatorio cuando la Historia lo marca o cuando el
  cambio toca logica critica, APIs publicas o multiples archivos.
- **Security** es obligatorio cuando hay auth, secretos, PII,
  criptografia, uploads, integraciones OAuth/JWT, nuevas dependencias
  o exposicion de APIs.
- Un `RECHAZADO` o `CAMBIOS_REQUERIDOS` de Review/Verify devuelve el
  trabajo a Build para remediacion; maximo 3 ciclos antes de declarar
  `BLOQUEADO`.
- **Commit** no se invoca sin `Verify: APROBADO` y sin autorizacion
  explicita del usuario.
