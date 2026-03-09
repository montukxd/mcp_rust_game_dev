[Русский](README.ru.md) | [English](README.md) | **Español** | [中文](README.zh.md)

# Rust Oxide/uMod Plugin Development — MCP Server

Desarrollo totalmente automatizado de plugins para el juego Rust mediante LLM usando el framework uMod (Oxide).

El LLM desarrolla, despliega y depura plugins C# en un servidor Rust en vivo — sin copiar archivos manualmente, sin reiniciar el servidor, sin leer logs. Compatible con **Cursor, Claude Desktop, VS Code (Cline, Continue.dev), Codex CLI, Gemini CLI, Grok** y cualquier cliente compatible con MCP.

**Autor:** Aleksei Gavrish — Telegram: [@alexey_gavrish](https://t.me/alexey_gavrish)

---

## Características

- **26 herramientas MCP** — ciclo completo del plugin: despliegue, recarga, RCON, config, permisos, monitoreo, documentación, pruebas
- **Despliegue en una llamada** (`rust_plugin_push`) — copia `.cs` al servidor, espera compilación de Oxide, analiza errores, auto-corrige y reenvía (hasta 5 iteraciones)
- **Recarga en caliente** — `o.reload` vía WebSocket RCON, sin reiniciar servidor
- **Documentación en vivo** — 700+ hooks con firmas, 25+ guías de [docs.oxidemod.com](https://docs.oxidemod.com) (timers, CUI, DB, permisos, corrutinas, pooling, almacenamiento, atributos), enlaces al catálogo de plugins y comunidad de [umod.org](https://umod.org)
- **Modos de despliegue** — copia local, FTP, SFTP para servidores remotos
- **Diagnósticos** — perfilado de tiempo de hooks, análisis de NullReferenceException / InvalidCast con sugerencias, monitoreo de FPS
- **Auto-despliegue al guardar** — file watcher detecta cambios en `.cs` y despliega automáticamente
- **Análisis de plugins** — análisis estático, generación de pruebas, generación de documentación Markdown
- **Universal** — stdio (Cursor, Cline, Continue.dev, Codex, Gemini) + HTTP (Claude Desktop, Grok, cualquier cliente web)

---

## Requisitos

- **Node.js** 18+
- **Servidor Rust** con WebSocket RCON habilitado (`+rcon.web 1 +rcon.port 28016 +rcon.password "pass"`)
- **Oxide / uMod** instalado en el servidor

---

## Configuración

### 1. Clonar

```bash
git clone https://github.com/montukxd/mcp_rust_game_dev.git
```

No requiere compilación — se distribuye como bundle listo para usar.

### 2. Configurar `.cursor/mcp.json`

El archivo de configuración ya viene incluido en el proyecto. Ábrelo y completa con tus valores:

```json
{
  "mcpServers": {
    "rust-oxide-dev": {
      "command": "node",
      "args": ["./mcp-server/dist/index.mjs"],
      "env": {
        "RUST_RCON_HOST": "127.0.0.1",
        "RUST_RCON_PORT": "28016",
        "RUST_RCON_PASSWORD": "tu_contraseña_rcon",
        "RUST_SERVER_PATH": "C:/RustServer/Server",
        "RUST_DEPLOY_MODE": "local"
      }
    }
  }
}
```

**`RUST_SERVER_PATH`** — la carpeta padre de `oxide/plugins/`. Encuentra dónde el servidor almacena los plugins y sube un nivel. Ejemplo: si los plugins están en `D:/MyServer/oxide/plugins/`, establece `D:/MyServer`.

> **Requisitos de nombres de archivos del servidor** para auto-detección:
> - El **log de consola** debe llamarse `output.txt` o `output_log.txt` y estar en `RUST_SERVER_PATH`. Si tu log tiene otro nombre o ruta, configura `RUST_CONSOLE_LOG_PATH` en env. El MCP también lee el argumento `-logfile` del script de arranque si está presente.
> - El **script de arranque** debe llamarse `start.bat` o `run.bat` (también soporta `RustDedicated.bat`, `start.cmd`, `run.cmd`, `start.sh`). Si el tuyo tiene otro nombre, configura `RUST_STARTUP_FILE` en env.

**`RUST_DEPLOY_MODE`** — cómo se entregan los plugins al servidor:

| Modo | Cuándo usar | Variables requeridas |
|---|---|---|
| `local` | Servidor en la misma máquina | `RUST_SERVER_PATH` |
| `ftp` | Servidor remoto vía FTP | `RUST_FTP_HOST`, `RUST_FTP_USER`, `RUST_FTP_PASSWORD` |
| `sftp` | Servidor remoto vía SSH | `RUST_SFTP_HOST`, `RUST_SFTP_USER`, `RUST_SFTP_PASSWORD` o `RUST_SFTP_KEY` |

> El modo FTP/SFTP requiere instalar dependencias opcionales: `cd mcp-server && npm install`

<details>
<summary><b>Lista completa de variables FTP/SFTP</b></summary>

| Variable | Por defecto | Descripción |
|---|---|---|
| `RUST_FTP_HOST` | — | Dirección del servidor FTP |
| `RUST_FTP_PORT` | `21` | Puerto FTP |
| `RUST_FTP_USER` | — | Login FTP |
| `RUST_FTP_PASSWORD` | — | Contraseña FTP |
| `RUST_FTP_PATH` | `/oxide/plugins` | Ruta remota a la carpeta de plugins |
| `RUST_SFTP_HOST` | — | Dirección del servidor SSH |
| `RUST_SFTP_PORT` | `22` | Puerto SSH |
| `RUST_SFTP_USER` | — | Login SSH |
| `RUST_SFTP_PASSWORD` | — | Contraseña SSH (o usar clave) |
| `RUST_SFTP_KEY` | — | Ruta a la clave SSH privada |
| `RUST_SFTP_PATH` | `/oxide/plugins` | Ruta remota a la carpeta de plugins |
</details>

### 3. Abrir en Cursor — listo

Abre la carpeta `mcp_rust_game_dev` en Cursor. El servidor MCP, Skill y Rules están preconfigurados y se activarán automáticamente.

Si clonaste `mcp_rust_game_dev` **dentro de otro proyecto**, crea `.cursor/mcp.json` en la raíz de tu proyecto con el mismo contenido, pero cambia la ruta en `args` a `"./mcp_rust_game_dev/mcp-server/dist/index.mjs"`.

---

## Otros clientes

<details>
<summary><b>Claude Desktop</b></summary>

Copia `configs/claude-desktop.json` a:
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`

Configura la contraseña RCON y las variables de despliegue, reinicia Claude Desktop.
</details>

<details>
<summary><b>VS Code + Cline</b></summary>

Configuración de Cline → MCP Servers → añade la configuración desde `configs/vscode-cline.json`. Configura las credenciales.
</details>

<details>
<summary><b>Continue.dev (VS Code / JetBrains)</b></summary>

Añade `experimental.modelContextProtocolServers` desde `configs/continue-dev.json` a `~/.continue/config.json`. Configura las credenciales.
</details>

<details>
<summary><b>OpenAI Codex CLI</b></summary>

Añade a `~/.codex/config.json` — mismo formato JSON que `.cursor/mcp.json`, usa ruta absoluta en `args`.
</details>

<details>
<summary><b>Google Gemini CLI</b></summary>

Mismo formato JSON — añade a `.gemini/settings.json`, usa ruta absoluta en `args`.
</details>

<details>
<summary><b>Grok / clientes HTTP</b></summary>

```bash
node mcp_rust_game_dev/mcp-server/dist/index.mjs --http
# → http://localhost:3100/mcp
```

Apunta cualquier cliente MCP a `http://localhost:3100/mcp`.
</details>

---

## Herramientas MCP

### Ciclo de vida del plugin
| Herramienta | Descripción |
|---|---|
| `rust_plugin_push` | Desplegar → compilar → verificar (herramienta principal) |
| `rust_plugin_load` / `unload` / `reload` | Control manual del plugin |
| `rust_list_plugins` | Lista de plugins cargados |

### Servidor y RCON
| Herramienta | Descripción |
|---|---|
| `rust_server_command` | Ejecutar cualquier comando RCON |
| `rust_server_status` | Información del servidor (mapa, jugadores, versión) |
| `rust_server_fps` | FPS y métricas de salud |
| `rust_read_logs` | Leer logs de Oxide |
| `rust_read_console_log` | Leer log de consola del servidor (output.txt) con filtrado de errores |

### Config, datos y permisos
| Herramienta | Descripción |
|---|---|
| `rust_read_config` / `rust_write_config` | Config del plugin (recarga automática) |
| `rust_read_data` | Archivos de datos del plugin |
| `rust_grant_permission` / `rust_revoke_permission` | Gestión de permisos |
| `rust_show_permissions` | Listar permisos |

### Búsqueda en documentación
| Herramienta | Descripción |
|---|---|
| `rust_docs_search_hook` | Buscar hooks por palabra clave (700+ índice local) |
| `rust_docs_get_hook` | Firma, ejemplo y código fuente desde docs.oxidemod.com |
| `rust_docs_search_api` | Buscar en 25+ guías y referencia API |
| `rust_docs_get_examples` | Ejemplos de código para patrones (CUI, timers, DB, etc.) |
| `rust_docs_browse` | Todos los recursos — docs.oxidemod.com y enlaces a umod.org |

### Monitoreo y análisis
| Herramienta | Descripción |
|---|---|
| `rust_plugin_performance` | Tiempos de ejecución de hooks |
| `rust_check_runtime_errors` | Analizar excepciones con sugerencias de corrección |
| `rust_analyze_plugin` | Análisis estático del plugin |

### Automatización
| Herramienta | Descripción |
|---|---|
| `rust_watch_directory` / `rust_unwatch_directory` | Despliegue automático al guardar archivo |
| `rust_generate_tests` | Generar plugin de pruebas |
| `rust_generate_docs` | Generar documentación del plugin |

---

## Flujo de trabajo

```
1. Investigar  →  rust_docs_search_hook / rust_docs_get_hook
2. Codificar   →  El LLM escribe el plugin .cs
3. Desplegar   →  rust_plugin_push  (copiar → compilar → verificar)
4. Corregir    →  auto-fix + re-push (hasta 5 iteraciones)
5. Probar      →  rust_check_runtime_errors (cuando el usuario prueba)
6. Rendimiento →  rust_plugin_performance (para hooks pesados)
7. Finalizar   →  rust_generate_docs / rust_generate_tests
```

Todos los pasos los realiza el LLM automáticamente. El desarrollador solo describe qué debe hacer el plugin.

---

## Solución de problemas

| Problema | Solución |
|---|---|
| MCP no arranca | Comprueba `node -v` (se requiere 18+), verifica que existe `mcp-server/dist/index.mjs` |
| Sin conexión RCON | Asegúrate de `+rcon.web 1` en los parámetros del servidor, comprueba puerto y contraseña |
| El plugin no compila | `rust_plugin_push` muestra los errores automáticamente |
| Archivo del plugin no encontrado | `RUST_SERVER_PATH` debe ser la carpeta padre de `oxide/plugins/` de donde el servidor carga plugins |
| FTP/SFTP falla | Ejecuta `cd mcp-server && npm install`, comprueba credenciales |

---

## Estructura del proyecto

```
mcp_rust_game_dev/
├── mcp-server/
│   ├── dist/index.mjs        # Bundle del servidor MCP listo para ejecutar
│   └── package.json          # Dependencias opcionales (solo FTP/SFTP)
├── configs/                   # Configuraciones predefinidas para clientes LLM
├── .cursor/
│   ├── mcp.json               # Configuración MCP de Cursor (editar este archivo)
│   ├── rules/                 # Auto-reglas de Cursor para archivos .cs
│   └── skills/
│       └── rust-oxide-plugin-dev/
│           └── SKILL.md       # Skill para LLM: patrones, hooks, reglas
├── skills/                    # Mismo skill (para clientes fuera de Cursor)
├── README.md
└── LICENSE
```

---

## Licencia

Todos los plugins creados con esta herramienta **deben** incluir atribución de autor. Ver [LICENSE](LICENSE).

**Autor:** Aleksei Gavrish — [@alexey_gavrish](https://t.me/alexey_gavrish)
