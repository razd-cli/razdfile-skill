# Razdfile.yml Reference

Complete field reference for all Razdfile.yml sections.

## Top-Level Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `version` | string | **Yes** | Schema version. Must be `"1"`. |
| `dependencies` | object | No* | Unified dependencies section. Mutually exclusive with `mise`/`devbox`. |
| `mise` | object | No* | Mise-native configuration. Mutually exclusive with `dependencies`. |
| `devbox` | object | No* | Devbox-native configuration. Mutually exclusive with `dependencies`. |
| `tasks` | object | No* | Task definitions. |
| `env` | object | No | Global environment variables. |
| `vars` | object | No | Global variables. |
| `includes` | object | No | Imported taskfiles. |
| `silent` | bool | No | Suppress task name/command output. Default: `false`. |
| `dotenv` | list | No | List of `.env` file paths. |
| `method` | string | No | Up-to-date check: `"none"`, `"checksum"`, `"timestamp"`. |
| `output` | string | No | Output mode: `"interleaved"`, `"prefixed"`, `"group"`. |
| `run` | string | No | Run mode: `"always"`, `"once"`, `"when_changed"`. |
| `set` | list | No | POSIX shell options. |
| `shopt` | list | No | Bash shell options. |

*At least one of `tasks`, `mise`, `devbox`, or `dependencies` must be present.

## dependencies

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `using` | string | **Yes** | `"mise"` or `"devbox"`. |
| `ensure` | list | No | Dependencies in `tool@version` format. |
| `extra.mise` | object | No | Native mise config pass-through (when `using: mise`). |
| `extra.devbox` | object | No | Native devbox config pass-through (when `using: devbox`). |

**ensure format:** `^[a-z][a-z0-9_-]*@[a-zA-Z0-9._-]+$`

## mise

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tools` | object | No | Tool definitions. Keys=tool names, values=version string/array/object. |
| `env` | object | No | Environment variables. |
| `settings` | object | No | Mise settings (`auto_install`, `experimental`, `quiet`, `verbose`, `jobs`). |
| `hooks` | object | No | Lifecycle hooks. |
| `plugins` | object | No | Custom plugin URLs (name→URL). |
| `min_version` | string | No | Minimum required mise version. |

**Tool value formats:**
- String: `node: "22"` → simple version
- Array: `node: ["20", "22"]` → multiple versions, first is primary
- Object: `node: {version: "22", os: [linux, darwin], postinstall: "npm i -g npm"}` → full config

## devbox

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | string | No | Project name. |
| `description` | string | No | Project description. |
| `packages` | list or object | No | Package definitions. Array: `["node@22"]`, Object: `{node: {version: "22"}}`. |
| `env` | object | No | Environment variables (all string values). |
| `shell` | object | No | Shell config with `init_hook` (string or list) and `scripts` (name→command). |
| `include` | list | No | Additional plugins. |
| `env_from` | string | No | File path to load env vars from. |
| `nixpkgs` | object | No | `{commit: "hash"}` for nixpkgs pinning. |

## tasks

Each task is a named object under `tasks:`:

| Field | Type | Description |
|-------|------|-------------|
| `cmds` | list | Commands to execute sequentially. |
| `cmd` | string | Single command shorthand (alternative to `cmds`). |
| `deps` | list | Task dependencies to run in parallel first. |
| `desc` | string | Short description (shown in `razd run --list`). |
| `env` | object | Task-scoped environment variables. |
| `vars` | object | Task-scoped variables. |
| `silent` | bool | Suppress output. Default: `false`. |
| `dir` | string | Working directory. |
| `sources` | list | Source file globs for up-to-date checks. |
| `generates` | list | Output file globs. |
| `status` | list | Commands to check if task should run. |
| `platforms` | list | OS platforms this task runs on. |
| `internal` | bool | Hide from CLI listing. Default: `false`. |
| `interactive` | bool | Mark as interactive. Default: `false`. |
| `ignore_error` | bool | Continue on error. Default: `false`. |

**Shorthand:** A task value can be a string (single command) or list of strings instead of an object.

**Task call in cmds:**
```yaml
cmds:
  - echo "running"
  - task: install      # Call another task
  - task: build
      vars: {NODE_ENV: production}
```