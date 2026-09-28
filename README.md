# total-agent-memory plugin

Persistent memory for AI coding agents. The plugin adds two things to Claude Code and Claude Cowork:

- the **total-agent-memory MCP server**, which stores decisions, solutions, facts, lessons, errors and session summaries on your machine and returns them by search;
- the **memory-protocol skill**, which tells the agent when to recall and when to save.

The server's source code, documentation and benchmarks live in the main repository: [vbcherepanov/total-agent-memory](https://github.com/vbcherepanov/total-agent-memory). This repository only holds the plugin.

## Install

In Claude Code:

```
/plugin marketplace add vbcherepanov/total-agent-memory-plugin
/plugin install total-agent-memory@vbcherepanov
```

In Claude Cowork, add the same marketplace under **Customize → Plugins**.

Requirement: [uv](https://docs.astral.sh/uv/getting-started/installation/) must be installed, because the server starts with `uvx`.

## What runs

The plugin starts one local process: `uvx total-agent-memory==14.6.0`. The version is pinned; the plugin never installs a newer release on its own. On first start, `uvx` downloads the `total-agent-memory` package from PyPI into its cache, which takes about ten seconds on a fast connection.

The server speaks MCP over stdio. It has no hooks in this plugin and runs no background jobs outside the MCP process.

## Where your data is

Everything is stored locally in `~/.tam/` (SQLite database with full-text search, local embeddings, raw call logs). Set `TAM_MEMORY_DIR` to use another folder. Nothing is sent to a server run by the author.

Credentials that appear in saved text or tool arguments (API keys, tokens, passwords in URLs) are redacted before they are written. Redaction is best effort, so do not save secrets on purpose.

## When the network is used

- Installing the package from PyPI on first start.
- Downloading the local embedding models from Hugging Face on first use (up to about 500 MB, cached by fastembed).
- Calling an external language model only if you configure one yourself (for example `MEMORY_LLM_PROVIDER` and a key in the dashboard's Settings page). By default the server makes no LLM calls.

## Where it works

| Surface | What works |
|---|---|
| Claude Code (terminal, desktop, IDE) | MCP server and skill |
| Claude Cowork | MCP server and skill |
| claude.ai chat | Only the skill. Local MCP servers do not run in claude.ai chat, so memory is not saved there. |

## Delete your data

Remove the `~/.tam/` folder, or delete single records with the `memory_delete` tool (`hard=true` erases a record and its earlier versions).

## License

MIT. See [LICENSE](LICENSE).

---

## Codex / ChatGPT

The same repository is also a Codex plugin in the portable [Agent Plugins](https://agent-plugins.org) format. Codex reads different files than Claude Code, so the two sets do not interfere:

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code, Claude Cowork |
| `plugin.json`, `mcp.json`, `.agents/plugins/marketplace.json`, `assets/` | Codex |
| `skills/memory-protocol/SKILL.md` | both |

### Install in Codex CLI

```
codex plugin marketplace add vbcherepanov/total-agent-memory-plugin
codex plugin add total-agent-memory@vbcherepanov
```

Requirement: [uv](https://docs.astral.sh/uv/getting-started/installation/), as for Claude Code. The plugin's `mcp.json` declares one local stdio server, `uvx total-agent-memory==14.6.0`, with the version pinned. Codex starts it when a session begins and shows its tools under the server name `memory`.

Codex passes only its own allowlist of environment variables to plugin MCP servers, so a `TAM_MEMORY_DIR` exported in your shell does not reach the server. Data goes to `~/.tam/`, the same folder the Claude Code plugin uses, so both agents share one memory. If you need another folder or other settings, install the server with the installer from the main repository (`install.sh --ide codex`) instead of the plugin, which writes a regular `[mcp_servers]` entry to `~/.codex/config.toml`. Use one of the two, not both.

To turn the server off without removing the skill:

```toml
[plugins."total-agent-memory@vbcherepanov".mcp_servers.memory]
enabled = false
```

### ChatGPT

The ChatGPT desktop app reads the same local marketplaces as Codex; this was not tested with this plugin. In ChatGPT on the web and on mobile local MCP servers do not run, and this plugin is not listed in the public ChatGPT and Codex plugin directory. That directory accepts MCP servers only on a public HTTPS address. total-agent-memory keeps its data on your machine, so there is no public endpoint to list.
