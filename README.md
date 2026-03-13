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
claude plugin install novadb-consulting@novadb-plugins
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

### Local Development (NovaDB Consultants)

To actively develop and test skills and agents locally:

1. Clone this repository:

   ```bash
   git clone https://github.com/novadb/claude-plugins.git
   ```

2. Start Claude Code with the local plugin directory:

   ```bash
   claude --plugin-dir=/path/to/claude-plugins/consulting
   ```

   This loads the skills and agents directly from your local checkout instead of the installed marketplace version.

3. Create a `feature/*` branch for your changes and open a pull request when ready.

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
