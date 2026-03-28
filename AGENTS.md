# AI Development Governance

## Workspace Convention

This is a **governance workspace**. The project code and all product-specific governance live in `project/`. All agent file operations on code, specs, backlog, architecture, and decision-log target paths under `project/`.

The governance framework (this repo's root-level files) provides the methodology. The project provides the product-specific context.

## Taxonomy

- **Rules** (`.opencode/rules/`): durable constraints that are always true. Agents apply them without being asked.
- **Commands** (`.opencode/commands/`): user-triggered actions (slash commands).
- **Skills** (`.opencode/skills/`): methodology an agent loads on demand for a specific kind of work.
- **Agents** (`.opencode/agents/`): distinct personas with their own system prompts, used for delegation.
- **MCP servers** (`opencode.json` > `mcp`): external tool integrations agents can call.
- **Hooks** (`.opencode/hooks/`): CI-style checks that run automatically before or after agent actions.
- **Memory** (`.opencode/memory/`): per-session context that persists across compactions.

## Primary UX -- Two Agents

| Agent               | File                                      | Purpose                                                                              |
| ------------------- | ----------------------------------------- | ------------------------------------------------------------------------------------ |
| Backlog-Planner     | `.opencode/agents/backlog-planner.md`     | Planning: create, update, reprioritize backlog items in `project/backlog/proposed/`. |
| Change-Orchestrator | `.opencode/agents/change-orchestrator.md` | Implementation: pick up items, implement, test, update specs in `project/`.          |

Workflow: **plan** (Backlog-Planner) -> **human approves** -> **implement** (Change-Orchestrator).

## Source of Truth

All product-specific source-of-truth directories live under `project/`:

| Directory               | Contains                                                           |
| ----------------------- | ------------------------------------------------------------------ |
| `project/backlog/`      | Work items: proposed, in-progress, done.                           |
| `project/spec/`         | Product behavior specifications (narrative + testable assertions). |
| `project/decision-log/` | Architectural and library decisions with rationale.                |
| `project/architecture/` | Codebase map, domain glossary, system overview.                    |
| `project/README.md`     | Developer startup guide.                                           |

## Governance Skills

- **opencode-governance-authoring**: load when creating or updating any governance artifact (rules, commands, skills, agents, MCP config). Guides correct taxonomy placement and review.
- **continuous-learning**: load when repeated user guidance or friction signals a reusable governance improvement.

## Portability Rules

- Do not rely on `.opencode/memory/` for cross-session correctness. The backlog item file is the durable record.
- Do not assume any host-level configuration. Everything an agent needs is in this repo or in `opencode.json`.
- MCP server configuration lives in `opencode.json`, not in agent prompts.
- When a change touches behavior, update the relevant `project/spec/` file, tests, and architectural docs together.

## Product-Specific Overrides

Projects can provide product-specific agent overrides at:

- `project/.opencode-overrides/agents/backlog-planner.override.md` -- vocabulary, personas, themes.
- `project/.opencode-overrides/agents/change-orchestrator.override.md` -- test commands, validation steps.
- `project/.opencode-overrides/rules/*.md` -- product-specific rules loaded alongside framework rules.

These files live outside `.opencode/` so OpenCode does not auto-discover them as standalone agents or rules. Framework agents read them explicitly at session start via their Override loading sections.
