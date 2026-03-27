# Memory Bank

## 1. Qué es

El Memory Bank es un sistema de contexto persistente basado en archivos para agentes de IA. Vive en `.infracode/memory-bank/` y se compone de dos archivos Markdown plain: `project.md` y `technical.md`. No tiene dependencia de infraestructura externa: son archivos de texto que cualquier herramienta puede leer.

---

## 2. El problema que resuelve

Sin contexto persistente, el agente empieza cada sesión desde cero. Para el developer, esto se traduce en consecuencias concretas y repetitivas:

- Usa APIs deprecadas porque no sabe la versión del stack ni qué librerías se migraron
- Ignora las convenciones del equipo porque nadie se las comunicó en esta sesión
- No sabe si el proyecto corre sobre VTEX IO, Adobe Commerce, un framework propio u otro contexto crítico
- El developer tiene que re-explicar el proyecto en cada sesión antes de poder trabajar

El resultado es fricción acumulada: tiempo perdido en onboarding repetitivo y sugerencias del agente que no encajan con la realidad del proyecto.

---

## 3. Beneficios

| Beneficio | Sin Memory Bank | Con Memory Bank |
|---|---|---|
| Contexto entre sesiones | El agente empieza desde cero en cada conversación | El agente lee el contexto del proyecto al iniciar y lo tiene disponible de inmediato |
| Decisiones técnicas correctas | Puede sugerir librerías incompatibles o patrones que el equipo evita | Conoce el stack, las restricciones y las convenciones antes del primer prompt |
| Menos prompting repetitivo | El developer explica la plataforma, el contexto y las reglas en cada sesión | El contexto ya está escrito; el developer va directo a la tarea |
| Equipo alineado | El contexto del proyecto vive en la cabeza de cada persona | Los archivos son parte del repositorio y cualquier integrante del equipo los tiene disponibles |

---

## 4. Cómo funciona

El Memory Bank usa exactamente dos archivos con roles distintos.

### `project.md` — "Qué somos"

Describe el proyecto desde el punto de vista de negocio y decisiones estructurales. Casi no cambia una vez definido, y lo lidera el humano.

Secciones fijas:

- **Type** — tipo de proyecto (ecommerce, API, herramienta interna, etc.)
- **Platform** — plataforma tecnológica (VTEX IO, Adobe Commerce, Shopify, custom, etc.)
- **Name** — nombre del proyecto o cliente
- **Description** — descripción en 2-3 oraciones de qué hace el sistema
- **Business Context** — contexto de negocio relevante para entender decisiones técnicas
- **Key Constraints** — restricciones críticas que el agente no puede inferir del código (versiones fijadas, restricciones de licencia, limitaciones de plataforma, acuerdos de equipo)

### `technical.md` — "Cómo funcionamos"

Describe cómo está construido el proyecto: stack, arquitectura, módulos clave, integraciones y convenciones. Evoluciona con el proyecto y lo mantiene el agente, siempre con aprobación del developer.

Secciones fijas:

- **Stack** — lenguajes, frameworks, versiones principales
- **Architecture** — descripción de la arquitectura general (monolito, microservicios, serverless, etc.)
- **Key Modules/Extensions** — módulos, extensiones o packages críticos del proyecto
- **Integrations** — servicios externos, APIs de terceros, sistemas conectados
- **Conventions** — convenciones de código, naming, estructura de carpetas, patrones usados
- **Dev Workflow** — cómo se desarrolla, testea y despliega
- **Key Decisions** — decisiones técnicas importantes tomadas y su razón

---

## 5. Flujo de uso

1. **`infracode init`** — inicializa el proyecto y crea los templates vacíos de `project.md` y `technical.md` en `.infracode/memory-bank/`. Si los archivos ya existen, no los modifica.

2. **Completar `project.md` a mano** — abrir el archivo y completar las secciones: plataforma, nombre del cliente, contexto de negocio y constraints críticas. Esta información la conoce el developer, no el agente.

3. **`/infra-memory-bank-init`** — invocar el command en el asistente de IA. El Arquitecto analiza el proyecto (código, configuración, dependencias) y propone contenido para ambos archivos respetando las secciones fijas.

4. **Revisar la propuesta** — especialmente `project.md`: confirmar que la plataforma, el nombre y las constraints son correctos. Corregir lo inexacto antes de aprobar. Para `technical.md` alcanza con confirmar que refleja la realidad del código.

5. **`/infra-memory-bank-update`** — cuando el proyecto evoluciona (cambia el stack, se agrega una integración, se modifica la arquitectura), ejecutar este command. El agente detecta los cambios, propone un diff de `technical.md` y espera aprobación antes de aplicar.

---

## 6. Comandos disponibles

| Command | Cuándo usarlo |
|---|---|
| `/infra-memory-bank-init` | Al iniciar un proyecto nuevo o al incorporar el Memory Bank a un proyecto existente por primera vez. El agente analiza el proyecto y propone el contenido inicial de ambos archivos. |
| `/infra-memory-bank-update` | Cuando el proyecto evolucionó: se cambió el stack, se agregaron integraciones, se actualizó la arquitectura o las convenciones. El agente compara el estado actual del proyecto con el Memory Bank y propone los cambios necesarios en `technical.md`. |
| `/infra-memory-bank-migrate` | Cuando el proyecto ya tiene un memory bank en formato viejo (archivos como `brief.md`, `architecture.md`, `context.md`). El agente mapea el contenido al nuevo formato de dos archivos con secciones fijas. |

---

## 7. Quién modifica qué

| Archivo | El developer | El agente |
|---|---|---|
| `project.md` | Edita directamente y lidera la revisión de cualquier cambio. Es la fuente de verdad sobre el negocio y las constraints. | Propone contenido inicial en `/infra-memory-bank-init`. No modifica el archivo sin aprobación explícita del developer. |
| `technical.md` | Puede editar directamente en cualquier momento. Aprueba o rechaza los cambios propuestos por el agente. | Propone el contenido inicial en `/infra-memory-bank-init` y mantiene el archivo actualizado vía `/infra-memory-bank-update`, siempre mostrando un diff antes de aplicar. |

---

## 8. Buenas prácticas

- Ejecutar `/infra-memory-bank-update` al terminar un sprint o cada vez que cambia el stack, se agrega una integración o se toma una decisión técnica significativa.
- Mantener `project.md` actualizado cuando cambian las constraints o la plataforma — es el archivo que menos cambia pero el más crítico para que el agente tome decisiones alineadas.
- No agregar información de sesión (tareas actuales, bugs en curso, estado del sprint) — eso va en `.infracode/sessions/`, no en el Memory Bank.
- Si el agente propone algo incorrecto en `technical.md`, corregirlo directamente en el archivo — es plain Markdown y cualquier editor sirve.
- Commitear los archivos del Memory Bank al repositorio — son contexto del equipo, no solo de la máquina local. Cualquier integrante que clone el repo tiene el contexto disponible de inmediato.
- Cuanto más preciso y completo sea `project.md`, mejores serán las sugerencias del agente desde el primer prompt de cada sesión.
- Usar la sección **Key Constraints** de `project.md` para restricciones críticas que el agente no puede inferir del código: versiones de plataforma fijadas por contrato, librerías prohibidas, límites de licencia, decisiones de equipo que no se ven en el código.
