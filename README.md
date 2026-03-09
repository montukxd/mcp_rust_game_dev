[Русский](README.ru.md) | **English** | [Español](README.es.md) | [中文](README.zh.md)

# Rust Oxide/uMod Plugin Development — MCP Server

Fully automated Rust game plugin development via LLM using the uMod (Oxide) framework.

The LLM develops, deploys, and debugs C# plugins on a live Rust server — no manual file copying, no server restarts, no log reading. Compatible with **Cursor, Claude Desktop, VS Code (Cline, Continue.dev), Codex CLI, Gemini CLI, Grok** and any MCP-compatible client.

**Author:** Aleksei Gavrish — Telegram: [@alexey_gavrish](https://t.me/alexey_gavrish)

---

## Features

- **26 MCP tools** covering the full plugin lifecycle: deploy, reload, RCON, config, permissions, monitoring, documentation, tests
- **One-call deploy** (`rust_plugin_push`) — copies `.cs` to server, waits for Oxide compilation, parses errors, auto-fixes and re-pushes (up to 5 iterations)
- **Hot reload** — `o.reload` via WebSocket RCON, no server restart
- **Live documentation** — 700+ hooks with signatures, 25+ developer guides fetched from [docs.oxidemod.com](https://docs.oxidemod.com) (timers, CUI, database, permissions, coroutines, pooling, data storage, attributes), links to [umod.org](https://umod.org) plugin catalog and community
- **Deploy modes** — local file copy, FTP, SFTP for remote servers
- **Runtime diagnostics** — hook execution time profiling, NullReferenceException / InvalidCast parsing with fix suggestions, server FPS monitoring
- **Auto-deploy on save** — file watcher detects `.cs` changes and pushes automatically
- **Plugin analysis** — static code analysis, test generation, Markdown documentation generation
- **Universal** — stdio transport (Cursor, Cline, Continue.dev, Codex, Gemini) + HTTP transport (Claude Desktop, Grok, any browser client)

---

## Requirements

- **Node.js** 18+
- **Rust server** with WebSocket RCON enabled (`+rcon.web 1 +rcon.port 28016 +rcon.password "pass"`)
- **Oxide / uMod** installed on the server

---

## Setup

### 1. Clone

```bash
git clone https://github.com/montukxd/mcp_rust_game_dev.git
```

No build needed — ships as a ready bundle.

### 2. Configure `.cursor/mcp.json`

The config file is already in the project. Open it and fill in your values:

```json
{
  "mcpServers": {
    "rust-oxide-dev": {
      "command": "node",
      "args": ["./mcp-server/dist/index.mjs"],
      "env": {
        "RUST_RCON_HOST": "127.0.0.1",
        "RUST_RCON_PORT": "28016",
        "RUST_RCON_PASSWORD": "your_rcon_password",
        "RUST_SERVER_PATH": "C:/RustServer/Server",
        "RUST_DEPLOY_MODE": "local"
      }
    }
  }
}
```

**`RUST_SERVER_PATH`** — the folder that is the parent of `oxide/plugins/`. Find where your server keeps its plugins and set the path one level above. Example: if plugins are in `D:/MyServer/oxide/plugins/`, set `D:/MyServer`.

> **Server file naming requirements** for auto-detection:
> - **Console log** must be named `output.txt` or `output_log.txt` and located in `RUST_SERVER_PATH`. If your log has a different name or path, set `RUST_CONSOLE_LOG_PATH` in env. The MCP also reads the `-logfile` argument from the startup script if present.
> - **Startup script** must be named `start.bat` or `run.bat` (also supports `RustDedicated.bat`, `start.cmd`, `run.cmd`, `start.sh`). If yours has a different name, set `RUST_STARTUP_FILE` in env.

**`RUST_DEPLOY_MODE`** — how plugins are delivered to the server:

| Mode | When to use | Required variables |
|---|---|---|
| `local` | Server is on the same machine | `RUST_SERVER_PATH` |
| `ftp` | Remote server via FTP | `RUST_FTP_HOST`, `RUST_FTP_USER`, `RUST_FTP_PASSWORD` |
| `sftp` | Remote server via SSH | `RUST_SFTP_HOST`, `RUST_SFTP_USER`, `RUST_SFTP_PASSWORD` or `RUST_SFTP_KEY` |

> FTP/SFTP mode requires installing optional dependencies: `cd mcp-server && npm install`

<details>
<summary><b>Full list of FTP/SFTP variables</b></summary>

| Variable | Default | Description |
|---|---|---|
| `RUST_FTP_HOST` | — | FTP server address |
| `RUST_FTP_PORT` | `21` | FTP port |
| `RUST_FTP_USER` | — | FTP login |
| `RUST_FTP_PASSWORD` | — | FTP password |
| `RUST_FTP_PATH` | `/oxide/plugins` | Remote path to plugins folder |
| `RUST_SFTP_HOST` | — | SSH server address |
| `RUST_SFTP_PORT` | `22` | SSH port |
| `RUST_SFTP_USER` | — | SSH login |
| `RUST_SFTP_PASSWORD` | — | SSH password (or use key) |
| `RUST_SFTP_KEY` | — | Path to SSH private key |
| `RUST_SFTP_PATH` | `/oxide/plugins` | Remote path to plugins folder |
</details>

### 3. Open in Cursor — done

Open the `mcp_rust_game_dev` folder in Cursor. MCP server, Skill, and Rules are pre-configured and will activate automatically.

If you cloned `mcp_rust_game_dev` **inside another project**, create `.cursor/mcp.json` in your project root with the same content, but change `args` path to `"./mcp_rust_game_dev/mcp-server/dist/index.mjs"`.

---

## Other Clients

<details>
<summary><b>Claude Desktop</b></summary>

Copy `configs/claude-desktop.json` to:
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

Set RCON password and deploy variables, restart Claude Desktop.
</details>

<details>
<summary><b>VS Code + Cline</b></summary>

Cline settings → MCP Servers → add config from `configs/vscode-cline.json`. Set credentials.
</details>

<details>
<summary><b>Continue.dev (VS Code / JetBrains)</b></summary>

Add `experimental.modelContextProtocolServers` from `configs/continue-dev.json` to `~/.continue/config.json`. Set credentials.
</details>

<details>
<summary><b>OpenAI Codex CLI</b></summary>

Add to `~/.codex/config.json` — same JSON format as `.cursor/mcp.json`, use absolute path in `args`.
</details>

<details>
<summary><b>Google Gemini CLI</b></summary>

Add to `.gemini/settings.json` — same JSON format, use absolute path in `args`.
</details>

<details>
<summary><b>Grok / HTTP clients</b></summary>

```bash
node mcp_rust_game_dev/mcp-server/dist/index.mjs --http
# → http://localhost:3100/mcp
```

Point any MCP client to `http://localhost:3100/mcp`.
</details>

---

## MCP Tools

### Plugin Lifecycle
| Tool | Description |
|---|---|
| `rust_plugin_push` | Deploy → compile → verify (main tool) |
| `rust_plugin_load` / `unload` / `reload` | Manual plugin control |
| `rust_list_plugins` | Loaded plugins list |

### Server & RCON
| Tool | Description |
|---|---|
| `rust_server_command` | Execute any RCON command |
| `rust_server_status` | Server info (map, players, version) |
| `rust_server_fps` | FPS and health metrics |
| `rust_read_logs` | Read Oxide logs |
| `rust_read_console_log` | Read server console log (output.txt) with error filtering |

### Config, Data & Permissions
| Tool | Description |
|---|---|
| `rust_read_config` / `rust_write_config` | Plugin config (auto-reloads) |
| `rust_read_data` | Plugin data files |
| `rust_grant_permission` / `rust_revoke_permission` | Permission management |
| `rust_show_permissions` | List permissions |

### Documentation Lookup
| Tool | Description |
|---|---|
| `rust_docs_search_hook` | Find hooks by keyword (700+ local index) |
| `rust_docs_get_hook` | Hook signature, example, source from docs.oxidemod.com |
| `rust_docs_search_api` | Search 25+ developer guides and API reference |
| `rust_docs_get_examples` | Code examples for patterns (CUI, timers, database, etc.) |
| `rust_docs_browse` | List all docs — both docs.oxidemod.com and umod.org links |

### Monitoring & Analysis
| Tool | Description |
|---|---|
| `rust_plugin_performance` | Hook execution times |
| `rust_check_runtime_errors` | Parse runtime exceptions with fix hints |
| `rust_analyze_plugin` | Static plugin analysis |

### Automation
| Tool | Description |
|---|---|
| `rust_watch_directory` / `rust_unwatch_directory` | Auto-deploy on file save |
| `rust_generate_tests` | Generate test plugin |
| `rust_generate_docs` | Generate plugin documentation |

---

## Development Workflow

```
1. Research    →  rust_docs_search_hook / rust_docs_get_hook
2. Code        →  LLM writes the .cs plugin
3. Deploy      →  rust_plugin_push  (copy → compile → check)
4. Fix errors  →  auto-fix + re-push (up to 5 iterations)
5. Test        →  rust_check_runtime_errors (when user tests)
6. Performance →  rust_plugin_performance (for heavy hooks)
7. Finalize    →  rust_generate_docs / rust_generate_tests
```

All steps are handled by the LLM automatically. The developer only describes what the plugin should do.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| MCP won't start | Check `node -v` (18+ required), verify `mcp-server/dist/index.mjs` exists |
| No RCON connection | Ensure `+rcon.web 1` in server params, check port and password |
| Plugin won't compile | `rust_plugin_push` shows errors automatically |
| Plugin file missing | `RUST_SERVER_PATH` must be the parent of `oxide/plugins/` where your server loads plugins |
| FTP/SFTP fails | Run `cd mcp-server && npm install`, check credentials |

---

## Project Structure

```
mcp_rust_game_dev/
├── mcp-server/
│   ├── dist/index.mjs        # Ready-to-run MCP server bundle
│   └── package.json          # Optional deps (FTP/SFTP only)
├── configs/                   # Pre-made configs for LLM clients
├── .cursor/
│   ├── mcp.json               # Cursor MCP config (edit this)
│   ├── rules/                 # Cursor auto-rules for .cs files
│   └── skills/
│       └── rust-oxide-plugin-dev/
│           └── SKILL.md       # LLM skill: patterns, hooks, rules
├── skills/                    # Same skill (for non-Cursor clients)
├── README.md
└── LICENSE
```

---

## License

All plugins created with this tool **must** include author attribution. See [LICENSE](LICENSE).

**Author:** Aleksei Gavrish — [@alexey_gavrish](https://t.me/alexey_gavrish)
