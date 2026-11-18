---
name: repo-wiki
description: Build, update or check the project wiki in docs/wiki/ so Claude can navigate a large codebase or messy folder without reading everything. Use when asked to create, refresh or audit the wiki, or when the wiki is missing or stale.
---

# Repo Wiki

Full method, templates and rules: `docs/agent/working-with-claude/WIKI_PLAYBOOK.md`. Read it before starting.

Ask which mode if unclear: **build**, **update**, or **check**.

## Build (no wiki yet, or starting over)
1. Scan structure only: directory tree (skip dependencies, build output, vendored code, data dumps), manifests, configs, entry points, README, CI, recent commits. For a documents folder, run the audit from `AGENT_SESSION_PLAYBOOK.md` Section 1.1 first.
2. Draft `docs/wiki/INDEX.md` and `docs/wiki/overview.md` with a proposed list of modules or topics and key flows. **Stop and ask the human to confirm the split.**
3. Write one page per module or topic using sub-agents in parallel, one module each, with the page template from the playbook. Map, don't copy; cite paths; record `sources` and `last_verified` (date @ short commit SHA).
4. Write flow pages for the journeys the human names as most important.
5. Verify with an independent sub-agent: every path exists, every claim matches the source. Fix or flag.
6. Present open questions and uncertain items to the human. Never invent intent, history or business rules.
7. Confirm `.claude/CLAUDE.md` points Claude to `docs/wiki/INDEX.md`.

## Update (after changes)
1. For each page, run `git log <last_verified sha>..HEAD --oneline -- <sources>`.
2. Refresh pages whose sources changed; fix dead paths; add pages for new modules; update `INDEX.md`.
3. Bump `last_verified` only on pages actually re-checked against the source.

## Check (read-only health report)
Report: stale pages (sources changed since `last_verified`), dead paths, modules or folders with no page, `INDEX.md` over ~150 lines, pages over ~150 lines. Change nothing; propose fixes.

## Rules
- Keep `INDEX.md` and each page short (under ~150 lines).
- The code and original documents are the source of truth; if the wiki disagrees, fix the wiki and tell the human.
- Don't `@`-import wiki pages into `CLAUDE.md`.
- Ask before deleting pages or restructuring the wiki.
