# Conceptos Clave

> Los tres pilares del Infra AI Framework: Memory Bank, Skills y Presets.
>
> ← Volver al [índice](index.md)

---

## Memory Bank

### ¿Qué es?

El Memory Bank es un sistema de **memoria persistente del proyecto** que le da contexto a los asistentes de IA entre sesiones. Sin él, cada conversación con el asistente empieza de cero, sin conocimiento previo sobre tu proyecto.

### ¿Para qué sirve?

- Que el asistente de IA conozca el stack tecnológico, la plataforma y las convenciones del proyecto.
- Que las respuestas sean más precisas y alineadas al contexto real.
- Que no haya que repetir información básica en cada sesión.

### Estructura

El Memory Bank vive en `.infracode/memory-bank/` y contiene dos archivos:

| Archivo | Contenido |
|---------|-----------|
| `project.md` | Negocio y propósito del proyecto, plataforma, constraints |
| `technical.md` | Stack tecnológico, arquitectura, convenciones de código, integraciones |

### Inicialización

El Memory Bank se puede inicializar de dos formas:

#### 1. Durante `infracode init`

El wizard de inicialización pregunta si querés activar el Memory Bank. Si aceptás, se crean los archivos con templates mínimos:

```bash
infracode init
# → Seleccionar "Sí" cuando pregunte por Memory Bank
```

Los templates creados contienen secciones con comentarios HTML para que los completes:

```markdown
# Project

## Negocio y Propósito

<!-- Qué es el proyecto, a qué industria pertenece, cuál es su propósito principal -->

## Plataforma y Constraints

<!-- Detalles sobre la plataforma en la que corre y constraints de negocio -->
```

#### 2. Con el workflow `/infra:memory-bank-init`

Este workflow analiza el código del proyecto y genera automáticamente el contenido de `project.md` y `technical.md`:

```
/infra:memory-bank-init
```

El asistente analiza tu código, detecta stack/plataforma, y genera el contenido. Vos aprobás antes de que se guarde.

### Mantenimiento

Después de cambios significativos en el proyecto (nuevo framework, migración de base de datos, etc.), actualizá el Memory Bank:

```
/infra:memory-bank-update
```

### Comportamiento no destructivo

La inicialización del Memory Bank es **no destructiva**: si los archivos `project.md` y `technical.md` ya existen, no se sobreescriben. Esto protege el contenido que ya hayas personalizado.

### Verificación

Podés verificar si el Memory Bank está activo con:

```bash
infracode status
# → 🧠 Memory Bank: activo
```

> Para documentación detallada del Memory Bank, ver [memory-bank.md](memory-bank.md).

---

## Skills

### ¿Qué son?

Las Skills son **habilidades explícitas y reutilizables** que los asistentes de IA pueden ejecutar. Definen propósito, inputs esperados y outputs esperados, permitiendo que el asistente sepa exactamente qué puede hacer y cómo hacerlo.

### Tipos de skills

#### Skills embedded

Son skills que vienen incluidas en el binario del CLI y se instalan automáticamente durante `infracode init` o `infracode sync`. Ejemplos:

- **memory-bank**: gestión del Memory Bank del proyecto.
- **find-skills**: descubrimiento e instalación de skills adicionales.
- **skills-guide**: guía de referencia para gestionar skills.

Estas skills se instalan en:
- `.kilocode/skills/` para Kilo Code
- `.claude/skills/` para Claude Code

#### Skills de terceros

Son skills creadas por la comunidad o por equipos internos. Se instalan usando el comando `infracode skills add`:

```bash
infracode skills add <nombre-del-skill>
```

### Gestión desde el CLI

El comando [`infracode skills`](commands.md#skills) actúa como proxy hacia `npx skills`, inyectando automáticamente los IDs de agentes configurados en el proyecto.

| Acción | Comando |
|--------|---------|
| Buscar skills disponibles | `infracode skills find` |
| Agregar un skill | `infracode skills add <nombre>` |
| Listar skills instalados | `infracode skills list` |
| Eliminar un skill | `infracode skills remove <nombre>` |
| Verificar skills | `infracode skills check` |
| Actualizar skills | `infracode skills update` |

### Prerequisitos

El comando `skills` requiere **Node.js** instalado, ya que usa `npx` internamente para ejecutar el gestor de skills.

### Registro de skills

Los skills instalados se registran en `skills-lock.json` en la raíz del proyecto. Este archivo es leído por `infracode status` para mostrar las skills activas.

---

## Presets

### ¿Qué son?

Los Presets son **paquetes de configuración predefinidos** que se pueden instalar desde repositorios remotos de GitHub. Contienen reglas, workflows y configuraciones específicas para una plataforma, tecnología o caso de uso.

### Preset base vs. presets remotos

| Tipo | Descripción | Instalación |
|------|-------------|-------------|
| **Preset base** | Incluido en el binario del CLI. Contiene reglas base, Context7 MCP y workflow de Memory Bank. | Automática con `infracode init` |
| **Presets remotos** | Paquetes adicionales en repositorios de GitHub. Contienen configuraciones específicas de plataforma. | Manual con `infracode preset add` |

### Instalación de presets remotos

```bash
infracode preset add github.com/ifclatam/infracode-presets
```

**Flujo de instalación:**

1. El CLI valida la referencia del repositorio (debe empezar con `github.com/`).
2. Obtiene un token de GitHub para autenticación.
3. Descarga el contenido del preset y su manifiesto (`manifest.json` o similar).
4. Instala los archivos en `.infracode/presets/{nombre}/`.
5. Crea symlinks desde los archivos del preset hacia las ubicaciones esperadas por cada herramienta.

### Manifiesto del preset

Cada preset contiene un manifiesto que define:
- **Nombre** del preset.
- **Archivos** a instalar y sus destinos.
- **Symlinks** a crear para que las herramientas de IA encuentren los archivos.

### Fuente de verdad

Los archivos del preset en `.infracode/presets/` son la **fuente de verdad**. Si necesitás modificar algo, editá los archivos allí — los cambios se reflejan automáticamente a través de los symlinks.

### Ejemplos de presets

```bash
# Preset para Adobe Commerce
infracode preset add github.com/ifclatam/infracode-presets#platform/adobe-commerce

# Preset para VTEX
infracode preset add github.com/ifclatam/infracode-presets#platform/vtex

# Preset para Shopify
infracode preset add github.com/ifclatam/infracode-presets#platform/shopify
```

### Autenticación

La instalación de presets remotos requiere autenticación con GitHub. El token se obtiene de:
- Variable de entorno (ej: `GITHUB_TOKEN`)
- Configuración local de GitHub

---

## Relación entre conceptos

```
┌─────────────────────────────────────────────────────┐
│                  InfraCode CLI                       │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │  Memory   │  │  Skills  │  │     Presets       │   │
│  │  Bank     │  │          │  │                   │   │
│  │           │  │ embedded │  │ base (en binario) │   │
│  │ project.md│  │ + 3ros   │  │ + remotos (GitHub)│   │
│  │technical.md│ │          │  │                   │   │
│  └──────────┘  └──────────┘  └──────────────────┘   │
│       │              │              │                 │
│       ▼              ▼              ▼                 │
│  Contexto del    Capacidades    Configuración         │
│  proyecto        del agente     de plataforma         │
└─────────────────────────────────────────────────────┘
```

- **Memory Bank** → le da al agente **contexto** sobre el proyecto.
- **Skills** → le da al agente **capacidades** específicas.
- **Presets** → le da al agente **configuración** alineada a la plataforma.

Los tres trabajan juntos para que el asistente de IA sea más efectivo y alineado al proyecto real.

---

← Volver al [índice](index.md) · Ver [comandos](commands.md) · Ver [arquitectura](architecture.md)
