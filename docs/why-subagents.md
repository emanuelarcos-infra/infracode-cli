# Por qué el framework usa sub-agentes dedicados

**Tipo**: Decisión de arquitectura / Documentación conceptual
**Fecha**: 2026-03-10
**Audiencia**: Implementadores del framework, contribuidores, equipos que adoptan el approach

---

## El problema que resuelve

Un asistente de IA ejecutando una tarea compleja —como ir de un requerimiento a código deployable— necesita cumplir múltiples roles: analista, arquitecto, desarrollador, QA. Si un solo agente los ejecuta todos dentro de la misma conversación, ocurren dos problemas:

1. **Contaminación de contexto**: el razonamiento de análisis de negocio se mezcla con el de generación de código. Las decisiones anteriores sesgan las siguientes de formas difíciles de predecir.
2. **Comportamiento no acotado**: un agente con acceso a todos los tools puede tomar acciones fuera del scope esperado para su rol actual.

La solución del framework es **un agente por responsabilidad**, ejecutado como sub-proceso aislado.

---

## Fundamento técnico

### Aislamiento de contexto

Cada sub-agente corre con su propio historial de conversación, separado del agente orquestador. El orquestador recibe un único mensaje de vuelta —el output final— sin el ruido de todos los pasos intermedios.

Esto produce:
- Razonamiento más limpio en cada rol
- Outputs más predecibles y auditables
- El contexto del orquestador no crece con cada tarea delegada

### Restricción de tools

Un agente dedicado expone únicamente los tools necesarios para su rol:

| Agente | Tools habilitados |
|--------|------------------|
| `infra-spec` | Read, Write, Glob, Grep |
| `infra-plan` | Read, Write, Glob, Grep, Bash |
| `infra-code` | Read, Edit, Write, Bash, Glob, Grep |
| `infra-validate` | Read, Edit, Write, Bash, Glob, Grep |
| `infra-debug` | Read, Edit, Bash, Glob, Grep |
| `infra-orchestrator` | Agent + todos los anteriores |

Un `infra-spec` **no puede ejecutar código** aunque el prompt se lo pidiera — la restricción es estructural, no de comportamiento. Esto elimina una clase entera de errores accidentales.

### Paralelismo nativo

Sub-agentes independientes pueden correr en background de forma concurrente. Un orquestador puede delegar validación y documentación en paralelo sin bloquear su hilo principal.

### Identidad declarativa

Al llamar `subagent_type: "infra-spec"`, el código comunica **qué rol se está ejecutando**, no solo qué prompt se está usando. Esto hace el flujo legible para humanos y para otros agentes que lo coordinen.

---

## Por qué esto le da seriedad al framework

Un framework de desarrollo que usa agentes como si fueran un único chatbot omnisciente no es confiable en producción. Los principios que aplican a software aplican a agentes:

- **Separation of concerns**: cada agente tiene una responsabilidad bien definida
- **Least privilege**: cada agente accede solo a lo que necesita
- **Auditabilidad**: el output de cada etapa es verificable antes de pasar a la siguiente
- **Composabilidad**: los agentes son intercambiables si respetan la interfaz (inputs/outputs)

Esto alinea el framework con prácticas de ingeniería de software establecidas, no con prompting ad-hoc.

---

## Replicar el approach en otras herramientas

No todas las herramientas de IA tienen soporte nativo para sub-agentes tipados. El approach puede aproximarse con los mecanismos disponibles en cada herramienta.

### Herramientas con sub-agentes nativos

**Claude Code** soporta agentes dedicados via `.claude/agents/*.md`. Cada archivo define el rol, los tools habilitados, el modelo y el sistema de permisos. Es el target primario del framework.

### Herramientas sin sub-agentes: el approach equivalente

En herramientas como **OpenCode**, **Kilo Code**, **Cursor** o **Windsurf**, el equivalente funcional es un **comando/workflow con el system prompt del rol**.

El mecanismo es diferente, pero el efecto es similar:

```
Claude Code / OpenCode              Antigravity (Cursor/Windsurf)
─────────────────────────────────   ─────────────────────────────────
Agent tool con subagent_type        /infra-spec  (command/workflow)
  → proceso separado                  → mismo contexto, prompt expandido
  → tools restringidos                → sin restricción de tools nativa
  → contexto aislado                  → contexto compartido
  → paralelismo posible               → ejecución secuencial
```

**¿Qué se pierde?**

- El aislamiento de contexto es parcial: el historial de la conversación principal contamina la ejecución del rol
- No hay restricción estructural de tools: el agente puede ejecutar acciones fuera del scope si el modelo lo decide
- No hay paralelismo: los roles se ejecutan secuencialmente

**¿Qué se mantiene?**

- La identidad del rol: el agente sabe qué responsabilidad tiene en este momento
- El flujo SDD: spec → plan → code → validate se respeta
- La predictibilidad de outputs: si el prompt está bien definido, el output sigue siendo consistente
- La auditabilidad por etapa: cada paso produce un artefacto verificable

### Tabla de equivalencias

| Concepto | Claude Code | OpenCode / Kilo | Cursor / Windsurf |
|----------|-------------|-----------------|-------------------|
| Rol dedicado | `.claude/agents/infra-spec.md` | `.opencode/agents/infra-spec.md` | Prompt en command |
| Invocación | `Agent tool (subagent_type)` | `@infra-spec` | `/infra-spec` |
| Aislamiento | Proceso separado | Contexto compartido | Contexto compartido |
| Restricción tools | Frontmatter `tools:` | No disponible | No disponible |
| Paralelismo | Sí (background) | No | No |

---

## Recomendación para equipos

**Si la herramienta soporta sub-agentes**: usarlos. La inversión en definir roles con tools restringidos y contexto aislado paga en predictibilidad y seguridad operativa.

**Si la herramienta no los soporta**: usar commands/workflows con el mismo contenido de system prompt. Se pierde aislamiento y restricción de tools, pero se mantiene el flujo y la identidad de roles. Es un trade-off aceptable para equipos que quieren adoptar SDD sin cambiar de herramienta.

**El framework soporta ambos**: los agentes de cada herramienta están definidos en su formato nativo en `agents/{tool}/`. No hay transformaciones en runtime — cada herramienta tiene su fuente de verdad.

---

## Notas para contribuidores

Al agregar soporte para una nueva herramienta al framework:

1. Identificar si soporta sub-agentes con restricción de tools
2. Si sí: definir agentes con el menor scope de tools posible
3. Si no: definir commands/workflows con el system prompt equivalente
4. Documentar las limitaciones del approach para esa herramienta
5. Garantizar que el flujo SDD (spec → plan → code → validate) sea reproducible
