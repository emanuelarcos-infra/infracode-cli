# InfraAI Framework

> Spec-Driven Development: un proceso estandarizado para trabajar con inteligencia artificial en desarrollo de software.

---

## Qué es el InfraAI Framework

El Infra AI Framework es una metodología de desarrollo que estandariza la forma en que equipos de ingeniería trabajan con asistentes de IA para construir software. En lugar de usar la IA como un chat de preguntas y respuestas ("vibe coding"), el framework la convierte en un proceso de entrega estructurado — igual que un equipo humano de ingeniería.

El framework define un flujo de cuatro etapas — **Spec, Plan, Code, Validate** — donde cada etapa tiene un rol asignado, un input esperado, un output verificable, y un gate de aprobación humano. El resultado es un proceso colaborativo entre humanos e IA que es **consistente, medible y escalable**.

**InfraCode CLI** es la herramienta que implementa este framework. Configura los asistentes de IA con agentes especializados, inyecta contexto del proyecto, y gestiona el ciclo de vida de cada sesión de desarrollo.

---

## El problema que resuelve

Cuando un equipo usa asistentes de IA para desarrollar software sin un proceso definido, se enfrenta a estos problemas:

**Inconsistencia de output.** El mismo requerimiento puede producir resultados muy diferentes dependiendo de cómo se lo pidas al modelo. No hay garantía de calidad ni de estructura.

**Falta de contexto.** Cada nueva sesión con la IA empieza de cero. El asistente no sabe qué stack usás, qué convenciones tiene el proyecto, ni qué decisiones se tomaron antes.

**Imposibilidad de medir.** Si no hay criterios de aceptación definidos antes de la implementación, no hay forma objetiva de saber si lo que se generó está bien o no.

**No escala.** Lo que funciona para un developer en una tarde no funciona cuando tenés múltiples equipos trabajando en paralelo. Sin estandarización, cada persona usa la IA de forma diferente.

**Random acumulado.** Cada paso que la IA improvisa introduce variabilidad. Si la especificación es vaga, el plan será vago, el código será impredecible, y la validación no tiene contra qué comparar.

---

## La premisa fundamental

La IA responde bien cuando tiene dos cosas:

1. **Contexto suficiente** — conoce el proyecto, la plataforma, las restricciones, la arquitectura, las convenciones.
2. **Instrucciones claras** — sabe qué se espera como resultado, en qué formato, y con qué criterios se va a evaluar.

El framework provee ambas. El **Memory Bank** inyecta contexto persistente del proyecto. Los **agentes especializados** definen instrucciones claras para cada etapa del proceso. Y el **flujo SDD** asegura que el output de cada etapa esté bien definido antes de pasar a la siguiente, reduciendo el random en cada paso.

---

## Tradeoffs: cuándo usar SDD

SDD no es para todo. Tiene un costo de entrada — más tokens, tiempo de planificación, ciclo Spec-Plan-Code — que solo vale la pena cuando el problema es lo suficientemente complejo.

```
                    INVERSIÓN (costo)              RESULTADO (valor)
                    ─────────────────              ─────────────────
                    + Tokens (2-3x más)            + Performance dramático
                    + Tiempo de planificación      + Ambigüedad reducida
                    + Ciclo Spec-Plan-Code          + Features complejos correctos
                              \                        /
                               ────────────────────────
                                     SDD
```

### 🔴 Cuándo NO usar SDD

| Caso | Por qué no aplica |
|------|-------------------|
| Cambios pequeños | Fix de un typo, ajuste de color, rename de variable |
| Bug fixes rápidos | Error obvio con causa conocida y solución directa |
| Actualizaciones de config | Cambio de env var, ajuste de parámetro, update de dependencia |

→ Para estos casos, usá Plan Mode o prompt directamente al asistente. El overhead de spec + plan no vale la pena.

### 🟢 Cuándo SDD brilla

| Caso | Por qué funciona |
|------|-----------------|
| Features complejos | Múltiples casos de uso, validaciones y edge cases que hay que definir antes de codear |
| Cambios multi-archivo | Modificaciones que impactan varios módulos o capas del sistema |
| Lógica multi-dominio | La implementación cruza boundaries (ej: auth + pagos + notificaciones) |

→ El driver es **reducción de ambigüedad**. Cuanto más vaga puede ser la interpretación del requerimiento, más valor da la spec.

---

## Las cuatro etapas

### Etapa 1: Spec — Definir qué construir

**Rol:** Business Analyst

Un requerimiento crudo ("necesito que el checkout soporte cuotas") se transforma en una especificación funcional completa con criterios de aceptación medibles, edge cases, supuestos y riesgos.

**Input:** Requerimiento del usuario (en lenguaje natural).

**Output:** `spec.md` — especificación funcional con:

- Requerimiento original (literal, sin parafrasear)
- Objetivo y alcance (qué incluye, qué no)
- Descripción funcional detallada
- Criterios de aceptación (formato AC-01, AC-02... con condiciones verificables)
- Edge cases identificados
- Supuestos y riesgos funcionales
- Glosario

**Gate:** El humano revisa la spec y decide: aprobar, pedir cambios, o cancelar. Nada avanza hasta que la spec esté aprobada.

**Por qué importa:** La spec es la única fuente de verdad. Todo lo que viene después — el plan, el código, la validación — se mide contra este documento. Si la spec es vaga, todo lo demás es impredecible.

---

### Etapa 2: Plan — Diseñar cómo construirlo

**Rol:** Arquitecto de Software

A partir de la spec aprobada y el contexto del proyecto, se genera un plan de implementación ejecutable. No es una lista de pasos genérica — es un diseño técnico con decisiones evaluadas, trade-offs documentados, y tareas atómicas con dependencias.

**Input:** `spec.md` aprobada + contexto del proyecto (Memory Bank + codebase).

**Output:**

- `plan.md` — plan de implementación con:
  - Estrategia técnica con opciones evaluadas
  - Decisiones de arquitectura con trade-offs
  - Componentes impactados
  - Riesgos técnicos y mitigación
  - Plan de pruebas
- `tasks.json` — backlog de tareas con:
  - ID, título, descripción
  - Archivos a crear/modificar
  - Criterios de aceptación por tarea
  - Dependencias entre tareas
  - Flag de paralelización

**Gate:** El humano revisa el plan y decide: aprobar, pedir cambios, o cancelar.

**Por qué importa:** El plan es el puente entre el "qué" y el "cómo". Convierte criterios de aceptación abstractos en tareas concretas. Y al definir dependencias, permite que las tareas sin dependencia se ejecuten en paralelo, acelerando los tiempos de desarrollo.

---

### Etapa 3: Code — Implementar

**Rol:** Developer

La IA no improvisa — sigue el plan aprobado, tarea por tarea. Lee el plan, la spec, y el contexto del proyecto. Implementa el código alineado a las convenciones y la arquitectura existente. Escribe tests. Ejecuta build y tests después de cada tarea.

**Input:** `tasks.json` con tareas pendientes + `plan.md` + `spec.md` + codebase.

**Output:**

- Código implementado
- Tests escritos
- Tareas actualizadas en `tasks.json` con evidencia de ejecución (build pass, tests pass, files modified)

**Ejecución de tareas:**

- Las tareas sin dependencias se pueden ejecutar en paralelo (en herramientas que lo soporten)
- Cuando una tarea se completa, se desbloquean las que dependían de ella
- Si el build o tests fallan, se activa automáticamente el Debug Agent

**Por qué importa:** La clave no es escribir código más rápido. Es escribir código **bien** — alineado al plan, verificado con tests, y consistente con las prácticas del proyecto.

---

### Etapa 4: Validate — Verificar

**Rol:** QA Specialist

Verifica que la implementación cumple todos los criterios de aceptación definidos en la spec. Ejecuta el build, corre tests, hace code review, y produce un reporte con resultado PASS o FAIL por cada criterio.

**Input:** `spec.md` + código implementado + tests.

**Output:** `qa-report.md` con:

- Resultado global: PASS o FAIL
- Tabla de criterios de aceptación con estado (PASS/FAIL) y evidencia
- Validaciones técnicas (build, tests, lint)
- Code review
- Hallazgos clasificados por severidad (critical, major, minor)

**Gate:** Si PASS, el feature está listo para PR. Si FAIL, se vuelve a Code con el feedback del QA (máximo 3 loops de corrección antes de escalar al humano).

**Por qué importa:** Sin validación objetiva, no hay forma de saber si lo que se generó está bien. Los criterios de aceptación son binarios — pasan o no pasan. Esto hace que la calidad del output de la IA sea **medible**.

---

## Dos approaches de implementación

El framework define las 4 etapas y los 7 roles, pero cómo se implementan depende de la herramienta de IA que uses. Hay dos approaches posibles, y el framework soporta ambos.

```
EL PROCESO (4 etapas):

  Requerimiento → [Spec] → [Plan] → [Code] → [Validate] → Done
                    │         │        │           │
                   GATE      GATE   build/test    GATE
                (aprobás?) (aprobás?)           (QA pass?)


CÓMO SE IMPLEMENTA:

  APPROACH 1: Sub-Agentes                 APPROACH 2: Commands / Skills
  ───────────────────────                 ─────────────────────────────
  [Orchestrator]                          [Agente Principal]
    ├─► [Spec Agent] → spec.md              ├─► /infra:spec → spec.md
    │    (contexto propio, tools acotados)  │    (contexto compartido)
    ├─► [Plan Agent] → plan.md              ├─► /infra:plan → plan.md
    │    (contexto propio, tools acotados)  │    (contexto compartido)
    ├─► [Code Agent] → código               ├─► /infra:code → código
    │    (contexto propio, full tools)      │    (contexto compartido)
    └─► [Validate]   → qa-report           └─► /infra:validate → qa-report
         (contexto propio, tools acotados)       (contexto compartido)

  Claude Code ✅ OpenCode 🔶 Kilo Code 🔶
```

### Approach 1: Orquestador con sub-agentes (recomendado)

En herramientas que soportan sub-agentes (Claude Code, OpenCode y Kilo Code), cada rol se ejecuta como un **proceso aislado** con su propio contexto. El orquestador delega tareas a sub-agentes especializados, y cada sub-agente opera con una ventana de contexto limpia y acotada.

```
[Orchestrator] ──delega──► [Spec Agent]  (contexto propio, acotado)
                                │
                           spec.md ──► resultado vuelve al orchestrator

[Orchestrator] ──delega──► [Plan Agent]  (contexto propio, acotado)
                                │
                           plan.md ──► resultado vuelve al orchestrator
```

**Herramientas que lo soportan:** Claude Code, Open Code, Kilo Code (sin paralelismo de tareas)

### Approach 2: Agente principal con commands/skills

En herramientas sin soporte nativo de sub-agentes, el flujo se implementa con **commands o skills** que inyectan el system prompt del rol correspondiente. El agente principal ejecuta cada etapa secuencialmente, en la misma sesión.

```
[Main Agent] ──comando──► /infra:spec  (system prompt del Spec Agent)
                              │
                         spec.md generada en la misma sesión

[Main Agent] ──comando──► /infra:plan  (system prompt del Plan Agent)
                              │
                         plan.md generada en la misma sesión
```

**Herramientas que usan este approach:** Cualquier herramienta de IA sin soporte nativo de sub-agentes (Cursor, Windsurf, Gemini CLI, etc.)

### Variante del Approach 2: sesiones manuales por etapa

Se puede mitigar la acumulación de contexto del Approach 2 si el developer abre manualmente una sesión nueva para cada etapa. Por ejemplo: correr la spec en una sesión, cerrarla, abrir una nueva, pasarle la spec como input, y correr el plan ahí. Esto simula el aislamiento de contexto del Approach 1.

Funciona, pero es más engorroso. El developer actúa como orquestador manual — tiene que manejar los inputs y outputs entre sesiones, copiar artefactos, y coordinar el flujo. Es una opción válida, pero con más fricción comparado con un orquestador automatizado que hace todo eso solo.

### Por qué sub-agentes es el approach preferido

La ventaja más importante es el **manejo eficiente de la ventana de contexto**.

Cuando trabajás todo en una misma sesión (approach 2), la ventana de contexto crece y crece con cada paso. El análisis de negocio de la spec se mezcla con el razonamiento del plan, que se mezcla con el código generado, que se mezcla con la validación. Todo ese historial pesa, consume tokens, y puede sesgar las decisiones del modelo en pasos posteriores.

Con sub-agentes, cada delegación abre un **contexto limpio**. El Spec Agent trabaja con su ventana de contexto acotada — recibe el requerimiento, genera la spec, y devuelve el resultado al orquestador. El orquestador no hereda todo el razonamiento intermedio del Spec Agent, solo el output final. Lo mismo con Plan, Code y Validate. Cada agente opera con el mínimo contexto necesario.

Esto tiene tres beneficios concretos:

**Mejor calidad de output.** El modelo razona mejor cuando el contexto es limpio y relevante. Un Code Agent que solo ve el plan y la tarea actual escribe mejor código que uno que también arrastra el razonamiento del análisis de negocio.

**Menor consumo de tokens.** El contexto del orquestador no crece con cada tarea delegada. Solo recibe el resultado — no los cientos de pasos intermedios de razonamiento.

**Restricción estructural de herramientas.** Un Spec Agent no puede ejecutar código aunque el prompt se lo pidiera — la restricción es de la herramienta, no del comportamiento del modelo. Esto elimina una clase entera de errores accidentales.

### Comparación directa

| Aspecto | Sub-agentes (Approach 1) | Commands/Skills (Approach 2) |
|---------|--------------------------|------------------------------|
| Aislamiento de contexto | Completo — cada agente tiene su propia ventana | Parcial — comparten la misma sesión |
| Consumo de tokens | Eficiente — contexto acotado por agente | Mayor — contexto acumulativo |
| Restricción de herramientas | Estructural — el agente no puede usar tools no habilitados | No disponible — el agente tiene acceso a todo |
| Paralelismo | Sí — tareas independientes pueden correr en background | No — ejecución secuencial |
| Calidad de razonamiento | Mayor — cada rol opera sin ruido de otros pasos | Buena — pero sesgada por contexto previo |
| Flujo SDD | Completo | Completo |
| Artefactos verificables | Sí | Sí |
| Identidad de roles | Sí | Sí |

**Conclusión:** Ambos approaches siguen el mismo flujo SDD y producen los mismos artefactos. La diferencia está en la calidad operativa. Cuando la herramienta lo permite, sub-agentes es siempre la opción preferida. Cuando no, commands/skills es un trade-off aceptable que mantiene la estructura y la disciplina del proceso.

### Los 7 agentes del framework

| Agente | Rol | Responsabilidad |
|--------|-----|-----------------|
| **Orchestrator** | Coordinador | Recibe el requerimiento, delega a los agentes especializados, gestiona los gates de aprobación y el flujo completo |
| **Spec** | Business Analyst | Traduce requerimientos en especificaciones funcionales con criterios de aceptación |
| **Plan** | Arquitecto | Diseña el plan de implementación con tareas, dependencias y decisiones técnicas |
| **Code** | Developer | Implementa las tareas del backlog siguiendo el plan |
| **Debug** | Debugger | Diagnostica y resuelve problemas con un ciclo hipótesis → validación → fix |
| **Validate** | QA Specialist | Valida la implementación contra los criterios de aceptación de la spec |
| **Context** | Analista | Genera y mantiene el Memory Bank del proyecto |

### Cómo funciona el orquestador

El orquestador es el punto de entrada. Cuando recibe un requerimiento:

```
Requerimiento
     │
     ▼
[Orchestrator] ──delega──► [Spec Agent] ──► spec.md
     │                                          │
     │◄── resultado + gate de aprobación ───────┘
     │
     │  (si humano aprueba)
     │
     ├──delega──► [Plan Agent] ──► plan.md + tasks.json
     │                                       │
     │◄── resultado + gate de aprobación ────┘
     │
     │  (si humano aprueba)
     │
     ├──delega──► [Code Agent] ──► implementación
     │               │
     │               ├── (si falla) ──► [Debug Agent]
     │               │
     │◄── tareas completadas ────────────┘
     │
     ├──delega──► [Validate Agent] ──► qa-report.md
     │                                      │
     │◄── resultado ────────────────────────┘
     │
     ▼
   DONE (o loop de corrección si FAIL)
```

En herramientas que soportan sub-agentes (Claude Code, OpenCode, Kilo Code), el orquestador delega usando la herramienta de agentes, creando procesos aislados. En herramientas sin soporte nativo, el flujo se implementa con commands/workflows que inyectan el system prompt del rol correspondiente.

---

## Contexto del proyecto: Memory Bank

### Qué es

El Memory Bank es un sistema de memoria persistente que le da al asistente de IA contexto sobre el proyecto entre sesiones. Son dos archivos markdown que se leen automáticamente al inicio de cada sesión:

### `project.md` — Contexto de negocio

Define qué es el proyecto, para qué sirve, y cuáles son sus restricciones:

- Descripción del proyecto
- Contexto de negocio (industria, mercado, usuarios)
- Plataforma (Adobe Commerce, VTEX, Shopify, custom)
- Key constraints (limitaciones de negocio, técnicas, regulatorias)

### `technical.md` — Contexto técnico

Define cómo funciona el proyecto por dentro:

- Stack tecnológico (lenguaje, framework, base de datos)
- Arquitectura (monolito, microservicios, serverless)
- Módulos clave y extensiones
- Integraciones con servicios externos
- Convenciones de código (naming, estructura, patterns)
- Dev workflow (branching, CI/CD, deploys)
- Key decisions (decisiones de arquitectura ya tomadas)

### Por qué importa

Sin Memory Bank, cada sesión con la IA empieza de cero. El asistente no sabe si estás trabajando con Adobe Commerce o con un CLI en Go. No conoce tus convenciones ni tu arquitectura. El Memory Bank elimina esa brecha — cada agente arranca con contexto completo del proyecto.

### Comportamiento no destructivo

Los archivos del Memory Bank nunca se sobreescriben automáticamente. Si ya existen, se preservan. Esto protege el contenido que el equipo ya personalizó.

---

## Sesiones: trazabilidad completa

Cada feature o requerimiento que pasa por el framework se trackea en una **sesión**. Las sesiones viven en `.infracode/sessions/{id}/` y contienen todos los artefactos generados:

```
.infracode/sessions/feat-checkout-cuotas-a1b2c3d4/
├── session.json    ← Estado de la sesión (spec_ready, plan_ready, coding, done)
├── spec.md         ← Especificación funcional
├── plan.md         ← Plan de implementación
├── tasks.json      ← Backlog de tareas con estado y evidencia
└── qa-report.md    ← Reporte de validación
```

Esto da trazabilidad completa: podés auditar qué se decidió, por qué, y con qué resultado. Las sesiones se pueden retomar entre conversaciones — si cerrás el IDE y volvés mañana, el orquestador te ofrece retomar donde dejaste.

---

## InfraCode CLI: la implementación

InfraCode CLI es la herramienta que hace que todo esto funcione en la práctica. Es un binario en Go, sin dependencias externas, que se instala con un solo comando y configura todo lo necesario.

### Qué hace

- **Configura agentes especializados** para la herramienta de IA que uses (Claude Code, OpenCode, Kilo Code)
- **Inyecta contexto por plataforma** — si trabajás con Adobe Commerce, los agentes saben cómo crear módulos, qué estructura seguir, qué APIs usar
- **Inicializa el Memory Bank** del proyecto
- **Gestiona skills** — capacidades adicionales que le dan a los agentes conocimiento específico
- **Conecta con el Infra AI Gateway** para centralizar el acceso a modelos
- **Trackea sesiones** de desarrollo

### Herramientas soportadas

InfraCode usa un patrón de adaptadores que permite soportar múltiples herramientas de IA. Cada adaptador traduce las definiciones universales de agentes al formato específico de cada herramienta.

| Herramienta | Cómo se implementan los agentes | Sub-agentes nativos |
|-------------|--------------------------------|---------------------|
| **Claude Code** | `.claude/agents/infra-*.md` (Markdown con frontmatter) | Sí — Agent tool con proceso aislado |
| **OpenCode** | `.opencode/agents/infra-*.md` (Markdown) | Sí — agents con contexto propio |
| **Kilo Code** | `.kilocodemodes` (YAML con modos) | Sí — `new_task` para delegación |

### Comandos principales

| Comando | Qué hace |
|---------|----------|
| `infracode init` | Inicializa un proyecto con el framework (wizard interactivo) |
| `infracode add <tool>` | Agrega soporte para una herramienta de IA adicional |
| `infracode sync` | Sincroniza los agentes con la versión actual del CLI |
| `infracode status` | Muestra el estado del proyecto (herramientas, Memory Bank, sesiones) |
| `infracode update` | Actualiza el CLI a la última versión |
| `infracode gateway` | Configura el Infra AI Gateway (routing de modelos) |
| `infracode skills` | Gestiona skills de agentes (buscar, instalar, actualizar) |
| `infracode preset add` | Instala presets remotos (configuraciones por plataforma) |

### Flujo de inicialización

```bash
cd mi-proyecto
infracode init
```

El wizard te guía por tres decisiones:

1. **Herramientas de IA** — cuáles querés configurar (Claude Code, OpenCode, Kilo Code)
2. **Plataforma** — cuál es la plataforma del proyecto (Adobe Commerce, VTEX, Shopify, o ninguna)
3. **Memory Bank** — si querés inicializar el contexto persistente del proyecto

Después de eso, InfraCode:

- Crea los directorios de configuración por herramienta (`.claude/`, `.opencode/`, etc.)
- Instala los 7 agentes especializados con sus system prompts
- Instala los workflow commands (`/infra:spec`, `/infra:plan`, etc.)
- Instala reglas de comportamiento
- Inicializa el Memory Bank (si se seleccionó)
- Genera `.infracode/config.json` con la configuración del proyecto
- Actualiza `.gitignore`

---

## Workflows disponibles

### Flujo principal (SDD)

| Workflow | Agente | Cuándo usarlo |
|----------|--------|---------------|
| `/infra:spec` | Spec Agent | Cuando tenés un requerimiento nuevo y querés generar la spec |
| `/infra:plan` | Plan Agent | Cuando la spec está aprobada y querés generar el plan |
| `/infra:code` | Code Agent | Cuando el plan está aprobado y querés implementar |
| `/infra:validate` | Validate Agent | Cuando la implementación está lista y querés validar |

### Workflow de soporte

| Workflow | Rol | Cuándo usarlo |
|----------|-----|---------------|
| `/infra:debug` | Debugger | Cuando hay un error de build, test failure, o bug |

### Memory Bank

| Workflow | Cuándo usarlo |
|----------|---------------|
| `/infra:memory-bank-init` | Primera vez — genera project.md y technical.md analizando el código |
| `/infra:memory-bank-update` | Después de cambios significativos en el proyecto |
| `/infra:memory-bank-migrate` | Para migrar de un formato viejo del Memory Bank al nuevo |

---

## Cómo el framework reduce la variabilidad

El "random" de la IA se acumula en cada paso. Si la spec es vaga, el plan será vago, el código impredecible, y la validación inútil. El framework reduce esa variabilidad en cada etapa:

```
Sin framework:
  Requerimiento vago → IA improvisa → código impredecible → sin validación
  Variabilidad: ████████████████████ (alta en cada paso, se acumula)

Con framework:
  Requerimiento → Spec (acotada) → Plan (acotado) → Code (acotado) → Validate (binario)
  Variabilidad:   ████               ███              ██               █
                  (cada etapa reduce el margen de error de la siguiente)
```

La clave es que el output de cada etapa es el input controlado de la siguiente. Si la spec tiene 20 criterios de aceptación específicos, el plan no puede inventar features nuevos. Si el plan define 12 tareas con archivos específicos, el Code Agent no puede salir del scope. Y si la validación compara contra los criterios de la spec, el resultado es objetivo.

---

## Beneficios medibles

**Consistencia.** El mismo requerimiento produce la misma estructura de output, independientemente de quién lo ejecute. Un junior y un senior usan el mismo proceso.

**Medibilidad.** Los criterios de aceptación son binarios — PASS o FAIL. Podés medir el acceptance criteria pass rate por sesión, por equipo, por plataforma.

**Trazabilidad.** Cada sesión produce artefactos auditables: spec, plan, tasks, QA report. Podés reconstruir qué se decidió, por qué, y con qué resultado.

**Escalabilidad.** El framework funciona igual para un proyecto que para veinte. InfraCode inyecta contexto por plataforma, por proyecto, y por equipo.

**Velocidad.** Las tareas sin dependencias se ejecutan en paralelo. El plan reduce el tiempo de ida y vuelta ("hacé esto... no, no así, así"). La spec elimina la ambigüedad desde el arranque.

**Control humano.** Los gates de aprobación aseguran que el humano decide en cada etapa si avanzar, ajustar, o frenar. La IA no opera en piloto automático.

---

## Estructura de archivos del framework

Cuando InfraCode configura un proyecto, genera la siguiente estructura:

```
mi-proyecto/
│
├── .infracode/                         ← Framework core
│   ├── config.json                     ← Configuración del proyecto
│   ├── memory-bank/
│   │   ├── project.md                  ← Contexto de negocio
│   │   └── technical.md                ← Contexto técnico
│   └── sessions/                       ← Sesiones de desarrollo (gitignored)
│       └── feat-xyz-a1b2c3d4/
│           ├── session.json
│           ├── spec.md
│           ├── plan.md
│           ├── tasks.json
│           └── qa-report.md
│
├── .claude/                            ← Config para Claude Code
│   ├── agents/                         ← Agentes especializados
│   │   ├── infra-orchestrator.md
│   │   ├── infra-spec.md
│   │   ├── infra-plan.md
│   │   ├── infra-code.md
│   │   ├── infra-debug.md
│   │   ├── infra-validate.md
│   │   └── infra-context.md
│   ├── commands/                       ← Workflow commands
│   │   ├── infra:spec.md
│   │   ├── infra:plan.md
│   │   ├── infra:code.md
│   │   ├── infra:validate.md
│   │   ├── infra:debug.md
│   │   ├── infra:memory-bank-init.md
│   │   ├── infra:memory-bank-update.md
│   │   └── infra:memory-bank-migrate.md
│   ├── rules/                          ← Reglas de comportamiento
│   │   └── memory-bank.md
│   └── skills/                         ← Skills del agente
│
├── .opencode/                          ← Config para OpenCode (si se instaló)
│   ├── agents/
│   └── commands/
│
└── .kilocodemodes                      ← Config para Kilo Code (si se instaló)
```

---

## Glosario

| Término | Definición |
|---------|-----------|
| **SDD** | Spec-Driven Development — la metodología de las 4 etapas |
| **Spec** | Especificación funcional con criterios de aceptación medibles |
| **Plan** | Plan de implementación con tareas, dependencias y decisiones técnicas |
| **Gate** | Punto de aprobación humana entre etapas |
| **Memory Bank** | Sistema de contexto persistente del proyecto (project.md + technical.md) |
| **Sesión** | Instancia de un feature pasando por el flujo SDD, con todos sus artefactos |
| **Agente** | Rol especializado de IA con responsabilidad, herramientas y restricciones definidas |
| **Orquestador** | Agente coordinador que delega a los agentes especializados |
| **Skill** | Capacidad explícita y reutilizable que un agente puede ejecutar |
| **Preset** | Paquete de configuración para una plataforma o tecnología específica |
| **Adapter** | Módulo que traduce las definiciones del framework al formato de una herramienta de IA |
| **InfraCode CLI** | Herramienta de línea de comandos que implementa y configura el framework |

---

← Ver [comandos](commands.md) · Ver [onboarding paso a paso](onboarding.md) · Ver [arquitectura del CLI](architecture.md) · Ver [por qué sub-agentes](why-subagents-architecture.md)
