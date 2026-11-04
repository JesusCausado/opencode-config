# Entorno Global de OpenCode

## Filosofia

Entorno basado en agentes primarios y subagentes con responsabilidades
claras. El usuario concede el consentimiento al inicio de la sesion y el
pipeline ejecuta autonomamente hasta el cierre, salvo bloqueos reales.

El entorno prioriza autonomia estilo Codex/Claude: bash permite todo por
defecto y deniega explicitamente lo destructivo o externo. Build corre sin
supervision (`question: deny`) y nunca se queda esperando un `ask`.

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
| `build` | Implementar sin supervision: especialistas, gates, verify y Cierre listo para commit | `edit: allow`, `question: deny`, bash global, `task` con allowlist cerrada |
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

Objetivo: que `build` termine una implementacion **sin supervision**. El
modelo es "permitir por defecto, denegar lo peligroso".

- El global define `bash: { "*": "allow", ...deny }`. Las reglas se
  encadenan global -> agente y **gana la ultima que coincide**. Un agente
  que redefine `"*"` anula los deny globales y debe repetir los que
  necesite (`commit`, `verify` y `security` usan `"*": deny` con
  allowlist propia).
- Se usa `deny` en lugar de `ask`: un `deny` no bloquea la sesion, el
  agente recibe el rechazo y busca otra via. Un `ask` deja la sesion
  esperando a alguien que puede no estar.
- `question`: solo `plan` lo tiene habilitado. `build` y los subagentes
  deciden lo reversible y reportan los bloqueos en el Cierre.
- `doom_loop: deny` corta bucles repetitivos. `external_directory: allow`:
  un deny bloquearia la lectura de skills (opencode agrega sus allow antes
  que la config) y bash ya puede salir del repo, asi que no protegeria.
- `edit: ask` global; cada agente que implementa declara `edit: allow`.
- Los comandos compuestos (`&&`, `||`, `;`, `|`) se evaluan subcomando
  a subcomando; un solo `deny` bloquea todo.

### Categorias denegadas

- **Git de historial/ramas**: `push`, `commit` (solo `commit` lo habilita),
  `reset`, `rebase`, `merge`, `clean`, `filter-branch`, `update-ref`,
  `stash drop/clear`, `checkout -- / .`, `restore` (salvo `--staged`),
  borrado/renombre de ramas y tags, y `git -C *` (evita saltar patrones).
- **Destructivos de disco**: `rm`, `del`, `erase`, `rd`, `rmdir`,
  `Remove-Item`/`ri`, `format`, `dd`, `mkfs`, `diskpart`, `cipher /w`.
- **Maquina**: instalaciones globales y `publish`, `choco`/`winget`/`scoop`,
  `sudo`, `runas`, `net user/localgroup`, `sc`, `reg`, `bcdedit`, `netsh`,
  `shutdown`, `Stop/Restart-Computer`, `Set-ExecutionPolicy`,
  `Start-Process`.
- **Efectos externos/produccion**: `ssh`, `scp`, `gh pr merge`,
  `gh release`, `gh repo delete`, borrados de `docker`, `kubectl
  apply/delete`, `terraform apply/destroy`, `dropdb`.
- **Wrappers de interprete**: `bash/sh/zsh -c`, `cmd /c`,
  `powershell|pwsh -c|-e...`, `Invoke-Expression`/`iex`, `eval`.

### Riesgo aceptado

Una lista de deny por prefijo no es una caja de arena: un comando permitido
(`node -e`, `python -c`, `npx <paquete>`) puede producir efectos que los
patrones no ven. La red de seguridad final es revisar el diff antes de
aprobar el commit, y que `push` siempre sea manual.

### Regla de no-delegacion

Los agentes que no implementan declaran `task: deny`. Solo `build`
(implementacion y gates) y `plan` (analisis) tienen `task`, con
allowlist cerrada a sus especialistas.

### Commit y push

- `commit` habilita `git commit` plano tras `Verify: APROBADO`; las
  variantes destructivas (`--amend`, `--no-verify`, `--author`, ...)
  siguen denegadas.
- `git push*` permanece denegado globalmente y se ejecuta manualmente.

### Efectos externos y hardware

- Una peticion explicita del usuario autoriza unicamente el efecto
  nombrado (impresion, servicio productivo).
- Si una validacion depende de una herramienta no disponible (CLI, MCP,
  dispositivo), el agente lo declara como limitacion; no simula resultados.

### Modelos

Ningun agente fija `model`: los subagentes heredan el del primario que
los invoca. Cambiar el modelo de la sesion de `build` (`/models`) cambia
el de todo el pipeline. Para fijar uno, agrega `model: <proveedor>/<modelo>`
al frontmatter del agente (ver `opencode models`).

Los cambios de configuracion requieren reiniciar OpenCode.

---

## Autonomia

Con esta configuracion no hace falta `opencode --auto`: lo no denegado ya
esta permitido. `--auto` sigue disponible como respaldo si algun proyecto
agrega reglas `ask` en su propio `opencode.json`; los `deny` explicitos
se respetan igual.

Flujo de uso: en `plan` resuelves las decisiones funcionales y apruebas la
Historia; cambias a `build` (Tab) y puedes irte; al volver lees el Cierre
(`Estado: LISTO PARA COMMIT | REQUIERE ATENCION`), revisas el diff y
cambias a `commit`.

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
