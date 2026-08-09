# Entorno Global de OpenCode

## Filosofia

Entorno basado en 3 agentes primarios con responsabilidades unicas. Conocimiento tecnico especifico vive en los proyectos mediante AutoSkills. El entorno global contiene solo metodologia, principios y conocimiento reutilizable.

---

## Estructura

```
~/.config/opencode/
  AGENTS.md                  Principios permanentes
  opencode.jsonc             Config global + instructions
  agents/                    Agentes primarios y especialistas
    plan.md                  Analisis -> Historia de Implementacion (orquesta explore/scout)
    build.md                 Implementacion tecnica (orquesta especialistas)
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

| Agente | Responsabilidad | Permisos |
|--------|-----------------|----------|
| `plan` | Analizar requerimiento y generar Historia de Implementacion | `edit: deny` |
| `build` | Implementar la Historia coordinando especialistas | completo |
| `commit` | Cerrar ciclo con commit semantico | `edit: deny` |

---

## Especialistas

### Subagentes de Implementacion (invocados por `build`)

| Especialista | Dominio |
|--------------|---------|
| `backend` | APIs, servicios, logica de negocio, integraciones |
| `frontend` | Componentes, interfaces, estado, UX, responsive |
| `database` | Queries, SQL, migraciones, indices, optimizacion |
| `security` | Auth, autorizacion, vulnerabilidades, criptografia |
| `review` | Calidad de codigo, bugs, regresiones, consistencia |
| `verify` | Evidencia de criterios, validaciones y decision de cierre |

`security` y `review` operan en modo lectura (`edit: deny`).

### Subagentes de Analisis (invocados por `plan`)

| Subagente | Proposito |
|-----------|-----------|
| `explore` | Busqueda rapida en el codebase local, read-only |
| `scout` | Investigacion de documentacion externa y dependencias, read-only |

`explore` y `scout` operan en modo lectura (`edit: deny`, `bash: deny`).

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
actualmente deshabilitados por defecto:

### Context7

**Proposito:** Acceso a documentación oficial y actualizada de frameworks y librerías.

**Configuracion:**
```json
"context7": {
  "type": "remote",
  "url": "https://mcp.context7.com/mcp",
  "headers": {
    "CONTEXT7_API_KEY": "{env:CONTEXT7_API_KEY}"
  },
  "enabled": false
}
```

**Cuando usar:**
- Antes de generar código con frameworks o librerías
- Para consultar APIs oficiales y vigentes
- Para validar que el código usa versiones actuales
- Cuando haya dudas sobre configuraciones específicas

**Tecnologias soportadas:**
Astro, Tailwind CSS, Spring Boot, Angular, React, Next.js, Material UI, Prisma, NestJS, Docker, Terraform, Azure SDK, y miles más.

**Uso en prompts:**
```
use context7 to query the latest Astro 5.x documentation
```

**Autenticación:** Configurada con API key desde `CONTEXT7_API_KEY` (variable de entorno). Si necesitas crear una, obtén una gratuita en [context7.com](https://context7.com) y configúrala como variable de entorno.

---

### Chrome DevTools MCP

**Proposito:** Inspección y control del navegador Chrome para analizar aplicaciones web en ejecución.

**Configuracion:**
```json
"chrome-devtools": {
  "type": "local",
  "command": ["npx", "-y", "chrome-devtools-mcp@1.6.0", "--autoConnect"],
  "enabled": false
}
```

**Requisito previo:** Chrome debe estar ejecutándose con remote debugging habilitado:
```bash
# Windows
chrome.exe --remote-debugging-port=9222

# macOS
Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222
```

**Capacidades:**
- Inspeccionar el DOM y CSS
- Leer errores de consola
- Revisar peticiones de red
- Analizar rendimiento
- Revisar accesibilidad
- Tomar capturas
- Navegar, hacer clic, escribir formularios

**Cuando usar:**
- Antes de proponer cambios visuales en frontend
- Para validar implementaciones contra el navegador real
- Para detectar errores de consola o problemas de CSS
- Para comparar implementación con diseño

**Uso en prompts:**
```
use chrome-devtools to inspect the current page and check for console errors
```

---

### Verificar MCPs

```bash
opencode mcp list
```

Debe mostrar ambos servidores en estado `disabled` mientras no se habiliten
explícitamente.

---

## Política global de permisos

La configuración global prioriza continuidad sin eliminar controles críticos:

- Lectura, exploración y consultas Git seguras no requieren aprobación.
- Las validaciones comunes tienen allowlist por nombre; no se permite el
  comodín global `npm run *`.
- El resto de comandos Bash permanece en `ask`.
- Una solicitud explícita de commit autoriza el commit; push, reescritura de
  historial y operaciones destructivas permanecen bloqueados.
- Reset, rebase, clean y otras operaciones destructivas permanecen bloqueadas
  para agentes de implementación.
- Secretos, directorios externos, bucles repetitivos y efectos externos
  mantienen controles de seguridad.
- Las preguntas funcionales bloqueantes deben agruparse; las preferencias
  reversibles deben resolverse sin interrumpir el flujo.

Los cambios de configuración requieren reiniciar OpenCode para surtir efecto.

---

## Flujo de Trabajo

```
  Plan              Build              Commit
  -----             -----              -----
  Analiza     ->    Implementa    ->   Cierra ciclo
  Historia         Coordina            Commit
                   especialistas       semantico
```

Especialistas disponibles para `build`:
- `backend` | `frontend` | `database` | `security` | `review` | `verify`

Subagentes disponibles para `plan`:
- `explore` | `scout`

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

---

## Notas

- Config se carga al iniciar. Reinicia OpenCode despues de cambios.
- `opencode.jsonc` declara `$schema` para validacion automatica.
- Conocimiento tecnico especifico (frameworks, BD, patrones) vive en AutoSkills del proyecto.
- Especialistas son `mode: subagent`, invisibles en el menu principal.
# opencode-config
