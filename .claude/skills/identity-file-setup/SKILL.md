---
name: identity-file-setup
description: Create or improve CLAUDE.md for this repo, a monorepo, or a folder containing several repos. Infers what it can from the code, interviews the human for the rest, keeps it short, and saves only after line-by-line approval. Use when asked to set up, write, fill in, improve or trim CLAUDE.md.
---

# CLAUDE.md Setup

**Principle:** Claude drafts; the human owns the result. Unreviewed AI-written context files tend to be generic, bloated or wrong, which makes the agent worse and costs tokens every session. Every line must be specific, true, and something Claude can't quickly work out from the code itself.

Background: `docs/agent/working-with-claude/AGENT_SESSION_PLAYBOOK.md` (Section 1.2) and `AGENT_OPERATIONS.md` (Section 1).

## Step 0: Pick the mode
- **New:** fill in the template at `.claude/CLAUDE.md` (placeholders in `[brackets]`).
- **Improve:** an existing CLAUDE.md: find gaps, stale facts, contradictions and bloat; propose edits.
- **Multi-repo:** a parent folder holding several repos or a monorepo (see the end of this file).

Ask if unclear. Never overwrite an existing CLAUDE.md without showing the diff and getting approval.

## Step 1: Discover (read-only, cheap)
Scan, don't read everything:
- If it exists, `docs/wiki/INDEX.md` first.
- README, package manifests (`package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, …), lockfile type (tells you the package manager), config files (lint, format, type check, test, build), `Makefile` / task runners, CI workflows, Dockerfiles, `.env.example` (variable **names** only, never values).
- Top-level directory tree (skip dependencies, build output, vendored code, data).
- Recent `git log` for commit and branch conventions.
- Existing agent instructions to merge: `CLAUDE.md`, `AGENTS.md`, `.cursorrules`, `.cursor/rules/`, `.github/copilot-instructions.md`.

## Step 2: Verify, don't assume
- Confirm each command you plan to list actually exists (in scripts, Makefile or CI). Run the read-only ones (test, lint, type check) if they're quick; report what passed or failed.
- Note anything surprising: two package managers, failing tests, conflicting configs. These belong in **Conventions & gotchas** or as questions.

## Step 3: Interview the human
Ask in small batches, only what the code can't tell you:
1. What the project does, for whom, and the current focus or milestone.
2. Their role and experience level; how much explanation they want.
3. Flows that must never break; compliance or privacy rules.
4. Domain terms Claude might misread.
5. Conventions that differ from defaults, and mistakes Claude or teammates have made before.
6. Communication preferences (length, format, tone).

Mark anything still unknown as `[CONFIRM: …]` rather than guessing.

## Step 4: Draft
Follow the existing template's section order. **Keep "How we work", "Definition of Done" and the playbook table as they are** unless the human asks to change them; only adapt items that conflict with this project.

What to include vs leave out:

| ✅ Include | ❌ Leave out |
|---|---|
| Exact commands (install, run, test one file, lint, type check, build, migrate) | Full directory listings or dependency lists (Claude can read them) |
| Where new code of each kind should go, if not obvious | Generic advice ("write clean code", "be careful") |
| Conventions that differ from tool defaults | Long explanations or tutorials (link a doc or wiki page instead) |
| Gotchas and past mistakes | Anything that changes weekly |
| Never-break flows, compliance rules, glossary | Secrets, tokens, internal URLs with credentials |
| Pointer to `docs/wiki/INDEX.md` | `@`-imports of large files (they load every session) |

Quality bar:
- **Under 200 lines; aim for 100–150.** Move detail to a wiki page or playbook and link it in plain text.
- **Specific and checkable:** "Run `pnpm test` before committing", not "test your changes".
- **No contradictions** with other CLAUDE.md files, `.claude/rules/`, or the playbooks.
- Delete the `e.g.` example lines and setup comments once real values replace them (HTML comments cost no tokens but add clutter).
- Must-follow rules for one area belong in a **nested** `CLAUDE.md` in that folder or in `.claude/rules/` with `paths:` frontmatter, not the root file.

## Step 5: Review with the human
1. Show the full draft (or a diff in improve mode), with `[CONFIRM]` items listed at the top.
2. Ask the human to read it line by line and correct anything wrong, vague or not in their voice.
3. Apply their edits. **Save only after explicit approval.**

## Step 6: Verify it loads
- Ask the human to run `/context` in a new session and check `CLAUDE.md` is listed under **Memory files**.
- Report the final line count and roughly how many tokens it adds per session.

## Improve mode: extra checks
- Facts that no longer match the code (renamed scripts, removed folders, changed stack).
- Rules the human keeps repeating in chat that should be added.
- Lines Claude could work out from the code: propose removal.
- Duplicates and contradictions across all loaded instruction files.
- Over 200 lines: propose what to move to the wiki, a playbook, a nested CLAUDE.md or a skill.

Claude Code also has built-in help (`/init` generates a starter file; recent versions of `/doctor` can propose trims). Use them alongside this skill if available, then apply the quality bar above.

## Multi-repo or monorepo
Claude Code loads `CLAUDE.md` from the folder it's launched in **and every parent folder** at startup; `CLAUDE.md` files in subfolders load only when Claude works with files there. Check the current docs if behaviour seems different.

```
workspace/
├── CLAUDE.md               ← workspace level: what each repo is, how they connect, shared conventions
├── docs/agent/             ← shared playbooks (one copy)
├── docs/wiki/INDEX.md      ← optional cross-repo map
├── repo-a/
│   └── CLAUDE.md           ← repo-specific: stack, commands, layout, gotchas
└── repo-b/
    └── CLAUDE.md
```

1. Discover each repo separately (Steps 1–2), ideally with one sub-agent per repo so the main conversation stays small.
2. **Workspace CLAUDE.md:** a short map of the repos (one line each: purpose, stack, owner), how they talk to each other (APIs, shared packages, events), shared conventions, and the shared "How we work" section. Keep it short; it loads with every repo.
3. **Per-repo CLAUDE.md:** only what's specific to that repo; don't repeat the workspace file.
4. Playbook paths: if the playbooks live once at the workspace root, make the paths in each CLAUDE.md point there (e.g. `../docs/agent/…`), or copy the setup into each repo that's opened on its own.
5. Review each file with the human (Step 5), workspace file first.
