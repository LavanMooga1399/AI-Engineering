# Agent Playbooks

Reference guides for Claude Code in this repo. **`.claude/CLAUDE.md` is loaded every session; these files are opened only when a task needs them**, so they cost no tokens otherwise. The table at the end of `.claude/CLAUDE.md` tells Claude which file to open for which task.

```
.claude/
├── CLAUDE.md                          ← identity file, loaded every session
└── skills/
    ├── identity-file-setup/SKILL.md   /identity-file-setup: draft or improve CLAUDE.md (one or many repos)
    └── repo-wiki/SKILL.md             /repo-wiki: build, update or check docs/wiki/
docs/agent/
├── README.md                          ← this index
├── working-with-claude/               How Claude works in this repo
│   ├── WORKFLOW.md                    Spec → build → walkthrough → test; templates & diagrams
│   ├── AGENT_OPERATIONS.md            Identity, tool permissions, skills, memory, trust levels
│   ├── AGENT_SESSION_PLAYBOOK.md      Token-efficient sessions: audit, caching, handoffs, recovery
│   └── WIKI_PLAYBOOK.md               Navigable wiki for a large codebase or messy folder
├── ai-system-design/                  Designing AI features and agents
│   ├── AI_SYSTEM_DESIGN.md            Building blocks: model, tools, knowledge, memory, evals, ops
│   ├── AI_SYSTEM_DESIGN_PROCESS.md    Five-layer design process + design-doc template
│   ├── AI_ENGINEERING_LAYERS.md       Prompt → context → harness → loop → graph
│   ├── HARNESS_ENGINEERING_PLAYBOOK.md  Build a harness: context, actions, boundaries, persistence
│   └── AUTOMATION_PLAYBOOK.md         What to automate, trust ladder, ownership, one at a time
├── ai-building-blocks/                Specific techniques
│   ├── RAG_PLAYBOOK.md                Retrieval methods, agentic retrieval, knowledge runtime
│   ├── MCP_PLAYBOOK.md                MCP architecture, 2026 spec changes, using & building servers
│   ├── DECISION_MODELS_PLAYBOOK.md    Bounded decisions, decision models, hybrid architecture
│   └── FINE_TUNING_PLAYBOOK.md        When and how to fine-tune, LoRA, RL, distillation
└── quality-and-safety/                Proving it works and keeping it safe
    ├── EVALS_PLAYBOOK.md              Reference datasets, error analysis, rubrics, LLM judges
    └── AI_SECURITY_PLAYBOOK.md        Information, instructions, actions + security testing
```

## Setup

1. Copy `.claude/` and `docs/` into your repo root. (`.claude` is a hidden folder; show hidden files to see it.)
2. Run `/identity-file-setup` to fill in `.claude/CLAUDE.md` with Claude (it interviews you and saves only after your review). For a large codebase, run step 4 first so the wiki can feed into it.
3. In a Claude Code session, run `/context` and check that `CLAUDE.md` is listed under **Memory files**.
4. For a large codebase or messy folder, run `/repo-wiki` (build mode) once, review the pages it drafts, and commit `docs/wiki/`.

## Folders Claude creates as needed

These are referenced by the playbooks and created the first time they're used:

| Path | Holds |
|---|---|
| `docs/wiki/` | Map of the codebase/folder (built by `/repo-wiki`) |
| `docs/specs/` | Feature specs and design docs |
| `progress.md` | Handoff file for multi-session work |
| `templates/` | Reusable output templates (emails, docs) |
| `evals/` | Eval datasets, judges, runner, results |
| `docs/automations/REGISTRY.md` | Registry of live automations |
| `.claude/skills/<name>/SKILL.md` | Repeatable workflows as skills |
| `.mcp.json` | Project MCP server config (no secrets) |

## Maintaining these files

- Keep `.claude/CLAUDE.md` under ~200 lines; put detail here instead.
- When adding a playbook, put it in the matching folder and add a row to the table in `.claude/CLAUDE.md`.
- Use relative links between playbooks (e.g. `../ai-building-blocks/RAG_PLAYBOOK.md`).
