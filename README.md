# NovaDB Claude Cowork Plugins

Plugins for [Claude Cowork](https://claude.ai/cowork) and [Claude Code](https://github.com/anthropics/claude-code) that extend Claude with skills and agents for working with [NovaDB](https://www.novadb.com).

## Available Plugins

| Plugin         | Description                                                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| **consulting** | Skills and agents for NovaDB consultants — schema modeling, branch management, job orchestration, and more. |

## Installation

Add this marketplace to Claude:

```bash
claude plugin marketplace add novadb/claude-plugins
```

Then install a plugin:

```bash
claude plugin install consulting@novadb-plugins
```

## Prerequisites

All plugins require a running [NovaDB MCP Server](https://github.com/novadb/novadb-mcp) connection. Configure it via `.mcp.json` in your project or user settings.

## How Plugins Work

Each plugin is a self-contained directory with:

- **`.claude-plugin/plugin.json`** — Plugin manifest (name, version, description)
- **`skills/`** — Auto-triggered domain knowledge (markdown-based)
- **`agents/`** — Sub-agents for complex multi-step tasks

No code, no infrastructure, no build steps — just markdown and JSON.

## Contributing

To add a new plugin, create a directory at the repo root with the structure above and register it in `.claude-plugin/marketplace.json`.

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
