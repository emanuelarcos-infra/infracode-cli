# Infra.Code CLI

CLI del **Infra AI Framework** de Infracommerce. Configura agentes de IA especializados en tus proyectos con un solo comando — y los conecta al flujo de Spec-Driven Development.

[Infra AI Framework](docs/framework.md) | [Primeros pasos](docs/getting-started.md) | [Onboarding](docs/onboarding.md) | [Conceptos](docs/concepts.md) | [Comandos](docs/commands.md) | [Memory Bank](docs/memory-bank.md) | [¿Por qué sub-agentes?](docs/why-subagents.md)

---

## Instalación

### macOS y Linux

```bash
curl -fsSL https://github.com/ifclatam/infracode-releases/releases/latest/download/install.sh | bash
```

### Windows

**Git Bash (recomendado):**

Requiere [Git for Windows](https://git-scm.com/download/win). Abrí **Git Bash** y ejecutá:

```bash
curl -fsSL https://github.com/ifclatam/infracode-releases/releases/latest/download/install-windows.sh | bash
```

El script detecta automáticamente el entorno (Git Bash, MSYS2, Cygwin o WSL), resuelve las rutas de Windows e instala en `%LOCALAPPDATA%\Programs\infracode`.

**PowerShell (alternativa):**

```powershell
irm https://github.com/ifclatam/infracode-releases/releases/latest/download/install.ps1 | iex
```

> Si el antivirus bloquea el script (error `ScriptContainedMaliciousContent`), usá el método de Git Bash de arriba.
> Si obtenés un error de Execution Policy: `Set-ExecutionPolicy RemoteSigned -Scope CurrentUser`

> En terminales sin soporte TUI (cmd.exe, ConHost), el CLI usa modo texto plano automáticamente. También podés forzarlo con `infracode --no-tui init` o `INFRACODE_NO_TUI=1`.

### Verificar instalación

```bash
infracode --version
```
