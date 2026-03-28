# OpenCode Governance Framework

A portable AI development governance layer for software projects. Provides agent definitions, workflow rules, testing patterns, and project scaffolding templates that work with [OpenCode](https://opencode.ai).

## What This Framework Provides

- **Two-agent workflow**: Backlog-Planner (planning) and Change-Orchestrator (implementation) with a human-approval gate between them.
- **Durable rules**: implementation gates for frontend quality, security, accessibility, testing, and architectural documentation.
- **Testing patterns**: architecture-level checklists for API resolvers, data access, UI components, auth guards, and migrations.
- **Session bootstrap**: orientation protocol so agents build context at the start of every session.
- **Governance skills**: continuous-learning (detect friction, suggest improvements) and governance-authoring (correct taxonomy placement).
- **Project templates**: scaffolding for backlog, specs, architecture docs, decision logs, and product-specific rules.

## Repository Structure

```
.opencode/
  agents/
    backlog-planner.md          Framework agent: planning and backlog management
    change-orchestrator.md      Framework agent: implementation and testing
  rules/
    global-guidelines.md        Implementation gate (responsive, tokens, security, a11y, testing)
    session-bootstrap.md        Read-these-files-first protocol
    testing-patterns.md         Architecture-level test checklists
  skills/
    continuous-learning/        Detect friction, suggest governance improvements
    opencode-governance-authoring/  Taxonomy placement guidance
  templates/
    devcontainer/               Dev container templates (governance-compatible)
    overrides/                  Starter override files for agents
    architecture/               Architecture doc templates
    backlog-README.md.template
    spec-README.md.template
    decision-log-README.md.template
    product-guidelines.md.template
    project-README.md.template
opencode.json                   MCP servers + dual instruction loading
AGENTS.md                       Workspace convention + agent taxonomy
FRAMEWORK.md                    This file
.gitignore                      Excludes project/ directory
```

## Workspace Convention

This framework uses a **governance-outer, project-inner** model:

```
governance-workspace/           This repo (outer git)
  .opencode/                    Framework governance
  opencode.json                 Loads rules from both layers
  AGENTS.md                     Workspace convention
  project/                      Your project (inner git, separate remote)
    .opencode-overrides/        Product-specific agent overrides and rules
    architecture/               Product-specific docs
    backlog/                    Product-specific work items
    spec/                       Product-specific specs
    apps/                       Your application code
```

OpenCode is opened from the governance workspace root. Agents discover `.opencode/` naturally and operate on `project/` for all code and product-specific files.

## Adopting This Framework

### New Project

```bash
# 1. Clone the governance framework
git clone <this-repo-url> my-project-workspace
cd my-project-workspace

# 2. Clone or initialize your project
git clone <your-project-url> project/
# OR: mkdir project && cd project && git init

# 3. Scaffold product-specific governance from templates
mkdir -p project/.opencode-overrides/agents project/.opencode-overrides/rules project/architecture project/backlog/proposed project/backlog/done project/spec project/decision-log

cp .opencode/templates/product-guidelines.md.template project/.opencode-overrides/rules/product-guidelines.md
cp .opencode/templates/overrides/backlog-planner.override.md.template project/.opencode-overrides/agents/backlog-planner.override.md
cp .opencode/templates/overrides/change-orchestrator.override.md.template project/.opencode-overrides/agents/change-orchestrator.override.md
cp .opencode/templates/backlog-README.md.template project/backlog/README.md
cp .opencode/templates/spec-README.md.template project/spec/README.md
cp .opencode/templates/decision-log-README.md.template project/decision-log/README.md
cp .opencode/templates/architecture/codebase-map.md.template project/architecture/codebase-map.md
cp .opencode/templates/architecture/domain-glossary.md.template project/architecture/domain-glossary.md
cp .opencode/templates/architecture/overview.md.template project/architecture/overview.md
cp .opencode/templates/project-README.md.template project/README.md

# 4. Set up dev container (recommended)
mkdir -p project/.devcontainer
cp .opencode/templates/devcontainer/devcontainer.json.template project/.devcontainer/devcontainer.json
cp .opencode/templates/devcontainer/docker-compose.devcontainer.yml.template project/docker-compose.devcontainer.yml
cp .opencode/templates/devcontainer/Dockerfile.template project/.devcontainer/Dockerfile

# 5. Customize:
#    - project/.devcontainer/devcontainer.json: name, ports, postCreateCommand
#    - project/.devcontainer/Dockerfile: base image, runtime tools
#    - project/docker-compose.devcontainer.yml: environment variables, volumes
#    - project/.opencode-overrides/: vocabulary, test commands, product rules
#    - project/README.md: project description, scripts, env vars

# 6. Open project/ in VS Code, select "Reopen in Container"
# 7. Inside the container, start OpenCode -- it loads governance via OPENCODE_CONFIG_DIR
```

### Existing Project

If your project already has governance files in `.opencode/`, separate them:

1. Clone this framework as the outer workspace.
2. Move your project into `project/`.
3. Move product-specific rules to `project/.opencode-overrides/rules/`.
4. Move product-specific agent overrides to `project/.opencode-overrides/agents/`.
5. Remove framework-level agents, skills, and rules from the project (they now come from the workspace).
6. Remove any `opencode.json` and `AGENTS.md` from the project root (these now live in the governance wrapper).

### Existing Project with Dev Container

If your project already has a `.devcontainer/` setup, update it for governance compatibility:

1. **docker-compose volume mount** -- change the workspace mount to include the governance parent:

   ```yaml
   volumes:
     # Before: only project mounted
     - .:/workspaces/my-project:cached
     # After: governance root mounted (project is inside it)
     - ..:/workspaces/workspace:cached
   ```

2. **Add governance env vars** to the docker-compose environment:

   ```yaml
   environment:
     OPENCODE_CONFIG_DIR: /workspaces/workspace/.opencode
     OPENCODE_CONFIG: /workspaces/workspace/opencode.json
   ```

3. **Update workspaceFolder** in `devcontainer.json`:

   ```json
   "workspaceFolder": "/workspaces/workspace/project"
   ```

4. **Update Dockerfile WORKDIR** (if present):

   ```dockerfile
   WORKDIR /workspaces/workspace/project
   ```

5. **Update any absolute paths** in commands or volume mounts that referenced the old workspace location.

6. **Remove project-level `opencode.json` and `AGENTS.md`** if they exist -- all AI config now comes from the governance wrapper.

7. **Move `.opencode/` contents** to `.opencode-overrides/`:
   ```bash
   mkdir -p .opencode-overrides/agents .opencode-overrides/rules
   # Move product-specific files (NOT framework agents/rules/skills)
   mv .opencode/agents/*override* .opencode-overrides/agents/ 2>/dev/null
   mv .opencode/rules/*.md .opencode-overrides/rules/ 2>/dev/null
   # Remove leftover framework files from .opencode/
   rm -rf .opencode/agents .opencode/rules .opencode/skills
   ```

## Pulling Framework Updates

```bash
cd my-project-workspace/
git pull origin main
```

No conflicts -- the outer repo only tracks framework files. Product-specific content lives in `project/` (a separate git repo, excluded via `.gitignore`).

## Contributing Framework Improvements

When you improve governance while working in a project:

```bash
# Edit framework files at the workspace root
# (e.g., .opencode/rules/testing-patterns.md)
cd my-project-workspace/
git add .opencode/ AGENTS.md opencode.json
git commit -m "improve: add migration rollback test pattern"
git push origin main
```

Other workspaces pull the improvement with `git pull`.

## Product-Specific Overrides

Projects customize agent behavior via override files that live **outside** `.opencode/` so OpenCode does not auto-discover them as standalone agents:

| Override file                                                        | Customizes                                       |
| -------------------------------------------------------------------- | ------------------------------------------------ |
| `project/.opencode-overrides/agents/backlog-planner.override.md`     | Vocabulary table, personas, backlog theme values |
| `project/.opencode-overrides/agents/change-orchestrator.override.md` | Test commands, validation steps, CI commands     |
| `project/.opencode-overrides/rules/*.md`                             | Any product-specific rules                       |

Override files are read by agents at session start. They extend (not replace) framework behavior.

## Design Principles

1. **Governance wraps the project, not the other way around.** The project repo is self-contained for CI/CD. Governance is an AI development enhancement.
2. **Two git repos, zero conflicts.** Framework files (outer) and product files (inner) never share a git history.
3. **Convention over configuration.** The project always lives at `project/`. No config file needed to locate it.
4. **Override, don't fork.** Product-specific customization uses override files, preserving the ability to pull framework updates cleanly.

## Troubleshooting

### OpenCode does not discover agents

**Symptom**: agents are not listed or available in OpenCode.

**Cause**: the `OPENCODE_CONFIG_DIR` environment variable is not set or points to the wrong path.

**Fix**: verify your `docker-compose.devcontainer.yml` includes these environment variables:

```yaml
environment:
  OPENCODE_CONFIG_DIR: /workspaces/workspace/.opencode
  OPENCODE_CONFIG: /workspaces/workspace/opencode.json
```

Also verify the governance root is mounted in the container:

```yaml
volumes:
  - ..:/workspaces/workspace:cached
```

If running outside a dev container, set the env vars manually:

```bash
export OPENCODE_CONFIG_DIR=/path/to/governance-workspace/.opencode
export OPENCODE_CONFIG=/path/to/governance-workspace/opencode.json
```

### Product-specific rules not loading

**Symptom**: agents do not follow product-specific guidelines (e.g., product test commands, localization rules).

**Cause**: `opencode.json` at the workspace root does not include the `project/.opencode-overrides/rules/*.md` instruction glob.

**Fix**: verify `opencode.json` has both instruction paths:

```json
{
  "instructions": [
    ".opencode/rules/*.md",
    "project/.opencode-overrides/rules/*.md"
  ]
}
```

### Override files not applied

**Symptom**: agents use generic vocabulary or the default test command preference order instead of project-specific values.

**Cause**: override files are missing or misnamed.

**Fix**: verify these files exist with exact names:

- `project/.opencode-overrides/agents/backlog-planner.override.md`
- `project/.opencode-overrides/agents/change-orchestrator.override.md`

### Agent reads files from wrong path

**Symptom**: agent tries to read `backlog/README.md` instead of `project/backlog/README.md`.

**Cause**: the session-bootstrap rule or agent definition has stale paths without the `project/` prefix.

**Fix**: verify `.opencode/rules/session-bootstrap.md` references `project/` paths. Pull the latest framework if needed.

### Shell commands fail (wrong working directory)

**Symptom**: `pnpm test:unit` or similar commands fail with "no package.json found."

**Cause**: the command ran from the governance workspace root instead of `project/`.

**Fix**: the change-orchestrator should use the `workdir` parameter pointing to `project/` (or its absolute path) for all shell commands. If this keeps happening, check the override file defines test commands clearly.

### Git confusion (which repo am I committing to?)

**Symptom**: `git status` shows unexpected files, or commits go to the wrong remote.

**Fix**: remember there are two git repos:

```bash
# Governance framework commits (outer)
cd my-project-workspace/
git status  # shows .opencode/, AGENTS.md, opencode.json changes

# Project commits (inner)
cd my-project-workspace/project/
git status  # shows application code, specs, backlog changes
```

### Pulling updates causes merge conflicts

**Symptom**: `git pull` in the workspace root shows conflicts.

**Cause**: product-specific files were accidentally committed to the outer (framework) repo instead of the inner (project) repo.

**Fix**: the outer repo should only contain framework governance files. Product-specific files (architecture/, backlog/, spec/, decision-log/, product rules, overrides) must live in `project/` and be committed to the project's git repo. Check `.gitignore` excludes `project/`.
