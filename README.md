# NoxAeApi

> A REST + WebSocket API plugin for Bukkit / Spigot / PaperMC Minecraft servers.

NoxAeApi exposes your Minecraft server's data and actions through a clean HTTP REST API and real-time WebSocket feeds, allowing external applications, dashboards, panels, or automation scripts to interact with your server without needing direct console or RCON access. It also supports aggregating multiple backend servers behind a single network-wide API.

🇮🇩 *Looking for the Indonesian version? See [README.id.md](README.id.md).*

---

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Licensing](#licensing)
- [Configuration](#configuration)
- [Authentication](#authentication)
- [API Endpoints](#api-endpoints)
- [WebSocket](#websocket)
- [Webhooks (Outbound)](#webhooks-outbound)
- [Discord Notifications](#discord-notifications)
- [Multi-Server Network Aggregator](#multi-server-network-aggregator)
- [Swagger UI](#swagger-ui)
- [In-Game Commands](#in-game-commands)
- [Permissions](#permissions)
- [Building From Source](#building-from-source)
- [Built With](#built-with)
- [Version](#version)

---

## Features

- **Player Management** — list online/offline players, fetch by UUID, kick, ban/unban, teleport, change gamemode, read stats, and view inventory.
- **Server Control** — execute console commands, restart the server, read recent console logs, list ops/whitelist, ban an IP, inspect loaded entities/chunks.
- **Plugin Management** — list installed plugins and enable/disable them remotely.
- **World Management** — list worlds, get world info, save world(s), download world data, change time/weather, and inspect entities in a world.
- **Economy (Vault)** — read economy info, pay/debit players, check balances, and view a top-balance leaderboard.
- **LuckPerms** — manage groups and permissions: list groups/permissions, grant/revoke permissions, set a player's primary group, check a specific permission.
- **PlaceholderAPI** — resolve placeholders for a player via the API.
- **Advancements** — list player advancements.
- **Scoreboards** — read scoreboards/objectives and set/remove individual scores.
- **Chat** — broadcast server-wide messages or send a private message to a specific player.
- **NoxAuth Integration** *(optional)* — read a player's auth status (registered/authenticated/last login/country) and run password checks.
- **WebSocket Streaming** — real-time console log streaming (`ws/console`) and real-time game-event streaming (`ws/events`: join/quit/chat/death/kick).
- **Outbound Webhooks** — push game events (join, quit, chat, death, kick) to your own HTTP endpoints.
- **Discord Notifications** — get alerted in a Discord channel for sensitive actions (unauthorized access, command execution, op/deop, whitelist changes, blocked IPs).
- **Multi-Server Network Aggregator** — turn one server into a hub that aggregates player lists, health, and status across multiple backend servers, and forwards/broadcasts requests to them.
- **Security Suite** — single or multi-key authentication (with per-key read-only mode and endpoint allow-lists), IP whitelisting, per-key/IP rate limiting, audit logging, blocked-paths configuration.
- **Swagger / OpenAPI UI** — built-in, browsable, interactive API documentation.
- **TLS Support** — optional HTTPS via a JKS keystore, with optional SNI enforcement.
- **CORS Configuration** — configurable list of allowed origins.
- **Licensed Plugin** — protected by NxGate online license validation with offline grace period.

---

## Requirements

- Java 21+
- PaperMC / Spigot / Bukkit, API version 1.21+
- Maven (only if building from source)
- A valid **NxGate license key** (see [Licensing](#licensing))

**Optional soft-dependencies** (endpoints degrade gracefully / return `503` if missing):
- [Vault](https://www.spigotmc.org/resources/vault.34315/) — required for `/v1/economy/*` endpoints
- [LuckPerms](https://luckperms.net/) — required for `/v1/luckperms/*` endpoints
- [PlaceholderAPI](https://www.spigotmc.org/resources/placeholderapi.6245/) — required for `/v1/placeholders/replace`
- **NoxAuth** — required for `/v1/noxauth/*` endpoints

## Licensing

NoxAeApi is a **licensed plugin**, protected via **NxGate** (`xyz.noxlydev.nxgate`):

- The plugin **will not start** without a valid `license-key` set in `config.yml`.
- License validation happens **online** when the server starts, and is **re-verified periodically in the background** (heartbeat).
- If the license server is temporarily unreachable, the plugin uses a **grace period** so your server doesn't go down due to a transient network issue.
- The license cache is HMAC-signed and bound to the machine and jar fingerprint to prevent tampering or key sharing.
- If a license is **revoked** while the server is running, the plugin reacts accordingly (e.g., disabling itself) rather than continuing to operate without a valid license.

Obtain your license key from the NxGate dashboard before installing the plugin.

---

## Configuration

Below is the full `config.yml` reference shipped with the plugin:

```yaml
# ── NxGate License ─────────────────────────────────────────────────────────
# Enter your license key from the NxGate dashboard. The plugin will not run
# without a valid key.
license-key: 'YOUR_LICENSE_KEY_HERE'

port: 4567
useKeyAuth: true
normalizeMessages: true

# ── Authentication ───────────────────────────────────────────────────────────
# Simple single key (default — used when no 'keys:' section is defined below).
key: 'change_me'

# Multi-key system — each key can have different permissions.
# When this section is present, the simple 'key' above is ignored.
# - value: the actual key string used in the API header
# - read-only: true means only GET requests are allowed
# - allowed-endpoints: restricts which paths this key can access (empty = all)
#
# keys:
#   admin:
#     value: 'change_me_admin'
#     read-only: false
#     allowed-endpoints: []
#   monitor:
#     value: 'change_me_monitor'
#     read-only: true
#     allowed-endpoints: []
#   luckperms-only:
#     value: 'change_me_luckperms'
#     read-only: false
#     allowed-endpoints:
#       - '/v1/luckperms/*'

# ── TLS ───────────────────────────────────────────────────────────────────────
tls:
  enabled: false
  keystore: keystore.jks
  keystorePassword: ''
  sni: false

# ── CORS ──────────────────────────────────────────────────────────────────────
corsOrigins:
  - '*'

# ── Rate Limiting ─────────────────────────────────────────────────────────────
# Limits requests per API key (or IP if no key) per minute. Does not apply to WebSockets.
rateLimit:
  enabled: false
  requestsPerMinute: 60

# ── Security ──────────────────────────────────────────────────────────────────
security:
  ip-whitelist:
    enabled: false
    allowed-ips:
      - "127.0.0.1"
      # - "192.168.1.100"

  audit-log:
    enabled: false   # writes every request to plugins/NoxAeApi/logs/audit.log

# ── Discord Notifications ─────────────────────────────────────────────────────
discord:
  enabled: false
  webhook-url: ""
  server-name: "My Minecraft Server"
  # Available events: unauthorized-access, exec-command, op-player, deop-player,
  #                    whitelist-change, ip-blocked
  notify-on:
    - unauthorized-access
    - exec-command
    - op-player
    - deop-player
    - whitelist-change
    - ip-blocked

# ── Webhooks (Minecraft game events → outbound HTTP push) ─────────────────────
webhooks:
  enabled: false
  endpoints: []

# ── WebSocket Events ───────────────────────────────────────────────────────────
# Connect at: ws://<host>:<port>/v1/ws/events?key=<your_key>
# Events: player_join, player_quit, player_chat, player_death, player_kick
websocket:
  events:
    enabled: true

# ── Misc ───────────────────────────────────────────────────────────────────────
debug: false
maxLogLines: 500       # set to 0 to disable console log buffering

# Enables NoxAuth data in player responses and the /v1/noxauth/* endpoints.
# Requires the NoxAuth plugin installed and loaded on the same server.
noxauth:
  enabled: false

disable-swagger: false

# Block specific paths (use the exact paths shown by swagger).
blocked-paths:
# - /v1/server/*

# ── Network Aggregator ─────────────────────────────────────────────────────────
# Only enable on your master/lobby server. Backend servers (survival, skyblock, etc)
# should leave this disabled.
network:
  enabled: false
  timeout: 5000   # ms timeout per backend request
  servers:
  # - id: survival
  #   label: "Survival"
  #   url: "http://10.0.0.2:4567"
  #   key: "secret_survival"
  # - id: skyblock
  #   label: "Skyblock"
  #   url: "http://10.0.0.3:4567"
  #   key: "secret_skyblock"
```

---

## Authentication

Every request must include a valid API key, either via header or query parameter:

```bash
# Via header
curl -H "key: your_secret_key_here" http://your-server:4567/v1/ping

# Via query parameter
curl "http://your-server:4567/v1/ping?key=your_secret_key_here"
```

NoxAeApi supports two authentication modes:

1. **Single key** — set `key:` in `config.yml`. Full access for any request presenting that key.
2. **Multi-key** — define a `keys:` map, where each named key can be:
   - **read-only**: restricted to `GET` requests only.
   - **scoped**: restricted to a list of `allowed-endpoints` (wildcards like `/v1/luckperms/*` are supported).

   When `keys:` is present, the simple `key:` value is ignored.

Additional protections you can layer on top:
- **IP whitelist** — only allow specific source IPs.
- **Rate limiting** — cap requests per minute per key/IP (does not apply to WebSocket connections).
- **Blocked paths** — explicitly disable specific endpoints regardless of key permissions.
- **Audit log** — every request is recorded to `plugins/NoxAeApi/logs/audit.log` when enabled.

---

## API Endpoints

### Server
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/ping` | Health check |
| GET | `/v1/server` | Server information |
| POST | `/v1/server/exec` | Execute a console command |
| POST | `/v1/server/restart` | Restart the server |
| GET | `/v1/server/logs` | Read recent console logs |
| GET | `/v1/server/entities` | List loaded entities |
| GET | `/v1/server/chunks` | List loaded chunks |
| POST | `/v1/server/ban-ip` | Ban an IP address |
| GET | `/v1/server/ops` | List server operators |
| POST | `/v1/server/ops` | Add an operator |
| DELETE | `/v1/server/ops` | Remove an operator |
| GET | `/v1/server/whitelist` | List whitelisted players |
| POST | `/v1/server/whitelist` | Add a player to the whitelist |
| DELETE | `/v1/server/whitelist` | Remove a player from the whitelist |

### Players
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/players` | List online players |
| GET | `/v1/players/all` | List all known players |
| GET | `/v1/players/{uuid}` | Get player by UUID |
| GET | `/v1/players/{playerUuid}/{worldUuid}/inventory` | Get player inventory |
| POST | `/v1/players/{uuid}/kick` | Kick a player |
| POST | `/v1/players/{uuid}/ban` | Ban a player |
| DELETE | `/v1/players/{uuid}/ban` | Unban a player |
| POST | `/v1/players/{uuid}/teleport` | Teleport a player |
| POST | `/v1/players/{uuid}/gamemode` | Change a player's gamemode |
| GET | `/v1/players/{uuid}/stats` | Get player statistics |

### Chat
| Method | Endpoint | Description |
|--------|----------|--------------|
| POST | `/v1/chat/broadcast` | Broadcast a message to the whole server |
| POST | `/v1/chat/tell` | Send a private message to a player |

### Economy (requires Vault)
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/economy` | Economy plugin info |
| POST | `/v1/economy/pay` | Deposit to a player's account |
| POST | `/v1/economy/debit` | Withdraw from a player's account |
| GET | `/v1/economy/balance/{uuid}` | Get a player's balance |
| GET | `/v1/economy/top` | Get the top-balance leaderboard |

### LuckPerms (requires LuckPerms)
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/luckperms/groups` | List all groups |
| GET | `/v1/luckperms/group/{name}/permissions` | Get group permissions |
| GET | `/v1/luckperms/player/{uuid}/groups` | Get a player's groups |
| GET | `/v1/luckperms/player/{uuid}/permissions` | Get a player's permissions |
| POST | `/v1/luckperms/player/{uuid}/permission` | Add a permission to a player |
| DELETE | `/v1/luckperms/player/{uuid}/permission` | Remove a permission from a player |
| POST | `/v1/luckperms/player/{uuid}/group` | Set a player's primary group |
| GET | `/v1/luckperms/player/{uuid}/check-permission` | Check if a player has a specific permission |

### Worlds
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/worlds` | List all worlds |
| GET | `/v1/worlds/{uuid}` | Get world info |
| POST | `/v1/worlds/save` | Save all worlds |
| POST | `/v1/worlds/{uuid}/save` | Save a specific world |
| GET | `/v1/worlds/download` | Download all worlds |
| GET | `/v1/worlds/{uuid}/download` | Download a specific world |
| POST | `/v1/worlds/{uuid}/time` | Set a world's time |
| POST | `/v1/worlds/{uuid}/weather` | Set a world's weather |
| GET | `/v1/worlds/{uuid}/entities` | List entities in a world |

### Plugins
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/plugins` | List installed plugins |
| POST | `/v1/plugins/{name}/enable` | Enable a plugin |
| POST | `/v1/plugins/{name}/disable` | Disable a plugin |

### Advancements & Scoreboards
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/advancements` | List advancements |
| GET | `/v1/scoreboard` | List scoreboards |
| GET | `/v1/scoreboard/{name}` | Get a scoreboard by name |
| POST | `/v1/scoreboard/{name}/score` | Set a score on an objective |
| DELETE | `/v1/scoreboard/{name}/score` | Remove a score from an objective |

### PlaceholderAPI
| Method | Endpoint | Description |
|--------|----------|--------------|
| POST | `/v1/placeholders/replace` | Replace PlaceholderAPI placeholders for a player |

### NoxAuth *(optional, requires NoxAuth plugin + `noxauth.enabled: true`)*
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/noxauth/player/{name}` | Get auth info (registered, authenticated, last login, country) |
| POST | `/v1/noxauth/player/{name}/check-password` | Check if a password matches a player's stored credentials |

### Network Aggregator *(optional, requires `network.enabled: true`)*
| Method | Endpoint | Description |
|--------|----------|--------------|
| GET | `/v1/network/status` | Status of all configured backend servers (parallel) |
| GET | `/v1/network/status/{id}` | Status of one backend server |
| GET | `/v1/network/players` | Aggregated online players from all servers |
| GET | `/v1/network/players/{uuid}` | Find which server a player is currently on |
| GET | `/v1/network/health` | Aggregated health info from all servers |
| POST | `/v1/network/broadcast` | Broadcast a message to all backend servers |
| GET/POST/PUT/DELETE | `/v1/network/{id}/*` | Forward a request directly to a backend server |

> All endpoints are documented interactively in [Swagger UI](#swagger-ui), including request/response models.

---

## WebSocket

NoxAeApi exposes two WebSocket endpoints (authenticated the same way as REST, via `?key=` query parameter):

| Path | Description |
|------|--------------|
| `ws://<host>:<port>/v1/ws/console` | Real-time stream of console log lines |
| `ws://<host>:<port>/v1/ws/events` | Real-time stream of game events: `player_join`, `player_quit`, `player_chat`, `player_death`, `player_kick` |

Example:
```js
const ws = new WebSocket("ws://your-server:4567/v1/ws/events?key=your_secret_key_here");
ws.onmessage = (msg) => console.log(JSON.parse(msg.data));
```

WebSocket connections are **not** subject to the `rateLimit` setting.

---

## Webhooks (Outbound)

Push the same game events available over WebSocket (`player_join`, `player_quit`, `player_chat`, `player_death`, `player_kick`) to your own HTTP endpoint(s), useful for triggering automations in n8n, Discord bots, or custom backends.

```yaml
webhooks:
  enabled: true
  endpoints:
    - url: "https://your-service.example.com/webhook"
      # secret: "optional-signing-secret"
```

---

## Discord Notifications

NoxAeApi can send alerts to a Discord channel via webhook URL whenever sensitive actions occur:

- `unauthorized-access` — failed authentication attempts
- `exec-command` — console command executed via the API
- `op-player` / `deop-player` — operator changes
- `whitelist-change` — whitelist additions/removals
- `ip-blocked` — requests blocked by the IP whitelist

Configure under the `discord:` section in `config.yml`, including your `webhook-url` and which events to `notify-on`.

---

## Multi-Server Network Aggregator

For networks running multiple backend servers (survival, skyblock, etc.) behind a hub/lobby server, NoxAeApi can act as an aggregator:

1. Enable `network.enabled: true` **only** on the hub/master server.
2. Register each backend server under `network.servers`, with its own `id`, `url`, and `key` (the API key of that backend's own NoxAeApi instance).
3. Backend servers themselves should keep `network.enabled: false` — they just run normal NoxAeApi instances.
4. The hub exposes aggregate endpoints (`/v1/network/status`, `/v1/network/players`, `/v1/network/health`, `/v1/network/broadcast`) plus a generic proxy (`/v1/network/{id}/*`) to forward arbitrary requests to a specific backend.

`network.timeout` controls how long (ms) the hub waits for each backend before treating it as unreachable.

---

## Swagger UI

Interactive, browsable API documentation (OpenAPI) is available at:

```
http://your-server:4567/swagger
```

Disable it by setting `disable-swagger: true` in `config.yml`.

---

## In-Game Commands

| Command | Aliases | Description |
|---------|---------|--------------|
| `/noxaeapi reload` | `/nae reload`, `/noxae reload` | Reload the plugin configuration |
| `/noxaeapi about` \| `info` \| `version` | — | Show plugin version, website, authors, and current settings (port, auth, TLS, NoxAuth, network) |

All subcommands require the `noxaeapi.admin` permission.

---

## Permissions

| Permission | Description |
|------------|--------------|
| `noxaeapi.admin` | Allows use of the `/noxaeapi` command and its subcommands |

---

## Building From Source

```bash
mvn clean package
```

The build pipeline:
1. **Compile** — Java 21, with the Javalin OpenAPI annotation processor generating the Swagger spec at compile time.
2. **Shade** — `maven-shade-plugin` bundles all dependencies into a single fat jar.
3. **Obfuscate** — `proguard-maven-plugin` obfuscates `net.wumx.noxaeapi.**` and `xyz.noxlydev.nxgate.**` classes only (third-party libraries inside the fat jar are passed through untouched), producing `target/NoxAeApi-<version>-obfuscated.jar`.

Use the `-obfuscated.jar` artifact for production deployments.

---

## Built With

- [Javalin 6.7.0](https://javalin.io/) — HTTP framework
- [Jetty 11](https://www.eclipse.org/jetty/) — embedded web server (HTTP + WebSocket)
- [Jackson](https://github.com/FasterXML/jackson) — JSON processing
- [LuckPerms API 5.4](https://luckperms.net/) — permission integration
- [VaultAPI 1.7](https://github.com/MilkBowl/VaultAPI) — economy integration
- [PlaceholderAPI 2.11.5](https://github.com/PlaceholderAPI/PlaceholderAPI) — placeholder integration
- [Item-NBT-API](https://github.com/tr7zw/Item-NBT-API) — inventory/item serialization
- [NxGate](https://www.noxlydev.xyz) — license validation
- ProGuard 7.4.2 — code obfuscation for release builds

---

## Version

`0.6.5-pre-relase` — Built for Minecraft API 1.21+, Java 21+.
