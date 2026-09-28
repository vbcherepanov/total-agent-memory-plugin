# Privacy policy: total-agent-memory plugin

Effective 28 September 2026. Applies to the total-agent-memory plugin for Claude Code, Claude Cowork and Codex published in this repository.

## Who runs it

The plugin is open-source software maintained by Vitalii Cherepanov. It runs entirely on your computer. The author operates no server for this plugin and receives no data from it.

## What data it handles

The plugin starts the total-agent-memory MCP server locally. The server stores what you or your agent explicitly save through its tools: decisions, notes, solutions, facts, errors, session summaries, and the arguments of tool calls in a local call log. It also keeps local search indexes and embeddings of that text.

## Where the data is stored

On your machine, in `~/.tam/` (a SQLite database and index files). On macOS and Linux the folder is created readable only by your user account. You can choose another folder with the `TAM_MEMORY_DIR` setting in Claude Code.

## What leaves your machine

The plugin does not send your memories anywhere. The network is used only to:

- download the pinned `total-agent-memory` package from PyPI when the server first starts;
- download the local embedding models from Hugging Face on first use;
- call an external language model provider, only if you configure one yourself. In that case text you send to that provider is covered by the provider's own privacy policy. By default no language model is called.

## Credentials

Common credential formats (API keys, tokens, passwords in URLs) are redacted before anything is written to disk. Redaction is best effort; do not save secrets on purpose.

## Retention and deletion

Data stays until you delete it. Delete single records with the `memory_delete` tool (`hard=true` removes a record and its earlier versions), or delete the `~/.tam/` folder to remove everything.

## Children

The plugin is a developer tool and is not directed at people under 18.

## Changes and contact

Changes to this policy are published in this repository with the date above. Questions: open an issue at https://github.com/vbcherepanov/total-agent-memory-plugin/issues.
