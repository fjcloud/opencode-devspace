# OpenCode DevSpace

Dev environment for OpenShift Dev Spaces with [OpenCode](https://github.com/opencode-ai/opencode) pre-installed.

## Quick Start

Open this workspace in Dev Spaces:

```
https://devspaces.apps.rosa.fja-hcp.cqul.p3.openshiftapps.com/#https://github.com/fjcloud/opencode-devspace
```

OpenCode will be automatically installed during workspace startup and launched in a dedicated terminal.

## Manual Usage

If you need to start OpenCode manually:

```bash
opencode
```

## Configuration

OpenCode requires an API key for an AI provider. Set one of these environment variables in your Dev Spaces user preferences or directly in the terminal:

```bash
export ANTHROPIC_API_KEY="your-key"
# or
export OPENAI_API_KEY="your-key"
# or
export GEMINI_API_KEY="your-key"
```

## What's Included

- **Universal Developer Image** (UDI) with common development tools
- **OpenCode** AI coding agent, auto-installed on workspace start
- **VS Code task** that auto-launches OpenCode when the workspace opens
