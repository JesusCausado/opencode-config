# Entorno Global de OpenCode

## Filosofia

Entorno basado en 3 agentes primarios con responsabilidades unicas. Conocimiento tecnico especifico vive en los proyectos mediante AutoSkills. El entorno global contiene solo metodologia, principios y conocimiento reutilizable.

El entorno prioriza autonomia local: los agentes primarios aceptan una
solicitud directa, ejecutan sin preguntar por preferencias reversibles y
solo detienen el flujo cuando hay una decision funcional bloqueante, una
accion destructiva, un secreto o un efecto externo no autorizado.

---

## Estructura

```
~/.config/opencode/
  AGENTS.md                  Principios permanentes
  opencode.jsonc             Config global + instructions
  agents/                    Agentes primarios y especialistas
    plan.md                  Analisis -> Historia de Implementacion (orquesta explore/scout)
    build.md                 Implementador autonomo (orquesta especialistas)
    commit.md                Cierre del ciclo
    explore.md               Subagente: exploracion rapida del codebase
    scout.md                 Subagente: investigacion externa de librerias
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

## Agentes Primarios

| Agente | Responsabilidad | Permisos clave |
|--------|-----------------|----------------|
| `plan` | Analizar requerimiento y generar Historia de Implementacion | `edit: deny`, `task: { explore, scout }` |
| `build` | Aceptar Historia o solicitud directa, planificar, implementar, validar y cerrar | `edit: allow`, bash con allowlist propia |
| `commit` | Cerrar ciclo con commit semantico | `edit: deny`, `task: deny`, git limitado |

---

## Especialistas

### Subagentes de implementacion (invocados por `build`)

| Especialista | Dominio |
|--------------|---------|
| `backend` | APIs, servicios, logica de negocio, integraciones |
| `frontend` | Componentes, interfaces, estado, UX, responsive |
| `database` | Queries, SQL, migraciones, indices, optimizacion |
| `security` | Auth, autorizacion, vulnerabilidades, criptografia |
| `review` | Calidad de codigo, bugs, regresiones, consistencia |
| `verify` | Evidencia de criterios, validaciones y decision de cierre |

`security`, `review`, `verify`, `explore` y `scout` operan en modo lectura
(`edit: deny`).

### Subagentes de analisis (invocados por `plan`)

| Subagente | Proposito |
|-----------|-----------|
| `explore` | Busqueda rapida en el codebase local, read-only |
| `scout` | Investigacion de documentacion externa y dependencias, read-only |

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

Cada ruleset mantiene `"*": "ask"` como primera regla (el default de
OpenCode es permitir todo; quitarla abriria el shell silenciosamente),
seguido de la allowlist especifica y, al final, los denies destructivos y
de sistema. La ultima regla que coincide gana.

### Categorias preautorizadas (sin pedir confirmacion)

- **Gestion local de dependencias**: `npm install/ci/add/remove/update/ls/view/outdated/audit`, equivalentes `pnpm`, `yarn` y `bun` (sin `-g`/`--global`).
- **Ejecucion de scripts del proyecto**: `npm run *`, `npm exec *`, `pnpm run *`, `pnpm exec *`, `yarn run *`, `bun run *`.
- **Runtime de proyecto**: `node *`, `python *`, `python3 *`, `py *`.
- **Paquetes Python locales**: `pip *`, `pip3 *`.
- **Validacion comun via npx**: `npx tsc*`, `npx eslint*`, `npx prettier*`, `npx vitest*`, `npx jest*`, `npx playwright*`, `npx ts-node*`, `npx nodemon*`. Cualquier otro `npx` cae en `ask`.
- **Inspeccion de solo lectura**: `ls`, `dir`, `cat`, `type`, `echo`, `pwd`, `where`, `which`, `head`, `tail`, `test`, `[ *`, versiones de runtimes.
- **Filesystem dentro del workspace**: `mkdir`, `cp`, `copy`, `mv`, `move`, `touch`. Los accesos fuera del workspace siguen cubiertos por `external_directory: ask`.
- **Git no destructivo**: `git status/diff/log/show/rev-parse`, `git add`, `git stash`, `git switch` (incluye `git switch -c*` para crear ramas), `git checkout`, `git fetch`, `git restore --staged`, lectura de ramas.
- **Impresion y spool**: `lp *`, `lpstat *`, `lpq *`, `lprm *`, `lpoptions *`, `lpr *`, `cancel *`, `print *`. Estas reglas se mantienen porque la autorizacion para imprimir se otorga por el contexto de la peticion explicita del usuario.

### Categorias protegidas (deny)

- **Destructivos**: `rm`, `del`, `rd`, `rmdir`, `Remove-Item`, `format`, `dd`, `mkfs`, `diskpart`.
- **Git de reescritura/externo**: `git push/reset/rebase/clean/amend`, y cualquier variante de borrado/creacion de ramas (`branch -d/-D/-m/-M/-c/-C/--delete/--move/--copy`). `git commit*` se deniega en `build` (solo `commit` lo permite) y se permite plano en `commit`.
- **Efectos sobre la maquina**: instalaciones globales (`-g`/`--global`/`yarn global`), publicacion de paquetes (`npm/pnpm/yarn/bun publish`), administracion (`sudo`, `runas`, `net user`, `net localgroup`, `sc`, `reg`, `bcdedit`, `netsh`).
- **Wrappers de interprete**: `bash -c *`, `sh -c *`, `zsh -c *`, `cmd /c *`, `powershell -Command *`, `pwsh -Command *`, `eval *`. Se bloquean porque cualquier comando permitido podria invocarse indirectamente a traves de ellos.

### Invariantes de proteccion

- `doom_loop: ask` y `external_directory: ask` permanecen activos en el global.
- Los comandos compuestos (`&&`, `||`, `;`, `|`) se dividen por opencode y se
  evaluan subcomando a subcomando; un solo `deny` bloquea todo el bloque.
- `commit` solo autoriza la familia `git` (lectura, staging y commit plano).
  No puede delegar a otros agentes ni escribir archivos.
- `build` declara su propia allowlist de bash para que el resto de los
  agentes no herede automaticamente categorias mas amplias.

### Regla de no-delegacion

Los agentes que no implementan declaran `task: deny` para que no puedan
invocar especialistas con `edit: allow`. Esto aplica a `commit`, `verify`,
`review`, `explore`, `scout` y `security`. Solo `build` (orquesta
implementacion) y `plan` (orquesta analisis) tienen `task` permitido y con
allowlist cerrada a sus especialistas.

### Commit y push

- Una solicitud explicita de commit autoriza el commit plano (`git commit*`).
  Variantes destructivas (`--amend`, `--no-verify`, `--reuse-message`,
  `--author`, `--reset-author`) permanecen denegadas.
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

## Flujo de Trabajo

```
  Plan              Build              Commit
  -----             -----              -----
  Analiza     ->    Plan visible  ->   Cierra ciclo
  Historia         Implementa          Commit
                   Valida              semantico
                   Reporta hallazgos
```

Build acepta una Historia o una solicitud directa. Cuando recibe una
solicitud directa, publica primero un plan y una lista de todos, los
mantiene visibles durante la ejecucion y cierra con un bloque estandar
de cambios, validaciones, hallazgos, pendientes y limitaciones.

Especialistas disponibles para `build`:
`backend` | `frontend` | `database` | `security` | `review` | `verify`.

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
