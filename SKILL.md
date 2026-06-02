---
name: razdfile-skill
description: "Generate and validate Razdfile.yml configuration files for razd projects. Use when creating, editing, or validating Razdfile.yml, or when setting up project provisioning with mise or devbox. Triggers on mentions of Razdfile, razdfile, razd init, razd config, provisioning config, or task runner setup."
argument-hint: "[description or path]"
disable-model-invocation: false
user-invocable: true
allowed-tools: Read Write Bash
license: MIT
metadata:
  author: razd-cli
  version: "1.1"
  category: configuration
---

# razdfile-skill — Razdfile.yml Generator

You are a Razdfile.yml configuration generator. Create valid `Razdfile.yml` files based on user requirements.

## What is Razdfile.yml?

`Razdfile.yml` is the configuration file for [razd](https://github.com/razd-cli/razd) — a project environment provisioning CLI. It defines:

- **Provisioner** — mise or devbox (how tools are installed)
- **Dependencies** — which language runtimes and tools to install
- **Tasks** — commands to run (build, test, dev, etc.)
- **Environment** — env vars, settings, hooks

## Supported Version

Only `version: "1"` is valid. Always include it as the first field.

## Only Valid Format: `dependencies`

ALL Razdfile.yml files MUST use the `dependencies` section. The `mise` and `devbox` top-level sections are NOT allowed.

```yaml
version: "1"
dependencies:
  using: "mise"          # REQUIRED: "mise" or "devbox"
  ensure:                # Package list in tool@version format
    - "node@22"
    - "pnpm@latest"
  extra:                  # OPTIONAL: native provider config pass-through
    mise:
      env:
        NODE_ENV: development
      settings:
        experimental: true
tasks:
  default:
    cmds:
      - npm run dev
```

### With devbox provider

```yaml
version: "1"
dependencies:
  using: "devbox"         # REQUIRED: "mise" or "devbox"
  ensure:
    - "nodejs@22"
  extra:
    devbox:
      shell:
        init_hook:
          - echo "Welcome!"
tasks:
  default:
    cmds:
      - npm run dev
```

## Mutual Exclusion Rules

- `dependencies` + `mise` section = ERROR (never use `mise:` as top-level key)
- `dependencies` + `devbox` section = ERROR (never use `devbox:` as top-level key)
- `dependencies` alone = OK (use `using` field to specify provider)
- NEVER generate `mise:` or `devbox:` as top-level sections

## Generation Rules

### When to Generate Immediately (no questions)

If the user provides:
- Project language/runtime (e.g., "Node.js project", "Python app")
- A general description (e.g., "Razdfile for a Go microservice")
- Tool list (e.g., "needs Node 22 and pnpm")

Then IMMEDIATELY generate the Razdfile.yml. DO NOT ask for more details.

### Fast Path Examples

- "Create a Razdfile for a Node.js project" → generate with `dependencies.using: mise, ensure: [node@22, pnpm@latest]`
- "Razdfile for Python with devbox" → generate with `dependencies.using: devbox, ensure: [python@3.11]`
- "My Go project needs task runner" → generate with `dependencies.using: mise, ensure: [go@1.23, task@latest]`

### Slow Path

Only if the user says something very vague like "create a Razdfile" without any context:
1. Ask what language/runtime
2. Ask which provisioner (mise or devbox, default: mise)
3. IMMEDIATELY generate after getting answers

**NEVER ask for confirmation before generating.**

## Validation Rules

When generating or reviewing a Razdfile.yml, enforce these rules:

1. **`version` is ALWAYS `"1"`** — first field, required
2. **Always use `dependencies` section** — never `mise:` or `devbox:` as top-level keys
3. **`dependencies.using` is required** — must be `"mise"` or `"devbox"`
4. **`dependencies.ensure` format** — each entry must match `tool@version` (e.g., `node@22`, `go@1.21`, `pnpm@latest`)
5. **No `dependencies` + `mise` section** — mutually exclusive
6. **No `dependencies` + `devbox` section** — mutually exclusive
7. **At least one content section** — `tasks` or `dependencies` must be present
8. **Use `extra` for provider-specific config** — mise env/settings go in `dependencies.extra.mise`, devbox shell/scripts go in `dependencies.extra.devbox`

## Task Reference

Tasks follow the [Taskfile](https://taskfile.dev/) format. Key fields:

```yaml
tasks:
  default:          # Task name
    desc: "Description shown in task list"
    cmds:           # Commands to run (sequential)
      - echo "Hello"
      - task: install  # Call another task
    deps: [build]   # Run dependencies in parallel first
    env:            # Task-scoped env vars
      NODE_ENV: development
    silent: false   # Suppress output
    sources:        # Up-to-date check sources
      - src/**/*.js
    generates:      # Up-to-date check outputs
      - dist/
```

### Task shorthand

```yaml
tasks:
  install: npm install        # String shorthand
  build: npm run build        # String shorthand
  test:                       # Object form
    desc: "Run tests"
    cmds:
      - npm test
```

## Tool Version Reference

Common tools and their version patterns for `dependencies.ensure`:

| Tool | Example Entry | Notes |
|------|--------------|-------|
| Node.js | `node@22` | Major version |
| Python | `python@3.11` | Major.minor |
| Go | `go@1.23` | Major.minor |
| Rust | `rust@latest` | Usually latest |
| Ruby | `ruby@3.3` | Major.minor |
| pnpm | `pnpm@latest` | Package manager |
| npm | `npm@latest` | Package manager |
| task | `task@latest` | Task runner (go-task) |
| bun | `bun@latest` | JS runtime |
| zig | `zig@latest` | Systems language |
| lua | `lua@5.4` | Major.minor |

## File Naming

The file must be named one of (in priority order):
1. `Razdfile.yml`
2. `Razdfile.yaml`
3. `razdfile.yml`
4. `razdfile.yaml`

Always use `Razdfile.yml` unless the user specifies otherwise.

## Templates

### Node.js project (mise)

```yaml
version: "1"
dependencies:
  using: "mise"
  ensure:
    - "node@22"
    - "pnpm@latest"
tasks:
  default:
    desc: "Set up and run development"
    cmds:
      - task: install
      - task: dev
  install:
    desc: "Install dependencies"
    cmds:
      - pnpm install
  dev:
    desc: "Start development server"
    cmds:
      - pnpm dev
  build:
    desc: "Build for production"
    cmds:
      - pnpm build
  test:
    desc: "Run tests"
    cmds:
      - pnpm test
```

### Python project (mise)

```yaml
version: "1"
dependencies:
  using: "mise"
  ensure:
    - "python@3.11"
    - "task@latest"
  extra:
    mise:
      env:
        PYTHONPATH: "."
tasks:
  default:
    desc: "Set up and run"
    cmds:
      - task: install
      - task: dev
  install:
    desc: "Install dependencies"
    cmds:
      - pip install -r requirements.txt
  dev:
    desc: "Run development server"
    cmds:
      - python app.py
  test:
    desc: "Run tests"
    cmds:
      - pytest
```

### Go project (mise)

```yaml
version: "1"
dependencies:
  using: "mise"
  ensure:
    - "go@1.23"
    - "task@latest"
tasks:
  default:
    desc: "Build and run"
    cmds:
      - task: build
      - ./bin/app
  build:
    desc: "Compile project"
    cmds:
      - go build -o bin/app .
  test:
    desc: "Run tests"
    cmds:
      - go test ./...
  fmt:
    desc: "Format code"
    cmds:
      - go fmt ./...
```

### Devbox project

```yaml
version: "1"
dependencies:
  using: "devbox"
  ensure:
    - "nodejs@22"
  extra:
    devbox:
      shell:
        init_hook:
          - echo "Welcome to the dev environment!"
tasks:
  default:
    desc: "Install and run"
    cmds:
      - task: install
      - task: dev
  install:
    desc: "Install dependencies"
    cmds:
      - npm install
  dev:
    desc: "Start development"
    cmds:
      - npm run dev
```

### Ruby project (mise)

```yaml
version: "1"
dependencies:
  using: "mise"
  ensure:
    - "ruby@3.3"
    - "task@latest"
tasks:
  default:
    desc: "Set up and run"
    cmds:
      - task: install
      - task: dev
  install:
    desc: "Install gems"
    cmds:
      - bundle install
  dev:
    desc: "Start development server"
    cmds:
      - rackup app.ru -p 9292
  test:
    desc: "Run tests"
    cmds:
      - bundle exec rspec
```

## Modifying Existing Files

When the user asks to modify an existing Razdfile.yml:

1. Read the current file
2. If it uses `mise:` or `devbox:` top-level sections, convert to `dependencies` format
3. Apply requested changes
4. Validate the result against the rules above
5. Never remove `version: "1"` or change it
6. Preserve the structure and comments

Common modifications:
- "Add python to dependencies" → add to `dependencies.ensure`
- "Switch from mise to devbox" → change `dependencies.using` to `"devbox"`, adjust `ensure` format
- "Add a build task" → add entry under `tasks`
- "Add environment variable" → add to `dependencies.extra.mise.env` or `dependencies.extra.devbox.env`

## Converting Legacy Formats

If an existing Razdfile uses `mise:` or `devbox:` as top-level sections, convert it:

**Before (invalid):**
```yaml
version: "1"
mise:
  tools:
    node: "22"
tasks:
  dev: npm run dev
```

**After (valid):**
```yaml
version: "1"
dependencies:
  using: "mise"
  ensure:
    - "node@22"
tasks:
  dev: npm run dev
```

## Validation Checklist

Before outputting any Razdfile.yml, verify:

- [ ] `version: "1"` is present as the first field
- [ ] `dependencies` section is present with `using` field
- [ ] `using` value is `"mise"` or `"devbox"`
- [ ] No `mise:` top-level section (use `dependencies.extra.mise` instead)
- [ ] No `devbox:` top-level section (use `dependencies.extra.devbox` instead)
- [ ] All `ensure` entries follow `tool@version` format
- [ ] At least one content section exists (tasks or dependencies with tasks)
- [ ] Task names are valid YAML identifiers
- [ ] `cmds` entries are strings or objects with `cmd`/`task` keys