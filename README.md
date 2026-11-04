# Entorno Global de OpenCode

## Filosofia

Entorno basado en agentes primarios y subagentes con responsabilidades
claras. El usuario concede el consentimiento al inicio de la sesion y el
pipeline ejecuta autonomamente hasta el cierre, salvo bloqueos reales.

El entorno prioriza autonomia local estilo Codex/Claude: `opencode --auto`
(o activar **Enable auto-approve permissions** en la TUI) aprueba todos
los `ask` durante la sesion. Los `deny` explicitos permanecen bloqueados.

---

## Estructura

```
~/.config/opencode/
  AGENTS.md                  Principios permanentes y pipeline canonico
  opencode.jsonc             Config global + instrucciones
  agents/                    Agentes primarios y especialistas
    plan.md                  Analisis -> Historia de Implementacion
    build.md                 Implementador autonomo + orquestador de gates
    commit.md                Cierre del ciclo (solo tras Verify APROBADO)
    explore.md               Subagente: exploracion rapida del codebase
    scout.md                 Subagente: investigacion externa
    backend.md               Especialista backend
    frontend.md              Especialista frontend
    database.md              Especialista base de datos
    security.md              Especialista seguridad
    review.md                Especialista revision
    verify.md                Subagente: verificacion independiente
  skills/                    Conocimiento reutilizable
    conventional-commits/    Formato de mensajes commit
    documentation/           Criterios para actualizar docs
    implementation-history/  Validacion de historias
    code-review-principles/  Principios de revision
```

---

## Pipeline canonico

```
Plan  ->  Build  ->  especialistas  ->  Security  ->  Review  ->  Verify  ->  Commit
                              \____________ remediacion ___________/
```

- **Plan** es el unico agente donde se agrupan preguntas funcionales.
- **Build** orquesta la implementacion y los gates.
- Los subagentes ejecutan y reportan; nunca preguntan al usuario.
- **Verify** es obligatorio y precede a **Commit**.
- **Commit** solo se invoca con autorizacion explicita del usuario.

---

## Agentes Primarios

| Agente | Responsabilidad | Permisos clave |
|--------|-----------------|----------------|
| `plan` | Analizar requerimiento y generar Historia de Implementacion | `edit: deny`, `bash: deny`, `task: { explore, scout }` |
| `build` | Aceptar Historia o solicitud directa, planificar, implementar, validar y cerrar | `edit: allow`, `question: allow`, bash global, `task` con allowlist cerrada |
| `commit` | Cerrar ciclo con commit semantico (solo tras Verify APROBADO) | `edit: deny`, `task: deny`, `question: deny`, git restringido |

---

## Especialistas

### Subagentes de implementacion (invocados por `build`)

| Especialista | Dominio | Permisos clave |
|--------------|---------|----------------|
| `backend` | APIs, servicios, logica de negocio, integraciones | `edit: allow`, `question: deny`, `task: deny` |
| `frontend` | Componentes, interfaces, estado, UX, responsive | `edit: allow`, `question: deny`, `task: deny` |
| `database` | Queries, SQL, migraciones, indices, optimizacion | `edit: allow`, `question: deny`, `task: deny` |

### Subagentes de gates (invocados por `build`)

| Subagente | Gate | Permisos clave |
|-----------|------|----------------|
| `security` | Seguridad | `edit: deny`, `bash: deny` con allowlist minima, `question: deny` |
| `review` | Calidad | `edit: deny`, `bash: deny`, `question: deny` (solo lectura) |
| `verify` | Cierre obligatorio | `edit: deny`, `bash: deny` con allowlist de validadores, `question: deny` |

### Subagentes de analisis (invocados por `plan` y `build`)

| Subagente | Proposito | Permisos clave |
|-----------|-----------|----------------|
| `explore` | Busqueda rapida en el codebase local, read-only | `edit: deny`, `bash: deny`, `webfetch: deny`, `question: deny` |
| `scout` | Investigacion de documentacion externa, read-only | `edit: deny`, `bash: deny`, `question: deny` |

---

## Skills Globales

| Skill | Proposito |
|-------|-----------|
| `conventional-commits` | Generar mensajes commit siguiendo Conventional Commits |
| `documentation` | Determinar cuando actualizar documentacion |
| `implementation-history` | Validar Historia de Implementacion antes y despues |
| `code-review-principles` | Principios generales para revision de codigo |

---

## MCPs Configurados

OpenCode tiene configurados dos servidores MCP (Model Context Protocol),
ambos deshabilitados por defecto. Habilitarlos requiere autorizacion
explicita por la superficie de acceso que otorgan.

### Context7

**Proposito:** Acceso a documentacion oficial y actualizada de frameworks y librerias.

```json
"context7": {
  "type": "remote",
  "url": "https://mcp.context7.com/mcp",
  "headers": { "CONTEXT7_API_KEY": "{env:CONTEXT7_API_KEY}" },
  "enabled": false
}
```

**Cuando usar:**
- Antes de generar codigo con frameworks o librerias.
- Para consultar APIs oficiales y vigentes.
- Para validar que el codigo usa versiones actuales.

**Tecnologias soportadas:**
Astro, Tailwind CSS, Spring Boot, Angular, React, Next.js, Material UI,
Prisma, NestJS, Docker, Terraform, Azure SDK, y miles mas.

**Autenticacion:** API key desde `CONTEXT7_API_KEY` (variable de entorno).

### Chrome DevTools MCP

**Proposito:** Inspeccion y control del navegador Chrome para analizar aplicaciones web en ejecucion.

```json
"chrome-devtools": {
  "type": "local",
  "command": ["npx", "-y", "chrome-devtools-mcp@1.6.0", "--autoConnect"],
  "enabled": false
}
```

**Requisito previo:** Chrome debe estar ejecutandose con remote debugging
habilitado (`chrome --remote-debugging-port=9222`).

**Capacidades:**
- Inspeccionar DOM/CSS.
- Leer consola y red.
- Accesibilidad y rendimiento.
- Tomar capturas y automatizar interacciones.

### Verificar MCPs

```bash
opencode mcp list
```

Debe mostrar ambos servidores en estado `disabled` mientras no se habiliten
explicitamente.

---

## Politica global de permisos

La allowlist bash global cubre las operaciones locales habituales (lectura,
runtimes, scripts npm/pnpm/yarn/bun, validadores, Git no destructivo,
instalacion local, CLIs de base de datos). Comandos desconocidos caen en
`ask` y se aprueban con `always` para la sesion o con `opencode --auto`
al iniciar OpenCode.

### Categorias preautorizadas (sin pedir confirmacion)

- **Gestion local de dependencias**: `npm install/ci/add/remove/update/ls/view/outdated/audit`, equivalentes `pnpm`, `yarn` y `bun` (sin `-g`/`--global`).
- **Ejecucion de scripts del proyecto**: `npm run *`, `npm exec *`, `pnpm run *`, `pnpm exec *`, `pnpm dlx *`, `yarn run *`, `bun run *`.
- **Test runners directos**: `npm test*`, `pnpm test*`, `yarn test*`, `bun test*`. Cubren `pnpm test`, `pnpm test --watch`, `pnpm test 2>&1`, etc.
- **Runtimes de proyecto**: `node *`, `python *`, `python3 *`, `py *`, `dotnet *`, `go *`, `cargo *`, `gradle *`, `mvn *`, `flutter *`, `dart *`, `swift *`, `xcodebuild *`.
- **Paquetes Python locales**: `pip *`, `pip3 *`.
- **Validacion comun via npx**: `npx tsc*`, `npx eslint*`, `npx prettier*`, `npx vitest*`, `npx jest*`, `npx playwright*`, `npx next*`, `npx vite*`, `npx astro*`, `npx prisma*`, `npx drizzle-kit*`, `npx ts-node*`, `npx nodemon*`. Cualquier otro `npx` cae en `ask`.
- **Inspeccion de solo lectura**: `ls`, `dir`, `cat`, `type`, `echo`, `pwd`, `where`, `which`, `head`, `tail`, `test`, `[ *`, `rg`, `grep`, `find`, versiones de runtimes, cmdlets PowerShell de lectura.
- **Filesystem dentro del workspace**: `mkdir`, `cp`, `copy`, `mv`, `move`, `touch` y los cmdlets PowerShell `Move-Item *`, `Rename-Item *`, `Copy-Item *`. Los accesos fuera del workspace siguen cubiertos por `external_directory: ask`.
- **Git no destructivo**: `git status/diff/log/show/rev-parse`, `git add`, `git stash`, `git switch`, `git checkout`, `git fetch`, `git restore --staged`, lectura de ramas.
- **CLIs de base de datos (lectura/diagnostico)**: `psql`, `mysql`, `sqlite3`, `mongo`, `mongosh`, `redis-cli`, `docker ps/logs`.
- **Impresion y spool**: `lp *`, `lpstat *`, `lpq *`, `lprm *`, `lpoptions *`, `lpr *`, `cancel *`, `print *`. Estas reglas se mantienen porque la autorizacion para imprimir se otorga por el contexto de la peticion explicita del usuario.

### Categorias protegidas (deny)

- **Destructivos**: `rm`, `del`, `rd`, `rmdir`, `Remove-Item`, `format`, `dd`, `mkfs`, `diskpart`.
- **Git de reescritura/externo**: `git push/reset/rebase/clean/amend`, variantes destructivas de commit y cualquier variante de borrado/creacion de ramas (`branch -d/-D/-m/-M/-c/-C/--delete/--move/--copy`). `git commit*` se deniega globalmente; solo `commit` lo permite.
- **Efectos sobre la maquina**: instalaciones globales (`-g`/`--global`/`yarn global`), publicacion de paquetes (`npm/pnpm/yarn/bun publish`), administracion (`sudo`, `runas`, `net user`, `net localgroup`, `sc`, `reg`, `bcdedit`, `netsh`).
- **Wrappers de interprete**: `bash -c *`, `sh -c *`, `zsh -c *`, `cmd /c *`, `powershell -Command *`, `pwsh -Command *`, `eval *`. Se bloquean porque cualquier comando permitido podria invocarse indirectamente.

### Invariantes de proteccion

- `doom_loop: ask` y `external_directory: ask` permanecen activos en el global.
- Los comandos compuestos (`&&`, `||`, `;`, `|`) se dividen por opencode y se
  evaluan subcomando a subcomando; un solo `deny` bloquea todo el bloque.
- `commit` solo autoriza la familia `git` (lectura, staging y commit plano).
  No puede delegar a otros agentes, no puede editar y no puede preguntar.
- `build` declara una allowlist de task con `*: deny` cerrada a sus
  especialistas (frontend, backend, database, security, review, verify,
  explore, scout).
- Todos los subagentes tienen `question: deny`; solo `plan` y `build`
  pueden formular preguntas al usuario.

### Regla de no-delegacion

Los agentes que no implementan declaran `task: deny` para que no puedan
invocar especialistas con `edit: allow`. Esto aplica a `commit`,
`verify`, `review`, `security`, `explore` y `scout`. Solo `build`
(orquesta implementacion y gates) y `plan` (orquesta analisis) tienen
`task` permitido y con allowlist cerrada a sus especialistas.

### Commit y push

- Una solicitud explicita de commit autoriza el commit plano (`git commit*`)
  pero solo tras `Verify: APROBADO`. Variantes destructivas (`--amend`,
  `--no-verify`, `--reuse-message`, `--author`, `--reset-author`)
  permanecen denegadas.
- `git push*` permanece denegado globalmente y debe ejecutarse manualmente.

### Efectos externos y hardware

- Las reglas de impresion, red y administracion no se activan por si solas:
  una peticion explicita del usuario autoriza unicamente el efecto nombrado.
- Si una validacion depende de una herramienta no disponible (CLI, MCP,
  dispositivo), el agente lo declara como limitacion en el cierre; no
  simula un resultado.

Las preguntas funcionales bloqueantes deben agruparse; las preferencias
reversibles deben resolverse sin interrumpir el flujo.

Los cambios de configuracion requieren reiniciar OpenCode para surtir
efecto.

---

## Autonomia y consentimiento de sesion

OpenCode no ofrece un dialogo previo para agregar todos los permisos de
una tarea. La UX equivalente a Codex/Claude es:

1. Configurar la allowlist global cubriendo las operaciones locales
   habituales (lectura, scripts, validadores, runtimes, Git no
   destructivo, dependencias locales).
2. Para aprobar el resto en la sesion actual, ejecutar OpenCode con
   `opencode --auto` o activar **Enable auto-approve permissions** en
   la TUI antes de pedir el trabajo. Esto aprueba todos los `ask` en
   la sesion; los `deny` explicitos siguen bloqueados.
3. Sin `--auto`, cuando aparezca una solicitud, responder:
   - `once`: solo esa llamada.
   - `always`: la operacion exacta y futuras coincidencias durante la
     sesion (no se persiste en `opencode.jsonc`).
   - `reject`: rechazar.
4. Una peticion explicita del usuario en su mensaje inicial autoriza
   el efecto externo nombrado (por ejemplo, ejecutar una impresion o
   enviar a un servicio productivo); no constituye autorizacion
   generica para mas alla.

---

## Flujo de Trabajo

```
  Plan              Build              especialistas     Security   Review   Verify   Commit
  -----             -----              -------------     --------   ------   ------   ------
  Analiza     ->    Plan visible  ->   Implementa   ->   Audita ->  Revisa -> Verifica -> Cierra ciclo
  Historia         Coordina            Reportan
                   Reporta
```

Build acepta una Historia o una solicitud directa. Cuando recibe una
solicitud directa, publica primero un plan y una lista de todos, los
mantiene visibles durante la ejecucion y cierra con un bloque estandar
de cambios, validaciones, hallazgos, pendientes y limitaciones.

Especialistas disponibles para `build`:
`backend` | `frontend` | `database` | `security` | `review` | `verify`
| `explore` | `scout`.

Subagentes disponibles para `plan`:
`explore` | `scout`.

---

## Principios Globales

- Analiza antes de modificar.
- Comprende el problema antes de implementar.
- Busca reutilizacion antes de crear.
- Respeta la arquitectura definida por el proyecto.
- Manten cambios pequenos y enfocados.
- No introduzcas complejidad innecesaria.
- No inventes resultados de validaciones.
- Ejecuta las validaciones posibles antes de finalizar.
- Reporta riesgos y bloqueos encontrados.
- No realices commits ni push sin autorizacion.
- Actua con autonomia local: explora, edita, valida y delega sin
  preguntar por preferencias reversibles.
- Manten visibles el plan y los todos durante la ejecucion.
- Solo pide confirmacion para acciones destructivas, secretos,
  produccion o efectos externos no autorizados.
- Cuando una validacion externa o fisica no sea posible, declaralo en el
  cierre; no inventes resultados.
