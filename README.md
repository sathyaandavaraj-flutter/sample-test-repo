# sample-test-repo
Sample agent skills repository (created for ISS-144)

## Installing Claude Code in the CLI

Claude Code is Anthropic's agentic coding tool that runs in your terminal. Use one of the methods below to install the `claude` command-line interface.

### Requirements

- A supported OS: macOS 13.0+, Windows 10 1809+, or a modern Linux distribution (Ubuntu 20.04+, Debian 10+, Alpine 3.19+).
- 4 GB+ RAM and an internet connection.
- A shell such as Bash, Zsh, PowerShell, or CMD.

### Install

**Native installer (recommended)**

macOS, Linux, WSL:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Homebrew (macOS/Linux)**

```bash
brew install --cask claude-code
```

**WinGet (Windows)**

```powershell
winget install Anthropic.ClaudeCode
```

**npm (requires Node.js 18+)**

```bash
npm install -g @anthropic-ai/claude-code
```

> Do not use `sudo npm install -g`, as this can lead to permission issues and security risks.

### Verify the installation

```bash
claude --version
```

For a more detailed check of your installation and configuration, run:

```bash
claude doctor
```

### Get started

Open a terminal in the project you want to work in and start Claude Code:

```bash
claude
```

The first time you run `claude`, follow the browser prompts to authenticate. Claude Code requires a Pro, Max, Team, Enterprise, or Console account.

For more details, see the [official setup documentation](https://docs.anthropic.com/en/docs/claude-code/setup).
