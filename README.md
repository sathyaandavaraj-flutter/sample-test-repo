# sample-test-repo
Sample agent skills repository (created for ISS-144)

## Installing Claude Code in the CLI

Claude Code is Anthropic's agentic coding tool that runs in your terminal.

### Prerequisites

- A supported OS: macOS, Linux, or Windows (via WSL or PowerShell)
- [Node.js 18 or newer](https://nodejs.org/) (required for the npm install method)
- An active Anthropic account

### Install with npm

```bash
npm install -g @anthropic-ai/claude-code
```

### Install with the native installer

macOS, Linux, or WSL:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

### Verify the installation

```bash
claude --version
```

### Get started

Navigate to your project directory and launch Claude Code:

```bash
cd your-project
claude
```

The first time you run `claude`, you'll be prompted to sign in and complete a one-time authentication. For more details, see the [official Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code/overview).
