[Русский](README.ru.md) | [English](README.md) | [Español](README.es.md) | **中文**

# Rust Oxide/uMod 插件开发 — MCP 服务器

通过 LLM 使用 uMod (Oxide) 框架实现 Rust 游戏插件的全自动开发。

LLM 在运行中的 Rust 服务器上开发、部署和调试 C# 插件 — 无需手动复制文件、无需重启服务器、无需阅读日志。兼容 **Cursor、Claude Desktop、VS Code（Cline、Continue.dev）、Codex CLI、Gemini CLI、Grok** 及任何 MCP 兼容客户端。

**作者：** Aleksei Gavrish — Telegram: [@alexey_gavrish](https://t.me/alexey_gavrish)

---

## 功能特性

- **26 MCP 工具** — 覆盖插件完整生命周期：部署、重载、RCON、配置、权限、监控、文档、测试
- **一键部署** (`rust_plugin_push`) — 复制 `.cs` 到服务器，等待 Oxide 编译，解析错误，自动修复并重新推送（最多 5 次迭代）
- **热重载** — 通过 WebSocket RCON 执行 `o.reload`，无需重启服务器
- **实时文档** — 700+ hooks 及签名，25+ 开发指南来自 [docs.oxidemod.com](https://docs.oxidemod.com)（定时器、CUI、数据库、权限、协程、对象池、数据存储、属性），[umod.org](https://umod.org) 插件目录和社区链接
- **部署模式** — 本地文件复制、FTP、SFTP（远程服务器）
- **运行时诊断** — hook 执行时间分析、NullReferenceException / InvalidCast 解析及修复建议、服务器 FPS 监控
- **保存时自动部署** — 文件监视器检测 `.cs` 变更并自动推送
- **插件分析** — 静态代码分析、测试生成、Markdown 文档生成
- **通用性** — stdio（Cursor、Cline、Continue.dev、Codex、Gemini）+ HTTP（Claude Desktop、Grok、任意浏览器客户端）

---

## 系统要求

- **Node.js** 18+
- **Rust 服务器** 已启用 WebSocket RCON（`+rcon.web 1 +rcon.port 28016 +rcon.password "pass"`）
- 服务器已安装 **Oxide / uMod**

---

## 配置

### 1. 克隆

```bash
git clone https://github.com/montukxd/mcp_rust_game_dev.git
```

无需构建 — 已作为就绪的 bundle 分发。

### 2. 配置 `.cursor/mcp.json`

配置文件已包含在项目中。打开并填入您的值：

```json
{
  "mcpServers": {
    "rust-oxide-dev": {
      "command": "node",
      "args": ["./mcp-server/dist/index.mjs"],
      "env": {
        "RUST_RCON_HOST": "127.0.0.1",
        "RUST_RCON_PORT": "28016",
        "RUST_RCON_PASSWORD": "您的rcon密码",
        "RUST_SERVER_PATH": "C:/RustServer/Server",
        "RUST_DEPLOY_MODE": "local"
      }
    }
  }
}
```

**`RUST_SERVER_PATH`** — `oxide/plugins/` 的上级目录。找到服务器存放插件的位置，然后向上一级。示例：如果插件在 `D:/MyServer/oxide/plugins/`，则设置 `D:/MyServer`。

**`RUST_DEPLOY_MODE`** — 插件如何传送到服务器：

| 模式 | 使用场景 | 必需变量 |
|---|---|---|
| `local` | 服务器在本机 | `RUST_SERVER_PATH` |
| `ftp` | 通过 FTP 连接远程服务器 | `RUST_FTP_HOST`、`RUST_FTP_USER`、`RUST_FTP_PASSWORD` |
| `sftp` | 通过 SSH 连接远程服务器 | `RUST_SFTP_HOST`、`RUST_SFTP_USER`、`RUST_SFTP_PASSWORD` 或 `RUST_SFTP_KEY` |

> FTP/SFTP 模式需要安装可选依赖：`cd mcp-server && npm install`

<details>
<summary><b>FTP/SFTP 完整变量列表</b></summary>

| 变量 | 默认值 | 说明 |
|---|---|---|
| `RUST_FTP_HOST` | — | FTP 服务器地址 |
| `RUST_FTP_PORT` | `21` | FTP 端口 |
| `RUST_FTP_USER` | — | FTP 用户名 |
| `RUST_FTP_PASSWORD` | — | FTP 密码 |
| `RUST_FTP_PATH` | `/oxide/plugins` | 远程插件目录路径 |
| `RUST_SFTP_HOST` | — | SSH 服务器地址 |
| `RUST_SFTP_PORT` | `22` | SSH 端口 |
| `RUST_SFTP_USER` | — | SSH 用户名 |
| `RUST_SFTP_PASSWORD` | — | SSH 密码（或使用密钥） |
| `RUST_SFTP_KEY` | — | SSH 私钥路径 |
| `RUST_SFTP_PATH` | `/oxide/plugins` | 远程插件目录路径 |
</details>

### 3. 在 Cursor 中打开 — 完成

在 Cursor 中打开 `mcp_rust_game_dev` 文件夹。MCP 服务器、Skill 和 Rules 已预配置，将自动激活。

如果您将 `mcp_rust_game_dev` 克隆到**另一个项目内部**，请在项目根目录创建 `.cursor/mcp.json`，内容相同，但将 `args` 路径改为 `"./mcp_rust_game_dev/mcp-server/dist/index.mjs"`。

---

## 其他客户端

<details>
<summary><b>Claude Desktop</b></summary>

将 `configs/claude-desktop.json` 复制到：
- **Windows：** `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS：** `~/Library/Application Support/Claude/claude_desktop_config.json`

设置 RCON 密码和部署变量，重启 Claude Desktop。
</details>

<details>
<summary><b>VS Code + Cline</b></summary>

Cline 设置 → MCP Servers → 从 `configs/vscode-cline.json` 添加配置。设置凭据。
</details>

<details>
<summary><b>Continue.dev（VS Code / JetBrains）</b></summary>

将 `configs/continue-dev.json` 中的 `experimental.modelContextProtocolServers` 添加到 `~/.continue/config.json`。设置凭据。
</details>

<details>
<summary><b>OpenAI Codex CLI</b></summary>

添加到 `~/.codex/config.json` — 与 `.cursor/mcp.json` 相同的 JSON 格式，使用绝对路径。
</details>

<details>
<summary><b>Google Gemini CLI</b></summary>

相同的 JSON 格式 — 添加到 `.gemini/settings.json`，使用绝对路径。
</details>

<details>
<summary><b>Grok / HTTP 客户端</b></summary>

```bash
node mcp_rust_game_dev/mcp-server/dist/index.mjs --http
# → http://localhost:3100/mcp
```

将任意 MCP 客户端指向 `http://localhost:3100/mcp`。
</details>

---

## MCP 工具

### 插件生命周期
| 工具 | 说明 |
|---|---|
| `rust_plugin_push` | 部署 → 编译 → 验证（主要工具） |
| `rust_plugin_load` / `unload` / `reload` | 手动插件控制 |
| `rust_list_plugins` | 已加载插件列表 |

### 服务器与 RCON
| 工具 | 说明 |
|---|---|
| `rust_server_command` | 执行任意 RCON 命令 |
| `rust_server_status` | 服务器信息（地图、玩家、版本） |
| `rust_server_fps` | FPS 与健康指标 |
| `rust_read_logs` | 读取 Oxide 日志 |

### 配置、数据与权限
| 工具 | 说明 |
|---|---|
| `rust_read_config` / `rust_write_config` | 插件配置（自动重载） |
| `rust_read_data` | 插件数据文件 |
| `rust_grant_permission` / `rust_revoke_permission` | 权限管理 |
| `rust_show_permissions` | 列出权限 |

### 文档查找
| 工具 | 说明 |
|---|---|
| `rust_docs_search_hook` | 按关键词查找 hooks（700+ 本地索引） |
| `rust_docs_get_hook` | 从 docs.oxidemod.com 获取签名、示例和源代码 |
| `rust_docs_search_api` | 搜索 25+ 开发指南和 API 参考 |
| `rust_docs_get_examples` | 模式代码示例（CUI、定时器、数据库等） |
| `rust_docs_browse` | 所有资源 — docs.oxidemod.com 和 umod.org 链接 |

### 监控与分析
| 工具 | 说明 |
|---|---|
| `rust_plugin_performance` | Hook 执行时间 |
| `rust_check_runtime_errors` | 解析运行时异常并提供修复提示 |
| `rust_analyze_plugin` | 静态插件分析 |

### 自动化
| 工具 | 说明 |
|---|---|
| `rust_watch_directory` / `rust_unwatch_directory` | 文件保存时自动部署 |
| `rust_generate_tests` | 生成测试插件 |
| `rust_generate_docs` | 生成插件文档 |

---

## 开发工作流

```
1. 调研    →  rust_docs_search_hook / rust_docs_get_hook
2. 编码    →  LLM 编写 .cs 插件
3. 部署    →  rust_plugin_push  （复制 → 编译 → 检查）
4. 修复错误 →  自动修复 + 重新推送（最多 5 次迭代）
5. 测试    →  rust_check_runtime_errors（用户测试时）
6. 性能    →  rust_plugin_performance（针对重型 hooks）
7. 收尾    →  rust_generate_docs / rust_generate_tests
```

所有步骤由 LLM 自动完成。开发者只需描述插件应实现的功能。

---

## 故障排除

| 问题 | 解决方案 |
|---|---|
| MCP 无法启动 | 检查 `node -v`（需 18+），确认 `mcp-server/dist/index.mjs` 存在 |
| 无 RCON 连接 | 确保服务器参数中有 `+rcon.web 1`，检查端口和密码 |
| 插件无法编译 | `rust_plugin_push` 会自动显示错误 |
| 插件文件缺失 | `RUST_SERVER_PATH` 必须是服务器加载插件的 `oxide/plugins/` 的上级目录 |
| FTP/SFTP 失败 | 运行 `cd mcp-server && npm install`，检查凭据 |

---

## 项目结构

```
mcp_rust_game_dev/
├── mcp-server/
│   ├── dist/index.mjs        # 可直接运行的 MCP 服务器包
│   └── package.json          # 可选依赖（仅 FTP/SFTP）
├── configs/                   # 预置 LLM 客户端配置
├── .cursor/
│   ├── mcp.json               # Cursor MCP 配置（编辑此文件）
│   ├── rules/                 # .cs 文件的 Cursor 自动规则
│   └── skills/
│       └── rust-oxide-plugin-dev/
│           └── SKILL.md       # LLM skill：模式、hooks、规则
├── skills/                    # 同一 skill（适用于非 Cursor 客户端）
├── README.md
└── LICENSE
```

---

## 许可证

使用本工具创建的所有插件**必须**包含作者署名。参见 [LICENSE](LICENSE)。

**作者：** Aleksei Gavrish — [@alexey_gavrish](https://t.me/alexey_gavrish)
