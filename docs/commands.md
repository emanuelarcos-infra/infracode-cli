# Referencia de Comandos

> Referencia completa de todos los comandos disponibles en InfraCode CLI.
>
> ← Volver al [índice](index.md)

---

## Resumen de comandos

| Comando | Grupo | Descripción |
|---------|-------|-------------|
| [`init`](#init) | Core / Proyecto | Inicializa un proyecto con Infra AI Framework |
| [`add`](#add) | Core / Proyecto | Agrega una herramienta de IA al proyecto |
| [`status`](#status) | Core / Proyecto | Muestra el estado de las sesiones de los agentes |
| [`sync`](#sync) | Core / Proyecto | Sincroniza los agentes locales con la versión actual del CLI |
| [`preset`](#preset) | Extensiones / Ecosistema | Instala y administra presets remotos |
| [`skills`](#skills) | Extensiones / Ecosistema | Administra las capacidades (skills) de los agentes |
| [`update`](#update) | Configuración Global | Actualiza el binario del CLI a la última versión |
| [`gateway`](#gateway) | Configuración Global | Configura la conexión global al Infra AI Gateway |
| [`install`](#install) | *(eliminado)* | Comando eliminado — ver alternativas |

---

## Core / Proyecto

### `init`

Inicializa un proyecto con Infra AI Framework. Crea la estructura de directorios y configuración necesaria.

**Sintaxis:**

```bash
infracode init [flags]
```

**Flags:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--dry-run` | `bool` | Calcular acciones sin escribir archivos |
| `--force` | `bool` | Sobreescribir todos los conflictos sin confirmar |
| `--yes`, `-y` | `bool` | Modo no interactivo (acepta defaults, no pide confirmación) |
| `--platform` | `string` | Plataforma del proyecto (preselección) |

**Comportamiento:**

1. Detecta si hay una configuración existente y la usa como defaults.
2. Ejecuta un wizard interactivo para seleccionar herramientas de IA, plataforma y Memory Bank.
3. Construye un plan de instalación y resuelve conflictos con archivos existentes.
4. Instala assets (agentes, reglas, workflows) y skills embedded.
5. Crea la estructura `.infracode/` (config, sessions, memory-bank).
6. Actualiza `.gitignore` para excluir archivos de sesión.

**Ejemplos:**

```bash
# Inicialización interactiva (recomendado)
infracode init

# Modo no interactivo con defaults
infracode init --yes

# Ver qué se instalaría sin modificar nada
infracode init --dry-run

# Forzar sobreescritura de archivos existentes
infracode init --force

# Preseleccionar plataforma
infracode init --platform adobe-commerce
```

> **Nota:** En entornos no-TTY (CI/CD), el wizard se omite y se usan los defaults automáticamente.

---

### `add`

Agrega una herramienta de IA al proyecto. Requiere que el proyecto esté inicializado con `infracode init`.

**Sintaxis:**

```bash
infracode add <tool> [flags]
```

**Argumentos:**

| Argumento | Descripción |
|-----------|-------------|
| `<tool>` | Identificador de la herramienta a agregar (ej: `kilocode`, `claude`, `opencode`, `gemini`) |

**Flags:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--dry-run` | `bool` | Mostrar qué se instalaría sin modificar archivos |
| `--force` | `bool` | Sobreescribir archivos existentes |

**Comportamiento:**

1. Canonicaliza el ID de la herramienta.
2. Verifica que no esté ya instalada.
3. Construye un plan de instalación solo para esa herramienta.
4. Instala los assets correspondientes.
5. Actualiza `config.json` con la nueva herramienta.

**Ejemplos:**

```bash
# Agregar soporte para Claude Code
infracode add claude

# Agregar Kilo Code
infracode add kilocode

# Ver qué se instalaría
infracode add opencode --dry-run
```

---

### `status`

Muestra el estado general del proyecto: Memory Bank, plataforma, skills instaladas y sesiones activas.

**Sintaxis:**

```bash
infracode status
```

**Comportamiento:**

- Muestra un banner con la versión del CLI.
- Indica si el Memory Bank está activo o inactivo.
- Muestra la plataforma configurada y las skills instaladas.
- Si la versión del CLI difiere de la del proyecto, sugiere ejecutar `infracode sync`.
- Lista todas las sesiones del proyecto con su estado (done, plan_ready, spec_ready, new, error).
- En terminales anchas muestra una tabla estilizada; en terminales angostas usa tarjetas compactas.

**Ejemplo:**

```bash
infracode status
```

Salida ejemplo:

```
🧠 Memory Bank: activo
🏗️  Plataforma: adobe-commerce
⚡ Skills: memory-bank, find-skills, skills-guide

📊 Sesiones del Proyecto
  Estado         ID                                         Fecha        Requerimiento
  ✅ done        feat-checkout-a1b2c3d4                     2026-03-10   Agregar checkout
  📝 plan_ready  fix-login-e5f6g7h8                         2026-03-11   Fix login redirect

Total: 2 sesiones — ✅ 1 done · 📝 1 plan_ready
```

---

### `sync`

Sincroniza los agentes locales con la versión actual del CLI. Reinstala assets y skills embedded, y detecta herramientas nuevas.

**Sintaxis:**

```bash
infracode sync [flags]
```

**Flags:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--dry-run` | `bool` | Mostrar qué se reinstalaría sin modificar archivos |
| `--force` | `bool` | Sobreescribir archivos existentes sin confirmar |

**Comportamiento:**

1. Lee la configuración del proyecto (`config.json`).
2. Descubre las herramientas disponibles en el FS embebido del CLI.
3. Detecta herramientas nuevas que no estaban en `installed_groups` y las instala automáticamente.
4. Reinstala los assets de todas las herramientas configuradas.
5. Sincroniza skills embedded.
6. Actualiza la versión en `config.json`.

**Ejemplos:**

```bash
# Sincronizar agentes
infracode sync

# Ver qué se sincronizaría
infracode sync --dry-run

# Forzar sobreescritura
infracode sync --force
```

---

## Extensiones / Ecosistema

### `preset`

Instala y administra presets remotos desde repositorios de GitHub.

**Sintaxis:**

```bash
infracode preset <subcomando> [args] [flags]
```

**Subcomandos:**

#### `preset add`

Instala un preset remoto desde GitHub.

```bash
infracode preset add <github.com/org/repo> [flags]
```

**Flags:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--dry-run` | `bool` | Mostrar qué se instalaría sin modificar archivos |
| `--force` | `bool` | Sobreescribir archivos existentes |

**Comportamiento:**

1. Valida que la referencia comience con `github.com/`.
2. Obtiene un token de GitHub para autenticación.
3. Descarga el preset y su manifiesto.
4. Instala los archivos del preset en el proyecto.
5. Crea symlinks según la configuración del manifiesto.

**Ejemplo:**

```bash
# Instalar un preset de plataforma
infracode preset add github.com/ifclatam/infracode-presets

# Ver qué se instalaría
infracode preset add github.com/ifclatam/infracode-presets --dry-run
```

> **Nota:** Requiere autenticación con GitHub. El token se obtiene de la variable de entorno o de la configuración local.

---

### `skills`

Administra las capacidades (skills) de los agentes. Este comando actúa como proxy hacia `npx skills`, inyectando automáticamente los IDs de agentes configurados en el proyecto.

**Sintaxis:**

```bash
infracode skills <subcomando> [args]
```

**Prerequisito:** Requiere Node.js instalado (usa `npx` internamente).

**Subcomandos:**

| Subcomando | Descripción | Inyecta `--agent` |
|------------|-------------|:-----------------:|
| `add` | Agrega un skill al proyecto | ✅ |
| `remove` (alias: `rm`) | Elimina skills del proyecto | ✅ |
| `list` (alias: `ls`) | Lista los skills instalados | ✅ |
| `find` | Busca skills disponibles | ❌ |
| `check` | Verifica los skills instalados | ❌ |
| `update` | Actualiza los skills instalados | ❌ |
| `init` | Inicializa skills en el proyecto | ❌ |
| `experimental_install` | Instala skills (experimental) | ❌ |
| `experimental_sync` | Sincroniza skills (experimental) | ✅ |

**Mapeo de herramientas a agentes:**

| Herramienta (InstalledGroups) | Agent ID |
|-------------------------------|----------|
| `claude` | `claude-code` |
| `opencode` | `opencode` |
| `kilocode` | `kilo` |
| `gemini` | `gemini-cli` |

**Ejemplos:**

```bash
# Buscar skills disponibles
infracode skills find

# Agregar un skill
infracode skills add memory-bank

# Listar skills instalados
infracode skills list

# Eliminar un skill
infracode skills remove memory-bank
```

> **Nota:** Los subcomandos que inyectan `--agent` pasan automáticamente los IDs de las herramientas configuradas en el proyecto. Si no hay herramientas de IA configuradas, el CLI muestra un error y sugiere ejecutar `infracode add`.

---

## Configuración Global

### `update`

Actualiza el binario del CLI a la última versión disponible en GitHub.

**Sintaxis:**

```bash
infracode update [flags]
```

**Flags:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--yes`, `-y` | `bool` | No pedir confirmación interactiva |

**Comportamiento:**

1. Consulta la API de GitHub para obtener la última release.
2. Compara la versión actual con la disponible.
3. Muestra el changelog (primeras 5 líneas).
4. Pide confirmación (a menos que se use `--yes`).
5. Descarga el binario correspondiente al OS/arquitectura actual.
6. Reemplaza el binario en ejecución.

**Ejemplos:**

```bash
# Actualizar con confirmación
infracode update

# Actualizar sin confirmación (útil para scripts)
infracode update --yes
```

---

### `gateway`

Configura la conexión global al Infra AI Gateway. Permite centralizar el acceso a modelos de IA a través de un gateway compartido.

**Sintaxis:**

```bash
infracode gateway [flags]
infracode gateway models [flags]
infracode gateway remove [flags]
```

**Flags del comando principal:**

| Flag | Tipo | Descripción |
|------|------|-------------|
| `--claude` | `bool` | Configurar Claude Code (`~/.claude/settings.json`) |
| `--opencode` | `bool` | Configurar OpenCode (`~/.config/opencode/opencode.jsonc`) |

**Subcomandos:**

#### `gateway models`

Cambia el modelo activo del gateway. Lista los modelos disponibles y permite seleccionar uno.

```bash
infracode gateway models [--claude] [--opencode]
```

#### `gateway remove`

Elimina la configuración del gateway de las herramientas. Sin flags, elimina de ambas herramientas.

```bash
infracode gateway remove [--claude] [--opencode]
```

**Comportamiento:**

- El comando principal ejecuta un wizard interactivo para configurar URL del gateway, token de autenticación y modelo.
- Las credenciales se guardan en `~/.infracode/gateway.json`.
- La configuración se aplica a Claude Code y/o OpenCode según los flags.
- `gateway remove` elimina la configuración del gateway pero mantiene las credenciales para uso futuro.

**Ejemplos:**

```bash
# Configurar gateway (wizard interactivo)
infracode gateway

# Configurar solo para Claude Code
infracode gateway --claude

# Cambiar el modelo activo
infracode gateway models

# Eliminar gateway de Claude Code
infracode gateway remove --claude

# Eliminar gateway de ambas herramientas
infracode gateway remove
```

> **Nota:** Este comando requiere un terminal interactivo (TTY). No funciona en entornos CI/CD.

---

## Comandos eliminados

### `install`

> ⚠️ **Este comando fue eliminado.** Está oculto en el CLI y solo muestra un mensaje de deprecación.

El comando `infracode install` fue reemplazado por los siguientes comandos:

| Caso de uso anterior | Alternativa actual |
|---------------------|--------------------|
| Instalar un preset remoto | `infracode preset add <github.com/org/repo>` |
| Reinstalar agentes | `infracode sync` |
| Inicializar un proyecto nuevo | `infracode init` |

Si ejecutás `infracode install`, el CLI te mostrará las alternativas disponibles.

---

## Flags globales

Estos flags están disponibles en todos los comandos:

| Flag | Descripción |
|------|-------------|
| `--version` | Muestra la versión del CLI en formato `infracode vX.Y.Z (os/arch)` |
| `--help`, `-h` | Muestra la ayuda del comando |

---

## Variables de entorno

| Variable | Efecto |
|----------|--------|
| `NO_COLOR` | Desactiva colores ANSI en la salida |
| `TERM=dumb` | Desactiva colores ANSI en la salida |
| `CI` | Indica entorno de CI — desactiva TTY interactivo |
| `GITHUB_ACTIONS` | Indica GitHub Actions — desactiva TTY interactivo |
| `GITLAB_CI` | Indica GitLab CI — desactiva TTY interactivo |

---

← Volver al [índice](index.md) · Ver [arquitectura](architecture.md) · Ver [conceptos](concepts.md)
