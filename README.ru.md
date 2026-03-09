**Русский** | [English](README.md) | [Español](README.es.md) | [中文](README.zh.md)

# Rust Oxide/uMod Plugin Development — MCP Server

Полностью автоматизированная разработка плагинов для игры Rust через LLM на базе фреймворка uMod (Oxide).

LLM разрабатывает, деплоит и отлаживает C# плагины на работающем сервере Rust — без ручного копирования файлов, без перезагрузки сервера, без чтения логов. Совместим с **Cursor, Claude Desktop, VS Code (Cline, Continue.dev), Codex CLI, Gemini CLI, Grok** и любым MCP-совместимым клиентом.

**Автор:** Aleksei Gavrish — Telegram: [@alexey_gavrish](https://t.me/alexey_gavrish)

---

## Возможности

- **26 MCP инструментов** — полный цикл разработки плагина: деплой, перезагрузка, RCON, конфиг, права, мониторинг, документация, тесты
- **Деплой одним вызовом** (`rust_plugin_push`) — копирует `.cs` на сервер, ждёт компиляции Oxide, парсит ошибки, автоматически исправляет и повторяет (до 5 итераций)
- **Горячая перезагрузка** — `o.reload` через WebSocket RCON, без перезапуска сервера
- **Живая документация** — 700+ хуков с сигнатурами, 25+ гайдов с [docs.oxidemod.com](https://docs.oxidemod.com) (таймеры, CUI, БД, права, корутины, пулинг, хранение данных, атрибуты), ссылки на каталог плагинов и сообщество [umod.org](https://umod.org)
- **Режимы деплоя** — локальное копирование, FTP, SFTP для удалённых серверов
- **Диагностика** — профилирование времени выполнения хуков, парсинг NullReferenceException / InvalidCast с подсказками по исправлению, мониторинг FPS сервера
- **Авто-деплой при сохранении** — file watcher отслеживает изменения `.cs` и пушит автоматически
- **Анализ плагинов** — статический анализ кода, генерация тестов, генерация Markdown-документации
- **Универсальность** — stdio (Cursor, Cline, Continue.dev, Codex, Gemini) + HTTP (Claude Desktop, Grok, любой браузерный клиент)

---

## Требования

- **Node.js** 18+
- **Rust сервер** с включённым WebSocket RCON (`+rcon.web 1 +rcon.port 28016 +rcon.password "pass"`)
- **Oxide / uMod** установленный на сервере

---

## Настройка

### 1. Клонирование

```bash
git clone https://github.com/montukxd/mcp_rust_game_dev.git
```

Сборка не требуется — проект поставляется в виде готового бандла.

### 2. Настройка `.cursor/mcp.json`

Файл конфигурации уже есть в проекте. Откройте его и заполните своими значениями:

```json
{
  "mcpServers": {
    "rust-oxide-dev": {
      "command": "node",
      "args": ["./mcp-server/dist/index.mjs"],
      "env": {
        "RUST_RCON_HOST": "127.0.0.1",
        "RUST_RCON_PORT": "28016",
        "RUST_RCON_PASSWORD": "ваш_rcon_пароль",
        "RUST_SERVER_PATH": "C:/RustServer/Server",
        "RUST_DEPLOY_MODE": "local"
      }
    }
  }
}
```

**`RUST_SERVER_PATH`** — папка, которая является родительской для `oxide/plugins/`. Найдите, где сервер хранит плагины, и укажите путь на уровень выше. Пример: если плагины в `D:/MyServer/oxide/plugins/`, укажите `D:/MyServer`.

**`RUST_DEPLOY_MODE`** — как плагины доставляются на сервер:

| Режим | Когда использовать | Обязательные переменные |
|---|---|---|
| `local` | Сервер на том же компьютере | `RUST_SERVER_PATH` |
| `ftp` | Удалённый сервер через FTP | `RUST_FTP_HOST`, `RUST_FTP_USER`, `RUST_FTP_PASSWORD` |
| `sftp` | Удалённый сервер через SSH | `RUST_SFTP_HOST`, `RUST_SFTP_USER`, `RUST_SFTP_PASSWORD` или `RUST_SFTP_KEY` |

> Для FTP/SFTP нужно установить зависимости: `cd mcp-server && npm install`

<details>
<summary><b>Полный список FTP/SFTP переменных</b></summary>

| Переменная | По умолчанию | Описание |
|---|---|---|
| `RUST_FTP_HOST` | — | Адрес FTP сервера |
| `RUST_FTP_PORT` | `21` | Порт FTP |
| `RUST_FTP_USER` | — | Логин FTP |
| `RUST_FTP_PASSWORD` | — | Пароль FTP |
| `RUST_FTP_PATH` | `/oxide/plugins` | Удалённый путь к папке плагинов |
| `RUST_SFTP_HOST` | — | Адрес SSH сервера |
| `RUST_SFTP_PORT` | `22` | Порт SSH |
| `RUST_SFTP_USER` | — | Логин SSH |
| `RUST_SFTP_PASSWORD` | — | Пароль SSH (или используйте ключ) |
| `RUST_SFTP_KEY` | — | Путь к SSH-ключу |
| `RUST_SFTP_PATH` | `/oxide/plugins` | Удалённый путь к папке плагинов |
</details>

### 3. Откройте в Cursor — готово

Откройте папку `mcp_rust_game_dev` в Cursor. MCP сервер, Skill и Rules уже настроены и активируются автоматически.

Если вы клонировали `mcp_rust_game_dev` **внутрь другого проекта**, создайте `.cursor/mcp.json` в корне вашего проекта с тем же содержимым, но измените путь в `args` на `"./mcp_rust_game_dev/mcp-server/dist/index.mjs"`.

---

## Другие клиенты

<details>
<summary><b>Claude Desktop</b></summary>

Скопируйте `configs/claude-desktop.json` в:
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

Укажите RCON пароль и переменные деплоя, перезапустите Claude Desktop.
</details>

<details>
<summary><b>VS Code + Cline</b></summary>

Настройки Cline → MCP Servers → добавьте конфиг из `configs/vscode-cline.json`. Укажите данные подключения.
</details>

<details>
<summary><b>Continue.dev (VS Code / JetBrains)</b></summary>

Добавьте `experimental.modelContextProtocolServers` из `configs/continue-dev.json` в `~/.continue/config.json`. Укажите данные подключения.
</details>

<details>
<summary><b>OpenAI Codex CLI</b></summary>

Добавьте в `~/.codex/config.json` — тот же JSON-формат что и `.cursor/mcp.json`, используйте абсолютный путь в `args`.
</details>

<details>
<summary><b>Google Gemini CLI</b></summary>

Добавьте в `.gemini/settings.json` — тот же JSON-формат, используйте абсолютный путь в `args`.
</details>

<details>
<summary><b>Grok / HTTP клиенты</b></summary>

```bash
node mcp_rust_game_dev/mcp-server/dist/index.mjs --http
# → http://localhost:3100/mcp
```

Укажите `http://localhost:3100/mcp` в настройках MCP клиента.
</details>

---

## MCP инструменты

### Жизненный цикл плагинов
| Инструмент | Описание |
|---|---|
| `rust_plugin_push` | Деплой → компиляция → проверка (основной) |
| `rust_plugin_load` / `unload` / `reload` | Ручное управление плагинами |
| `rust_list_plugins` | Список загруженных плагинов |

### Сервер и RCON
| Инструмент | Описание |
|---|---|
| `rust_server_command` | Выполнить любую RCON команду |
| `rust_server_status` | Информация о сервере |
| `rust_server_fps` | FPS и метрики здоровья |
| `rust_read_logs` | Чтение логов Oxide |

### Конфигурация, данные и права
| Инструмент | Описание |
|---|---|
| `rust_read_config` / `rust_write_config` | Конфиг плагина (авто-перезагрузка) |
| `rust_read_data` | Файлы данных плагина |
| `rust_grant_permission` / `rust_revoke_permission` | Управление правами |
| `rust_show_permissions` | Список прав |

### Поиск по документации
| Инструмент | Описание |
|---|---|
| `rust_docs_search_hook` | Поиск хуков по ключевому слову (700+ локальный индекс) |
| `rust_docs_get_hook` | Сигнатура, пример, исходник хука с docs.oxidemod.com |
| `rust_docs_search_api` | Поиск по 25+ гайдам и API-справочнику |
| `rust_docs_get_examples` | Примеры кода для паттернов (CUI, таймеры, БД и т.д.) |
| `rust_docs_browse` | Все ресурсы — docs.oxidemod.com и ссылки на umod.org |

### Мониторинг и анализ
| Инструмент | Описание |
|---|---|
| `rust_plugin_performance` | Время выполнения хуков |
| `rust_check_runtime_errors` | Парсинг runtime ошибок с подсказками |
| `rust_analyze_plugin` | Статический анализ плагина |

### Автоматизация
| Инструмент | Описание |
|---|---|
| `rust_watch_directory` / `rust_unwatch_directory` | Авто-деплой при сохранении |
| `rust_generate_tests` | Генерация тестового плагина |
| `rust_generate_docs` | Генерация документации плагина |

---

## Процесс разработки

```
1. Исследование  →  rust_docs_search_hook / rust_docs_get_hook
2. Код           →  LLM пишет .cs плагин
3. Деплой        →  rust_plugin_push  (копирование → компиляция → проверка)
4. Ошибки        →  авто-исправление + повторный push (до 5 итераций)
5. Тестирование  →  rust_check_runtime_errors (когда пользователь тестирует)
6. Производ.     →  rust_plugin_performance (для тяжёлых хуков)
7. Финализация   →  rust_generate_docs / rust_generate_tests
```

Все шаги выполняются LLM автоматически. Разработчик только описывает, что должен делать плагин.

---

## Решение проблем

| Проблема | Решение |
|---|---|
| MCP не запускается | Проверьте `node -v` (нужен 18+), наличие `mcp-server/dist/index.mjs` |
| Нет подключения к RCON | Убедитесь в `+rcon.web 1`, проверьте порт и пароль |
| Плагин не компилируется | `rust_plugin_push` показывает ошибки автоматически |
| Файл плагина не появляется | `RUST_SERVER_PATH` должен быть родительской папкой для `oxide/plugins/`, откуда сервер загружает плагины |
| FTP/SFTP не работает | Выполните `cd mcp-server && npm install`, проверьте данные |

---

## Структура проекта

```
mcp_rust_game_dev/
├── mcp-server/
│   ├── dist/index.mjs        # Готовый MCP сервер
│   └── package.json          # Опциональные зависимости (FTP/SFTP)
├── configs/                   # Готовые конфиги для LLM клиентов
├── .cursor/
│   ├── mcp.json               # Конфигурация MCP для Cursor (редактируйте этот файл)
│   ├── rules/                 # Авто-правила для .cs файлов
│   └── skills/
│       └── rust-oxide-plugin-dev/
│           └── SKILL.md       # LLM skill: паттерны, хуки, правила
├── skills/                    # Тот же skill (для клиентов вне Cursor)
├── README.md
└── LICENSE
```

---

## Лицензия

Все плагины, созданные с помощью этого инструмента, **обязаны** содержать атрибуцию автора. См. [LICENSE](LICENSE).

**Автор:** Aleksei Gavrish — [@alexey_gavrish](https://t.me/alexey_gavrish)
