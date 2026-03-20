# Primeros pasos

---

## 1. Configurar el Gateway

El Gateway centraliza el acceso a los modelos de IA. Configuralo **una sola vez** en tu máquina antes de empezar.

**Para Claude Code** (seleccionás 2 modelos: uno principal y uno para sub-agentes):

```bash
infracode gateway --claude
```

> Para volver a tu suscripción personal de Claude, remové el gateway y reabrí la sesión:
> ```bash
> infracode gateway remove --claude
> ```

**Para Open Code** (un solo modelo):

```bash
infracode gateway --opencode
```

> **Importante:** Después de cualquier cambio en el gateway, cerrá la sesión de Claude y reabrila para que los cambios tomen efecto:
> ```bash
> claude --resume <session-id>
> ```

---

## 2. Inicializar el proyecto

Desde el directorio raíz de tu proyecto:

```bash
cd mi-proyecto
infracode init
```

El wizard te guía para seleccionar herramientas de IA, plataforma y configurar el Memory Bank.

---

## 3. Configurar el Memory Bank

El Memory Bank le da al asistente contexto persistente sobre tu proyecto entre sesiones.

Primero, abrí tu herramienta de IA con el orquestador (ver paso 4). Una vez dentro, ejecutá el comando:

**Si es un proyecto existente** (el agente analiza el código):
```
/infra-memory-bank-init
```

**Si es un proyecto nuevo** (le describís qué querés construir):
```
/infra-memory-bank-init — descripción del proyecto, tech stack, objetivos
```

Esto genera `.infracode/memory-bank/project.md` y `technical.md` que los agentes leen automáticamente en cada sesión.

Una vez generados, **revisá ambos archivos**. Podés editarlos a mano o pedirle al agente que haga ajustes. El objetivo es que reflejen con precisión tu proyecto: stack, arquitectura, decisiones clave, convenciones. Toda información que sea relevante para que la IA trabaje bien en contexto va ahí. Lo que no aplica a tu proyecto, sacalo.

---

## 4. Iniciar el orquestador

Una vez configurado el proyecto, abrí tu herramienta de IA con el agente orquestador como punto de entrada.

**Claude Code:**

```bash
claude --agent infra-orchestrator
```

> Esto es clave. El orquestador es el coordinador del flujo — recibe tu requerimiento y delega a los agentes especializados (Spec, Plan, Code, Validate).

**OpenCode / Kilo Code:**

Abrí la herramienta y el orquestador estará disponible como modo/agente principal.

---

## El flujo de trabajo

Una vez dentro del orquestador, el flujo es:

```
Tu requerimiento
      │
      ▼
  [Spec]   →  spec.md con criterios de aceptación
      │            ↑ gate: vos aprobás
      ▼
  [Plan]   →  plan.md + tasks.json con tareas atómicas
      │            ↑ gate: vos aprobás
      ▼
  [Code]   →  implementación + tests
      │
      ▼
  [Validate] →  qa-report.md (PASS / FAIL por criterio)
```

Cada etapa produce artefactos verificables. Vos aprobás antes de avanzar.

---

## Actualizar el CLI

```bash
infracode update
```
