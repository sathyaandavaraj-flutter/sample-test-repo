# sample-test-repo
Sample agent skills repository (created for ISS-144)

## Installing Claude Code in the CLI

[Claude Code](https://docs.anthropic.com/en/docs/claude-code/overview) is Anthropic's agentic coding tool that runs in your terminal. Follow the steps below to install the `claude` command-line interface (CLI).

### System requirements

- **Operating system:** macOS 13.0+, Windows 10 1809+ (or Windows Server 2019+), Ubuntu 20.04+, Debian 10+, or Alpine Linux 3.19+.
- **Hardware:** 4 GB+ RAM, x64 or ARM64 processor.
- **Network:** an internet connection is required.
- **Shell:** Bash, Zsh, PowerShell, or CMD.

### Install

Choose one of the following methods.

**Native installer (recommended)**

macOS, Linux, WSL:

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Windows CMD:

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

> Native installations automatically update in the background to keep you on the latest version.

**Homebrew (macOS/Linux)**

```bash
brew install --cask claude-code
```

**WinGet (Windows)**

```powershell
winget install Anthropic.ClaudeCode
```

**npm (requires [Node.js 18+](https://nodejs.org/en/download))**

```bash
npm install -g @anthropic-ai/claude-code
```

> Do **not** use `sudo npm install -g`, as this can lead to permission issues and security risks.

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

For full details, see the [official setup documentation](https://docs.anthropic.com/en/docs/claude-code/setup).
