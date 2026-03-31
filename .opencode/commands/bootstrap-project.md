---
description: Scaffold a new project from zero to a startable dev container with OpenCode
agent: change-orchestrator
---

# Bootstrap Project

You are scaffolding a brand-new project inside this governance workspace. The `project/` directory should be empty or not yet exist. Follow four phases: Discovery, Recommendation, Confirmation, Scaffold.

---

## Phase 1: Discovery

Your goal is to understand what the user is building, who it is for, and what it needs to do. Ask **business and requirement questions**, not technical ones. You will derive the technology choices yourself in Phase 2.

Use the `question` tool. Ask all initial questions in a **single batch**. Use open-ended questions for most; use choices only when the options genuinely help the user think (not to quiz them on tech).

### Initial questions

1. **Project name** (open-ended)
   What should we call this project? Used as directory name and display name. Lowercase, hyphen-separated (e.g., `my-app`).

2. **What does this project do?** (open-ended)
   Describe the product in a few sentences. What problem does it solve? What is the core value?

3. **Who are the users?** (open-ended)
   Who will use this? Internal team, external customers, developers consuming an API, etc.? Roughly how many?

4. **Core capabilities** (open-ended)
   What are the 3-5 most important things a user needs to be able to do?

5. **Integrations and environment** (open-ended)
   Does this need to connect to existing systems, APIs, or data sources? Will it run in a specific environment (cloud provider, on-prem, edge)?

6. **Constraints and preferences** (open-ended)
   Anything the team already knows: preferred languages, existing infrastructure, team expertise, compliance requirements, timeline pressure, corporate network restrictions (e.g., proxy or CA certificates)?

### Follow-up questions

After the initial answers, assess whether you have enough information to make confident technology recommendations. If not, ask **targeted follow-up questions** -- for example:

- If the user mentions "real-time updates" but did not specify how, ask whether they mean live dashboards, chat, notifications, or collaborative editing.
- If the user says "internal tool" but did not mention auth, ask whether users log in and how (SSO, shared credentials, open access).
- If capabilities imply a UI but the user did not describe it, ask whether this is a web app, mobile app, CLI, or headless service.
- If they mention data but not persistence, ask whether data is transient, needs to survive restarts, or has relational structure.

Keep follow-ups minimal and focused. Do not ask about things you can confidently infer from the answers. When you have enough context to make a defensible recommendation, move to Phase 2.

---

## Phase 2: Recommendation

Derive the full technology stack from the business requirements gathered in Phase 1. For every technical choice, tie the rationale back to something the user said.

Present:

1. **Tech stack summary** -- table: Layer | Technology | Why (traced to user requirement)
2. **Directory structure** -- the `project/` tree that will be created
3. **Dev container** -- base image, installed tools, forwarded ports, env vars, initialization sequence
4. **Key dependencies** -- initial packages/libraries
5. **Governance seeding** -- how vocabulary, test commands, and product rules will be customized from what you learned about the domain

If any choice is a close call between two options, mention the trade-off briefly so the user can override it.

End with: _Say **implement** to scaffold, or tell me what you would change._

---

## Phase 3: Confirmation

Wait for the user to say "implement" (case-insensitive). If they refine or override specific choices, update the plan and re-present Phase 2. Do **not** create any files until confirmed.

---

## Phase 4: Scaffold

On "implement", create all files under `project/`. Read templates from `.opencode/templates/` and customize with the user's answers.

### 4.1 Create directories

```
project/
  .devcontainer/
  .opencode-overrides/agents/
  .opencode-overrides/rules/
  architecture/
  backlog/proposed/
  backlog/done/
  decision-log/
  scripts/
  spec/
  src/                    # adapt: apps/ for monorepo, lib/ for library, etc.
```

### 4.2 Dev container (3 files)

Read and customize `.opencode/templates/devcontainer/*`:

**`project/.devcontainer/devcontainer.json`** (from `devcontainer.json.template`)

- `name`: `"{project-name}-dev"`
- `forwardPorts`: from port defaults table below
- `remoteUser` / `containerUser`: match the `REMOTE_USER` ARG in the Dockerfile (`node` for javascript-node images, `vscode` for python/go/java/base images)
- `postCreateCommand`: include an explicit initialization chain in this order:
  1. `bash scripts/devcontainer-fix-home-perms.sh`
  2. `bash scripts/devcontainer-install-opencode.sh`
  3. project dependency install commands (backend first, then frontend when both exist)
- `postStartCommand`: run `bash scripts/devcontainer-fix-home-perms.sh` so permission drift from mounts is repaired on each start
- Keep all OpenCode mounts (data volume, state volume, auth.json bind) -- these are mandatory
- Add quality-of-life mounts when relevant:
  - SSH mount (`${localEnv:HOME}/.ssh`)
  - shell history named volume (`/commandhistory`)
  - netrc mount (`${localEnv:HOME}/.netrc`)
  - Playwright cache volume (`/home/<remote-user>/.cache/ms-playwright`)
- Keep the `sst-dev.opencode` VSCode extension

**`project/docker-compose.devcontainer.yml`** (from `docker-compose.devcontainer.yml.template`)

- Keep the governance bridge (volume `..:/workspaces/workspace:cached`, env vars `OPENCODE_CONFIG_DIR`, `OPENCODE_CONFIG`) -- these are mandatory
- Keep `command: sleep infinity` so editors can attach reliably
- Add database service block if a database was chosen (see database services below)
- Add `DATABASE_URL` env var if database chosen (see URL formats below)
- Expose database ports

**`project/.devcontainer/Dockerfile`** (from `Dockerfile.template`)

- `BASE_IMAGE`: from base image defaults table below
- `REMOTE_USER`: `node` for javascript-node images, `vscode` for python/go/java/base images
- Uncomment and fill the CA cert section if corporate cert is needed
- Add project runtime tools (package manager, language tools) in the marked section
- If a corporate CA certificate is configured, set both `NODE_EXTRA_CA_CERTS` and `REQUESTS_CA_BUNDLE`
- Pre-create and `chown` runtime directories for OpenCode and common tooling:
  - `/home/${REMOTE_USER}/.cache`
  - `/home/${REMOTE_USER}/.cache/ms-playwright`
  - `/home/${REMOTE_USER}/.local/share/opencode`
  - `/home/${REMOTE_USER}/.local/state/opencode`
  - `/home/${REMOTE_USER}/.local/share/uv/tools` (when Python tooling is present)
- **Do not remove** the OpenCode CLI section -- it bakes OpenCode into the image (Layer 1 of 2)
- `WORKDIR /workspaces/workspace/project`

### 4.3 Permission repair script

**`project/scripts/devcontainer-fix-home-perms.sh`** (from `.opencode/templates/scripts/devcontainer-fix-home-perms.sh.template`)

Copy the template and `chmod +x`. This script must be idempotent and safe to run in both `postCreateCommand` and `postStartCommand`.

The script must:

- `mkdir -p` required runtime directories under `/home/${REMOTE_USER}`
- `chown -R ${REMOTE_USER}:${REMOTE_USER}` over `/home/${REMOTE_USER}/.cache` and `/home/${REMOTE_USER}/.local`
- print concise status messages before and after applying fixes

### 4.4 OpenCode install script

**`project/scripts/devcontainer-install-opencode.sh`** (from `.opencode/templates/scripts/devcontainer-install-opencode.sh.template`)

Copy the template and `chmod +x`. This is Layer 2 of 2 -- it runs at container creation via `postCreateCommand` and re-installs OpenCode if the Dockerfile-baked binary is missing or stale. The script:

- Exits early if `opencode` is already on PATH (fast path)
- Supports corporate CA certs via `OPENCODE_CA_CERT` env var
- Falls back from curl to npm if curl install fails
- Never blocks container creation (exits 0 on failure with a warning)

### 4.5 Governance overrides (3 files)

Read and customize `.opencode/templates/overrides/*` and `.opencode/templates/product-guidelines.md.template`:

**`project/.opencode-overrides/agents/backlog-planner.override.md`**

- Seed vocabulary with 3-5 domain terms inferred from the project description
- Leave personas and themes sections as commented starters

**`project/.opencode-overrides/agents/change-orchestrator.override.md`**

- Fill test commands table with correct commands for chosen stack and package manager
- Fill CI-equivalent validation block (lint, build, test commands)

**`project/.opencode-overrides/rules/product-guidelines.md`**

- Set package manager name and lockfile validation command
- Set localization rules placeholder
- Fill implementation gate with stack-appropriate validation commands

### 4.6 Architecture docs (3 files)

Read and customize `.opencode/templates/architecture/*`:

**`project/architecture/overview.md`**

- System Shape: list chosen components (frontend, backend, database, auth)
- Technology Stack: fill table from tech stack summary
- Local Development: correct dev/test/build commands

**`project/architecture/codebase-map.md`**

- Map every directory and file created during scaffold

**`project/architecture/domain-glossary.md`**

- Seed with the same vocabulary terms used in backlog-planner override

### 4.7 Governance scaffolding (3 files)

Copy templates directly (minimal customization):

**`project/backlog/README.md`** (from `backlog-README.md.template`) -- next ID: BG-001
**`project/spec/README.md`** (from `spec-README.md.template`)
**`project/decision-log/README.md`** (from `decision-log-README.md.template`)

### 4.8 Project files

**`project/README.md`** (from `project-README.md.template`)

- Fill: project name, description, prerequisites, getting started, project structure, scripts, env vars, ports

**`project/.gitignore`** -- appropriate for the chosen stack (node_modules, dist, .env, **pycache**, etc.)

If using `pnpm`, include a non-interactive build-script approval strategy so dev container startup is repeatable:

- Prefer `pnpm.onlyBuiltDependencies` in `package.json` for required native packages (for example `esbuild` in Vite stacks)
- Use interactive `pnpm approve-builds` only as a local one-off fallback

**Stack config files** -- create the minimal set for the chosen stack:

- JS/TS: `package.json` (name, scripts stubs, no dependencies yet), `tsconfig.json` if TypeScript
- Python: `pyproject.toml`
- Go: `go.mod`
- Java: `pom.xml` or `build.gradle`
- Linter/formatter config as appropriate (e.g., `.eslintrc`, `.prettierrc`, `ruff.toml`)

### 4.9 Git initialization

If `project/` does not already have a `.git` directory:

- Run `git init` inside `project/`
- Create an initial commit with the scaffolded files

---

## Reference tables

### Base images

| Stack                       | Base image                                           |
| --------------------------- | ---------------------------------------------------- |
| Node.js / Full-stack (Node) | `mcr.microsoft.com/devcontainers/javascript-node:20` |
| Python                      | `mcr.microsoft.com/devcontainers/python:3.12`        |
| Go                          | `mcr.microsoft.com/devcontainers/go:1.22`            |
| Java                        | `mcr.microsoft.com/devcontainers/java:21`            |

### Default ports

| Service                         | Port  |
| ------------------------------- | ----- |
| React / Vue / Svelte dev server | 5173  |
| Next.js / Nuxt dev server       | 3000  |
| Node.js API server              | 3001  |
| Python API (uvicorn / gunicorn) | 8000  |
| Go API                          | 8080  |
| Java API (Spring Boot)          | 8080  |
| PostgreSQL                      | 5432  |
| MySQL                           | 3306  |
| MongoDB                         | 27017 |
| Redis                           | 6379  |

### Database docker-compose services

**PostgreSQL:**

```yaml
db:
  image: postgres:16
  restart: unless-stopped
  environment:
    POSTGRES_USER: dev
    POSTGRES_PASSWORD: dev
    POSTGRES_DB: PROJECT_NAME
  ports:
    - "5432:5432"
  volumes:
    - pgdata:/var/lib/postgresql/data
# add to top-level volumes:
#   pgdata:
```

**MySQL / MariaDB:**

```yaml
db:
  image: mysql:8
  restart: unless-stopped
  environment:
    MYSQL_ROOT_PASSWORD: dev
    MYSQL_DATABASE: PROJECT_NAME
    MYSQL_USER: dev
    MYSQL_PASSWORD: dev
  ports:
    - "3306:3306"
  volumes:
    - mysqldata:/var/lib/mysql
# add to top-level volumes:
#   mysqldata:
```

**MongoDB:**

```yaml
db:
  image: mongo:7
  restart: unless-stopped
  environment:
    MONGO_INITDB_ROOT_USERNAME: dev
    MONGO_INITDB_ROOT_PASSWORD: dev
  ports:
    - "27017:27017"
  volumes:
    - mongodata:/data/db
# add to top-level volumes:
#   mongodata:
```

### DATABASE_URL formats

| Database   | URL                                                        |
| ---------- | ---------------------------------------------------------- |
| PostgreSQL | `postgresql://dev:dev@db:5432/PROJECT_NAME`                |
| MySQL      | `mysql://dev:dev@db:3306/PROJECT_NAME`                     |
| MongoDB    | `mongodb://dev:dev@db:27017/PROJECT_NAME?authSource=admin` |
| SQLite     | `file:./dev.db`                                            |

---

## Output guarantee

After scaffolding, the user must be able to:

1. Open the governance workspace root in VS Code / Cursor
2. "Reopen in Container" -- the dev container builds and starts
3. The container has internet access (CA cert installed if selected)
4. OpenCode is installed and loads the governance framework (agents, rules, skills, commands)
5. `project/` has a valid structure with all governance scaffolding in place
6. Dev container startup is repeatable with no manual permission repair and no interactive package-manager prompts

Verify by listing the created file tree and confirming every expected file exists. Report the full list to the user.
