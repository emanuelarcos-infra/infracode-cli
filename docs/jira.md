# Integración con Jira

> Configurá InfraCode CLI para trabajar con tickets de Jira: arrancar sesiones desde un ticket, subir artifacts y sincronizar el estado del trabajo.
>
> ← Volver al [índice](index.md)

---

## ¿Para qué sirve?

Con la integración de Jira podés:

- Arrancar una sesión de trabajo directo desde un ticket existente.
- Descargar artifacts (spec, plan, tasks) que ya estén adjuntos al ticket.
- Subir los artifacts generados por los agentes al ticket correspondiente.
- Sincronizar el estado de las tareas (`tasks.json`) como subtasks en Jira.

**La integración es completamente opcional.** El flujo SDD funciona sin Jira. Si no lo necesitás, podés ignorar esta sección y trabajar con sesiones locales.

---

## Configuración

La configuración tiene dos pasos, y es importante entender la diferencia entre ellos:

> **Login vs Configure — diferencia clave**
>
> `infracode jira login` es **global**: se hace una sola vez en tu máquina y funciona para todos los proyectos. Guarda tus credenciales en `~/.infracode/jira.json`.
>
> `infracode jira configure` es **por proyecto**: se hace dentro de cada proyecto para indicar qué proyecto de Jira corresponde a ese repo. Guarda la selección en `.infracode/config.json` del proyecto (que va al repo).

### Paso 1: Login (una sola vez por máquina)

```bash
infracode jira login
```

Un wizard interactivo te pide tres datos:

| Campo | Descripción |
|-------|-------------|
| **URL del sitio** | La URL de tu instancia de Jira (ej: `https://tu-empresa.atlassian.net`) |
| **Email** | El email de tu cuenta de Atlassian |
| **API Token** | Token de acceso generado desde Atlassian |

Para generar tu API token: [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)

Las credenciales quedan guardadas en `~/.infracode/jira.json` y no se comparten con ningún repositorio.

Una vez que completás el login, el wizard continúa automáticamente al Paso 2.

---

### Paso 2: Configure (una vez por proyecto)

```bash
infracode jira configure
```

Parado en el directorio del proyecto, este comando lista los proyectos disponibles en tu Jira y te deja seleccionar cuál corresponde a este repo. La selección se guarda en `.infracode/config.json`.

Podés correrlo de nuevo en cualquier momento si querés cambiar el proyecto vinculado.

---

### Verificar que todo funciona

```bash
infracode jira status
```

Muestra el estado de la conexión y la configuración actual:

```
Jira: conectado
  URL:      https://tu-empresa.atlassian.net
  Email:    dev@tu-empresa.com
  Proyecto: DII (Demo Infracommerce Integration)
```

Si algo no está configurado, el comando te dice qué paso falta.

---

## Referencia de comandos

| Comando | Descripción |
|---------|-------------|
| `infracode jira login` | Wizard para configurar credenciales (global, una sola vez) |
| `infracode jira logout` | Elimina las credenciales guardadas |
| `infracode jira status` | Verifica la conexión y muestra la configuración actual |
| `infracode jira configure` | Selecciona el proyecto Jira para este repo (por proyecto) |
| `infracode jira issue get <ticket>` | Obtiene la metadata del ticket en formato JSON |
| `infracode jira pull <ticket> --session <ID>` | Descarga los artifacts adjuntos al ticket hacia la sesión local |
| `infracode jira push --session <ID>` | Sube los artifacts de la sesión al ticket vinculado |
| `infracode jira tickets` | Lista los tickets asignados a tu usuario en el proyecto configurado |

---

## Flujos de trabajo

### Flujo 1: Arrancar desde un ticket existente

Este es el flujo principal. Si ya tenés un ticket en Jira, simplemente mencionáselo al orquestador.

```
Usuario: "Trabajá en DII-137"

Orchestrator: Detectando ticket DII-137...
              Obteniendo metadata de Jira...

              Ticket DII-137: Demo end-to-end del flujo SDD con Jira
              Status: En curso · Assignee: vos · Artifacts: ninguno

              No hay sesión existente vinculada a DII-137.
              Arranco desde spec. ¿Dale?

Usuario: dale

Orchestrator: [lanza Spec Agent con el título y descripción del ticket como input]
```

Internamente, el orquestador:

1. Detecta el patrón de ticket (`PROJ-NNN`) en tu mensaje.
2. Verifica que Jira esté configurado en el proyecto.
3. Busca si ya existe una sesión local vinculada a ese ticket.
4. Si no existe: obtiene la metadata del ticket y crea la sesión.
5. Ejecuta `jira pull` para descargar artifacts si los hay.
6. Determina desde qué fase arrancar (ver tabla de detección de fase).
7. Muestra un resumen y pide confirmación antes de arrancar.

---

### Flujo 2: Vincular una sesión existente a Jira

Si arrancaste la spec sin un ticket y ahora querés vincularlo, podés hacerlo en cualquier momento.

Al completar la spec, el orquestador te lo ofrece automáticamente:

```
Orchestrator: Spec aprobada. ¿Querés crear un ticket en Jira con esta spec?
              [S/n]

Usuario: s

Orchestrator: Ticket DII-142 creado. Subiendo spec.md...
              Sesión feat-checkout-a1b2c3d4 vinculada a DII-142.
```

O podés hacerlo manualmente:

```bash
infracode jira push --session feat-checkout-a1b2c3d4
```

Esto sube todos los artifacts disponibles en la sesión (spec, plan, tasks, qa-report) y deja la sesión vinculada al ticket para futuros pushes.

---

### Flujo 3: Sincronización continua

Una vez que la sesión está vinculada a un ticket, la sincronización ocurre automáticamente en puntos clave:

| Evento | Acción automática |
|--------|------------------|
| Spec aprobada | Push de `spec.md` al ticket |
| QA PASS (validate completo) | Push de `qa-report.md` al ticket |
| Tarea del plan completada | Subtask en Jira marcada como completada |

También podés hacer push manual cuando quieras:

```bash
infracode jira push --session <ID>
```

Las tareas de `tasks.json` se sincronizan como subtasks del ticket. Cada tarea con `status: completed` actualiza su subtask correspondiente en Jira.

---

## Detección automática de fase

Cuando el orquestador descarga artifacts de un ticket, decide desde dónde arrancar según lo que encuentra:

| Artifacts en el ticket | Fase de arranque |
|------------------------|-----------------|
| Sin `spec.md` | `spec` — arranca generando la spec desde el título y descripción del ticket |
| Con `spec.md` | `plan` — la spec ya existe, arranca generando el plan |
| Con `spec.md` + `plan.md` | `code` — el plan ya existe, arranca la implementación |
| Con `spec.md` + `plan.md` + `qa-report.md` | `validate` — hay un reporte previo, arranca revisando el estado del QA |

Esta detección es automática. El orquestador siempre te muestra la fase detectada y te pide confirmación antes de arrancar.

---

## Troubleshooting

**"Jira no está configurado"**

```
Error: Jira no está configurado en este proyecto.
Ejecutá: infracode jira login
```

Si ya hiciste login antes (en otra máquina o reinstalaste), alcanza con `infracode jira configure` para vincular el proyecto.

---

**Ticket no encontrado**

```
Error: Ticket DII-999 no encontrado.
Verificá que la key sea correcta y que tengas acceso al proyecto.
```

Verificá que el número de ticket exista en tu instancia de Jira y que tu usuario tenga permisos de lectura sobre el proyecto.

---

**API token inválido o expirado**

```
Error: Autenticación fallida (401).
El API token puede estar expirado. Regeneralo en Atlassian y volvé a ejecutar: infracode jira login
```

Generá un token nuevo en [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens) y volvé a correr `infracode jira login`.

---

**Límite de tamaño de artifacts**

Los artifacts adjuntos a tickets en Jira tienen un límite de **10 MB por archivo**. Si un archivo supera ese límite, el push falla indicando el archivo problemático. Revisá que `spec.md`, `plan.md` y `qa-report.md` no superen ese límite.

---

**¿A qué ticket está vinculada una sesión?**

```bash
infracode status
```

La tabla de sesiones muestra la columna `Ticket` con el key de Jira vinculado (si existe).

---

← Volver al [índice](index.md) · Ver [flujo SDD](workflow.md)
