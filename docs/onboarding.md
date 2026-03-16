# Infra AI Framework — Guía de Onboarding

> Guía para desarrolladores que usan **InfraCode CLI** con el flujo de **Spec-Driven Development**.

---

## ¿Qué es esto?

El Infra AI Framework es un sistema de workflows que convierte tu asistente de IA en un equipo de desarrollo completo. En vez de pedirle al AI "haceme esto" y esperar que resuelva todo de una vez, el framework impone un proceso de ingeniería disciplinado:

1. **Primero se define qué** (Spec)
2. **Después se diseña cómo** (Plan)
3. **Luego se implementa** (Code)
4. **Finalmente se valida** contra criterios objetivos (Validate)

Cada etapa tiene su workflow, sus reglas, y un **gate de aprobación** donde vos decidís si avanzar o ajustar.

---

## Mapa del sistema

```
┌─────────────────────────────────────────────────────────────────┐
│                    FLUJO PRINCIPAL                               │
│                                                                  │
│   Requerimiento ──→ Spec ──→ Plan ──→ Code ──→ Validate ──→ Done │
│                      │        │        │          │              │
│                      ▼        ▼        ▼          ▼              │
│                    GATE     GATE    build/test   GATE            │
│                  (aprobás?) (aprobás?)  ↓       (QA PASS?)       │
│                                     Debug ◄──┘                   │
│                                     (si falla)                   │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                 WORKFLOW DE SOPORTE                              │
│                                                                  │
│   /infra:debug     → Diagnosticar bugs y problemas de build      │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                 MEMORIA DEL PROYECTO                             │
│                                                                  │
│   /infra:memory-bank-init    → Inicializar contexto del proyecto │
│   /infra:memory-bank-update  → Actualizar después de cambios     │
│   /infra:memory-bank-migrate → Migrar formato viejo a nuevo      │
└─────────────────────────────────────────────────────────────────┘
```

---

## Primeros pasos

### 1. Inicializar el Memory Bank (recomendado)

El Memory Bank le da contexto persistente sobre tu proyecto al AI. Sin él, cada conversación empieza de cero.

Abrí tu herramienta de IA con el orquestador (ver más abajo) y ejecutá:

```
/infra:memory-bank-init
```

El agente analiza tu código, detecta stack/plataforma, y genera `project.md` y `technical.md`. Vos aprobás el contenido antes de que se guarde.

> **Proyecto nuevo**: podés describir lo que querés construir directamente en el comando:
> ```
> /infra:memory-bank-init — API REST en Go con PostgreSQL para gestión de usuarios
> ```

### 2. Iniciar el orquestador

Una vez configurado el proyecto con `infracode init`, abrí tu herramienta de IA con el agente orquestador:

**Claude Code:**

```bash
claude --agent infra-orchestrator
```

**OpenCode / Kilo Code:**

Seleccioná el modo/agente `infra-orchestrator` desde la interfaz de tu herramienta.

### 3. Describir tu requerimiento

Simplemente describí lo que querés construir en lenguaje natural:

```
Quiero agregar un endpoint REST que permita listar todos
los usuarios filtrados por rol y paginados.
```

El orquestador detecta que es un feature nuevo y te guía por el flujo completo.

---

## El flujo completo: paso a paso

### Etapa 1: Spec (`/infra:spec`)

**Rol**: Business Analyst — define **qué** construir.

**Qué pasa**:
- El AI toma tu requerimiento y lo transforma en una **spec funcional** con criterios de aceptación medibles
- Identifica ambigüedades y te pregunta antes de asumir
- Define edge cases, supuestos y riesgos

**Lo que se genera**: `.infracode/sessions/{id}/spec.md` con:
- Requerimiento original (literal, sin parafrasear)
- Objetivo y alcance
- Flujo principal y flujos alternativos
- Criterios de aceptación (formato Given/When/Then)
- Edge cases, supuestos, riesgos

**El gate**:

> 🚦 "¿Aprobás esta spec o querés ajustar algo?"

- Si aprobás → avanza a Plan
- Si pedís cambios → re-genera la spec con tu feedback

**Ejemplo de criterio de aceptación**:
```
❌ Malo:  "El endpoint lista usuarios"
✅ Bueno: "Dado un request GET /api/users?role=admin&page=2&limit=10,
          cuando hay 25 usuarios admin, entonces retorna 10 usuarios
          con total=25, page=2, totalPages=3"
```

---

### Etapa 2: Plan (`/infra:plan`)

**Rol**: Arquitecto de Software — define **cómo** construirlo.

**Qué pasa**:
- Lee la spec aprobada + el código existente de tu proyecto
- Elige una estrategia técnica con trade-offs documentados
- Identifica archivos a crear/modificar
- Descompone todo en **tareas atómicas** con dependencias

**Lo que se genera**:
- `.infracode/sessions/{id}/plan.md` — estrategia, decisiones de arquitectura, riesgos
- `.infracode/sessions/{id}/tasks.json` — backlog de tareas con dependencias y criterios por tarea

**El gate**:

> 🚦 "¿Aprobás este plan? [N tareas, M paralelas]"

**Ejemplo de tarea en tasks.json**:
```json
{
  "id": "T01",
  "title": "Crear handler GET /api/users con filtro por rol",
  "status": "pending",
  "depends_on": [],
  "parallel": true,
  "files": ["internal/handler/users.go"],
  "acceptance_criteria": [
    "Handler registrado en router",
    "Acepta query params role, page, limit"
  ]
}
```

---

### Etapa 3: Code (`/infra:code`)

**Rol**: Developer — ejecuta las tareas del backlog.

**Qué pasa**:
- Lee la tarea del `tasks.json`, el plan y la spec
- Implementa el código siguiendo las decisiones del plan
- Escribe tests para la funcionalidad
- Corre build + tests + lint para verificar
- Actualiza `tasks.json` con evidencia de ejecución

**Lo que se genera**: el código, tests, y evidencia en `tasks.json`:
```json
{
  "id": "T01",
  "status": "completed",
  "evidence": {
    "build": "pass",
    "tests": "pass (5/5)",
    "lint": "pass"
  }
}
```

**Si el build o tests fallan**, se activa automáticamente `/infra:debug`.

---

### Etapa 4: Validate (`/infra:validate`)

**Rol**: QA Specialist — valida la implementación contra la spec.

**Qué pasa**:
- Lee la spec y revisa el código producido, criterio por criterio
- Corre build + tests + lint completos
- Hace un mini code review
- Clasifica hallazgos por severidad (critical/major/minor)

**Lo que se genera**: `.infracode/sessions/{id}/qa-report.md` con:
- Resultado global: **PASS** o **FAIL**
- Tabla de ACs con estado y evidencia
- Validaciones técnicas
- Hallazgos y correcciones requeridas

**Criterios de PASS**:
- ✅ Todos los ACs cumplidos
- ✅ Build pasa
- ✅ Todos los tests pasan
- ✅ Sin hallazgos críticos

**Si es FAIL**:

> 🚦 "¿Corregimos los issues?"

Vuelve a Code con el feedback del QA. Máximo 3 loops de corrección antes de escalar.

---

## Workflow de soporte

### `/infra:debug` — Debugger

**Cuándo**: un build falla, un test no pasa, hay un bug.

**Cómo funciona**:
1. Reproduce el error
2. Formula 2-3 hipótesis ordenadas por probabilidad
3. Valida cada una contra el código
4. Aplica un fix mínimo y verifica

Se activa automáticamente cuando `/infra:code` detecta un build/test failure. También podés invocarlo directamente ante cualquier problema.

---

## Cómo interactuar

### Workflows disponibles

```
/infra:spec              → generar spec de un requerimiento nuevo
/infra:plan              → generar plan técnico de una spec aprobada
/infra:code              → ejecutar tareas del backlog
/infra:validate          → correr QA sobre lo implementado
/infra:debug             → diagnosticar un problema

/infra:memory-bank-init  → inicializar contexto del proyecto
/infra:memory-bank-update → actualizar después de cambios significativos
```

### Flujo natural (recomendado)

También podés simplemente **describir lo que necesitás** y el orquestador te guía:

| Lo que decís | Lo que pasa |
|---|---|
| "Quiero agregar paginación al endpoint de usuarios" | Arranca el flujo Spec → Plan → Code → Validate |
| "El build rompe con error X" | Activa `/infra:debug` |

### Los gates son tu control

En cada gate:

- **"Aprobás"** → el flujo avanza
- **"Cambiá X"** → se re-genera con tu feedback
- **"Pará"** → todo se detiene

> 💡 El costo de parar es bajo; el costo de re-hacer es alto. Mejor ajustar en la spec que corregir en el código.

---

## Sesiones

Cada feature es una **sesión** con un ID único (ej: `feat-checkout-cuotas-a1b2c3d4`). Las sesiones:

- Viven en `.infracode/sessions/{id}/`
- Son la fuente de verdad del estado del flujo
- Se pueden retomar entre conversaciones
- Contienen todos los artefactos: spec, plan, tasks, qa-report

### Retomar una sesión

Si hay sesiones abiertas, el orquestador te ofrece retomar:

```
Encontré sesiones activas:
1. feat-checkout-cuotas-a1b2c3d4 (plan listo, 3/5 tareas completadas)
2. fix-login-redirect-e5f6g7h8 (spec lista, esperando plan)

¿Cuál retomamos?
```

---

## Ejemplo end-to-end

Querés agregar un comando `users list` a un CLI en Go.

### 1. Describís el requerimiento

```
Necesito agregar un comando "users list" al CLI que se conecte
a la API y muestre una tabla con los usuarios.
```

### 2. Spec (el AI genera, vos aprobás)

El AI produce una spec con:
- `AC-01`: `Dado el comando "infracode users list", cuando la API retorna 3 usuarios, entonces se muestra una tabla con Name, Email, Role`
- Edge cases: API no disponible, 0 usuarios, error de auth
- Vos aprobás ✅

### 3. Plan (el AI diseña, vos aprobás)

El AI produce:
- 4 tareas: T01 (struct User + API client), T02 (comando Cobra), T03 (tabla), T04 (tests)
- T01 y T02 son paralelas, T03 depende de T01
- Vos aprobás ✅

### 4. Code (el AI implementa)

El AI ejecuta T01 → T02 → T03 → T04:
- Escribe el código siguiendo patterns existentes
- Corre build y tests después de cada tarea
- Si algo rompe → entra en `/infra:debug` automáticamente

### 5. Validate (el AI valida)

```
AC-01: PASS — Test TestUsersList_ThreeUsers verifica tabla con 3 filas
AC-02: PASS — Test TestUsersList_APIError muestra error descriptivo
Build: PASS
Tests: PASS (12/12)
Resultado: QA PASS ✅
```

### 6. Done

```
Feature completado. QA PASS. Listo para PR.
```

---

## Tips para sacarle provecho

1. **Sé claro en el requerimiento**: mientras más contexto des al inicio, mejor la spec.
2. **Leé los artefactos**: la spec, plan y QA report están en `.infracode/sessions/`. Son la documentación del proceso.
3. **Usá los gates**: si algo no te cierra, pedí cambios. El AI itera.
4. **El Memory Bank es tu amigo**: inicializalo temprano y actualizalo cuando el proyecto cambia. Mejora la calidad de todas las respuestas.
5. **Una sesión = un feature**: no mezcles múltiples features en una sesión.

---

← [README](../README.md) · [InfraAI Framework](framework.md) · [Comandos](commands.md) · [Conceptos](concepts.md)
