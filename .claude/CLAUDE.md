<!--
=====================================================================
HOW TO USE THIS FILE  (for you, the human)
HTML comments like this one are stripped before Claude reads the file,
so these notes cost zero tokens. Keep them, or delete them once done.

1. Location: .claude/CLAUDE.md in your repo. Claude Code (desktop app
   and VS Code extension) loads it at the start of every session.
2. Fill the [PLACEHOLDERS] with Claude: run the /identity-file-setup
   skill. It scans the repo (or several repos), fills what it can
   infer, interviews you for the rest, and saves only after you've
   reviewed every line. Without the skill, use this prompt:

   "Let's fill in .claude/CLAUDE.md together. First read the repo:
    README, package/config files, folder structure, test setup and
    recent commits. Fill in everything you can infer, then interview
    me one small batch of questions at a time for the rest. Keep each
    entry short and specific, no generic filler. Show me the result
    before saving."

3. Delete the "e.g." example lines once you've replaced them.
4. Keep the whole file under ~200 lines; detail belongs in docs/agent/.
5. Check it loaded: run /context and look under "Memory files".
6. This file is guidance, not enforcement. For hard limits (never
   touch .env, never push to main) also add permissions.deny rules
   in .claude/settings.json, or a hook.
=====================================================================
-->

# [PROJECT NAME]

## About this project
- **What it does:** [2–3 sentences: the product, who it's for, the problem it solves]
  <!-- e.g. "Acme Analytics is a B2B SaaS dashboard that shows e-commerce brands where their revenue comes from. Customers connect Shopify and ad accounts; we combine the data into daily reports." -->
- **Users:** [who uses it and what they care about most]
  <!-- e.g. "Growth managers at 10–200 person e-commerce brands. They care about accurate numbers and fast dashboards." -->
- **Current focus:** [the milestone or feature being worked on now]
  <!-- e.g. "Launching the Slack weekly digest by end of November." -->
- **Me:** [your role, experience level, and how much explanation you want]
  <!-- e.g. "Solo founder-developer. Strong in backend, newer to frontend: explain frontend choices in more detail." -->
- **People:** [names Claude should know and what they own, or "solo project"]

## Tech stack & commands
- **Stack:** [language, framework, database, hosting]
  <!-- e.g. "TypeScript, Next.js 15 (App Router), PostgreSQL via Prisma, deployed on Vercel" -->
- **Package manager:** [e.g. pnpm, never npm or yarn]
- **Install:** `[e.g. pnpm install]`
- **Run locally:** `[e.g. pnpm dev]`
- **Test:** `[e.g. pnpm test]` · single file: `[e.g. pnpm test path/to/file]`
- **Lint / format / types:** `[e.g. pnpm lint && pnpm typecheck]`
- **Build:** `[e.g. pnpm build]`
- **DB migrations:** `[e.g. pnpm prisma migrate dev]`. Always ask before running.

## Project map
<!-- Only what Claude can't quickly work out from the folder names. -->
- `[e.g. src/app/]` – [e.g. pages and API routes]
- `[e.g. src/lib/]` – [e.g. shared business logic; put new logic here, not in routes]
- `[e.g. src/integrations/]` – [e.g. one folder per external API]
- `docs/wiki/` – map of this codebase/folder; **start at `docs/wiki/INDEX.md`**
- `docs/specs/` – feature specs
- `docs/agent/` – detailed playbooks in four topic folders (see the table at the end)

## Conventions
<!-- Things that differ from defaults, or that Claude got wrong before. -->
- [e.g. Validate every API input with zod schemas in src/lib/schemas/]
- [e.g. Use the logger in src/lib/log.ts, never console.log]
- [e.g. Branch names: feat/<short-name>; conventional commit messages]
- [e.g. Money is stored as integer cents]

## Domain rules & critical flows
- **Never break:** [e.g. login, billing, data sync]
- **Compliance / privacy:** [e.g. never log customer emails or tokens; data stays in EU region]
- **Glossary:** [domain terms Claude might misread]
  <!-- e.g. "Workspace = a paying customer account. Source = a connected integration." -->

---

## How we work

### Workflow for any non-trivial task
One-line changes skip steps 2 and 4.
1. **Understand:** read the relevant code first. Reuse existing patterns and abstractions.
2. **Spec:** before writing code, draft a spec with Goal, Inputs, Outputs, Edge cases (at least 3; brainstorm "what usually breaks"), Definition of Done, and Constraints. Use plan mode where available. Wait for my confirmation, then save it to `docs/specs/<feature>.md`. Full template: `docs/agent/working-with-claude/WORKFLOW.md`.
3. **Build:** exactly what the spec says, within its constraints. Small, reviewable changes; no unrelated edits.
4. **Explain:** walk me through what you built at three layers: architecture (structure, data flow, new abstractions), constructs (patterns, shortcuts), and logic (every assumption, how unexpected input is handled).
5. **Test:** against the spec's expected behaviour, including failure paths. Run the tests and report results honestly.
6. **Done:** check the Definition of Done below.

### Engineering standards
- Simplest correct solution; don't add abstractions the codebase doesn't need.
- Config in env vars, never hard-coded. No secrets in code, logs or commits.
- Validate inputs, set timeouts on external calls, handle partial failures. No silent failures.
- If a request could reasonably be built several ways, ask instead of picking one silently.

### Permissions
- **Go ahead:** read and search code, run tests, linters and type checks, edit files within the task's scope.
- **Ask first:** add or upgrade dependencies, run migrations or anything that changes data, `git commit` / `push` / PRs, edit config or CI or out-of-scope files, call external services that write or send, delete anything.
- **Never:** touch production systems or data; read, print or commit secrets.

### Communication
- [e.g. Be concise: bullet points, no long preambles]
- [e.g. No emojis]
- Cite `file:line` for claims about the code; say when you're relying on memory rather than docs.
- For large or vague tasks, interview me in small batches before building.
- If the session is going badly (repeated mistakes, forgotten decisions), say so and offer a `progress.md` handoff.

### Context & tokens
- **Orient via the wiki:** read `docs/wiki/INDEX.md` first, then only the 1–3 pages the task needs, then only the files they point to. No wiki yet in a large repo? Suggest `/repo-wiki`.
- **Load on demand:** open a playbook only when the table below says it applies; never read whole folders "for context".
- Prefer targeted search (grep, file listing) over opening many files; summarise large outputs instead of pasting them.
- One task per session; suggest a fresh session when the topic changes. Read `progress.md` first if it exists.
- The wiki is a map, not the truth: verify against the source before editing.

## Definition of Done
- [ ] Spec exists and the implementation matches it
- [ ] Every edge case handled and tested
- [ ] No constraint violated
- [ ] Three-layer walkthrough given, assumptions listed
- [ ] Tests (and lint/type checks) run and passing
- [ ] AI features: evals pass. Automations: owner, tripwires and logging in place
- [ ] Spec updated if reality differed
- [ ] Wiki pages updated if structure or behaviour changed

## Playbooks: read only when the task needs them
All under `docs/agent/` (index: `docs/agent/README.md`). Several may apply to one task.

| When the task… | Read |
|---|---|
| **Working with Claude** | `working-with-claude/` |
| Needs a spec template, review checklist or diagrams | `WORKFLOW.md` |
| Involves permissions, skills, memory or trust levels | `AGENT_OPERATIONS.md` |
| Is long or multi-session, relies on a docs folder, or is going badly | `AGENT_SESSION_PLAYBOOK.md` (read `progress.md` first if it exists) |
| Builds, updates or checks the wiki, or is in a large repo with no wiki | `WIKI_PLAYBOOK.md` (or run the `/repo-wiki` skill) |
| Creates, improves or trims this CLAUDE.md (one repo or several) | run the `/identity-file-setup` skill |
| **AI system design** | `ai-system-design/` |
| Adds or changes an LLM or AI feature | `AI_SYSTEM_DESIGN.md` |
| Designs a new AI feature/agent end to end, scales one to production, or is interview practice | `AI_SYSTEM_DESIGN_PROCESS.md` |
| Designs an agent, loop, or multi-step / multi-agent workflow | `AI_ENGINEERING_LAYERS.md` |
| Builds an internal assistant/harness, or adds context, tools, permissions or persistence to an agent | `HARNESS_ENGINEERING_PLAYBOOK.md` |
| Builds or changes an automation, bot or scheduled job | `AUTOMATION_PLAYBOOK.md` |
| **AI building blocks** | `ai-building-blocks/` |
| Gives an LLM documents or data (RAG, search, Q&A) | `RAG_PLAYBOOK.md` |
| Connects to external tools/data, or adds, configures or builds an MCP server | `MCP_PLAYBOOK.md` |
| Routes, classifies or scores at volume, or aims to cut AI cost/latency | `DECISION_MODELS_PLAYBOOK.md` |
| Considers fine-tuning, LoRA, RL or distillation, or a model keeps failing despite good prompts/context | `FINE_TUNING_PLAYBOOK.md` |
| **Quality & safety** | `quality-and-safety/` |
| Builds, runs or debugs evals, or asks if an AI feature is "ready" | `EVALS_PLAYBOOK.md` |
| AI feature touches private data, untrusted content, or write/send tools; or goes to production | `AI_SECURITY_PLAYBOOK.md` |

## Learned preferences
<!-- When you say "remember…", Claude Code saves it to its own auto memory (personal, on your machine). For rules that should live in the repo, say "add this to CLAUDE.md" and Claude adds one line here. -->
- [e.g. Use date-fns, not moment]
