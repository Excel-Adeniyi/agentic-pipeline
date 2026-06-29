# Project Agent Instructions

> Place this file at the repo root as `workflow-instructions.md`. It's written to work for any
> project/stack — the agent detects what's actually there and fills sections in.
> Keep entries short and concrete — agents follow specific commands far better than prose.

## 0. Before You Start (always do this first)
1. **Inspect the repo** — manifest/config files (`package.json`, `pubspec.yaml`,
   `pyproject.toml`, `go.mod`, `Cargo.toml`, `composer.json`, `*.csproj`, etc.),
   lockfiles, Dockerfiles/`docker-compose.yml`, CI config, and folder structure.
2. **Summarize what you find**: detected stack(s) and versions, package
   manager, key scripts/commands (install, run, test, lint, build), repo
   layout, how each part is run (local vs Docker/containers), and any env vars
   required.
3. **Confirm with the user** that your summary matches reality and matches
   what's written in this file, before changing anything or filling in
   placeholders below.
4. **If anything is ambiguous or undetectable** (unclear which script is
   canonical, missing env example, multiple compose files, conflicting
   configs) — **do not guess**. Ask the user explicitly for that detail.
5. Only after confirmation, update the `[bracketed]` placeholders in the
   sections below to match the real project. Repeat this check if the stack
   changes significantly later (new service added, major dependency swap, etc.).

## 1. Project Overview
- **What this is:** [1-2 sentence description of the product's purpose.]
- **Stack:** Node.js application (single stack — confirm framework, e.g. Express/Fastify/Nest, during Step 0).
- **Spec source of truth:** numbered spec files `#1`–`#15` at the **root of the repo**.
  These define the product behavior. Code and docs must stay consistent with them; if a
  spec file and the codebase disagree, flag it rather than picking one silently.
- **Repo layout:**
  ```
  /#1 ... #15  -> spec files (root)
  /[folder]    -> [what lives here]
  /[folder]    -> [what lives here]
  ```

## 2. Environment Setup
- Runtime version(s): [e.g. Node 20.x via nvm, Python 3.12 via venv, Flutter 3.x via FVM]
- Package manager: [npm / yarn / pnpm / pip / poetry / pub / etc.] — don't mix lockfiles
- Required env vars: see `[.env.example or equivalent]`. Never commit secrets.

## 3. Commands (Node.js)
- Install: `[npm install / yarn / pnpm install]`
- Run (dev): `[command]`
- Run tests: `[command]`
- Lint / format: `[command]`
- Type check: `[command, if TypeScript]`
- Build: `[command]`

**Conventions:** [project-specific patterns to follow, e.g. folder conventions, validation library, error-handling pattern, naming rules.]

## 4. Running via Docker (if applicable)
- Build image: `docker build -t [name] [path]`
- Run via Compose: `docker compose up` (`-d` to detach, `--build` to force rebuild)
- Compose file(s): `[location(s) — flag if there's more than one and which is canonical]`
- Services defined: `[list]`
- Logs: `docker compose logs -f [service]`
- Run one-off command in container: `docker compose exec [service] [command]`
- Stop/remove: `docker compose down` (`-v` only with explicit confirmation — wipes volumes/data)
- Env vars: `[where they're sourced from]` — never hardcode secrets into Dockerfiles/compose files.

## 5. Cross-Cutting Conventions
- Branching: [e.g. "feature/*, fix/*, never push directly to main."]
- Commit messages: [e.g. Conventional Commits]
- Contract/shared types: [where source of truth lives, if multiple stacks talk to each other]
- Before opening a PR: run lint + tests for whichever part(s) you touched.

## 6. Guardrails — Things the agent should NOT do
- Do not run destructive commands (`rm -rf`, force-push, dropping DB tables,
  `docker compose down -v`, `docker system prune`) without explicit confirmation.
- Do not modify generated/auto-generated files directly — regenerate them instead.
- Do not add new dependencies without checking if an existing one already covers the need.
- Do not commit `.env`, keys, or credentials.
- Do not change CI/CD config, Dockerfiles, or deployment scripts without flagging it explicitly.
- Do not assume undocumented behavior — if step 0 left something unclear, ask rather than guess.

## 7. Useful Context
- [Ports, base URLs, how services connect to each other locally.]
- [Any quirky setup steps specific to this project.]
- [Known flaky tests or areas of tech debt worth knowing about.]

## 8. Documenting Fixes
- Every fix must be documented in a markdown file inside `docs/`.
- One file per issue — each file maps to a single, unique issue (no bundling multiple unrelated fixes into one doc).
- File naming: `[e.g. docs/<issue-id-or-short-slug>.md, such as docs/fix-login-timeout.md]`
- Max length: **500 lines per file**. If a fix needs more than that, split it into multiple linked files rather than exceeding the limit.
- Suggested contents per file: issue description, root cause, fix applied, files changed, how it was verified/tested.
- Before creating a new doc, check `docs/` for an existing file tied to the same issue and update it instead of creating a duplicate.

## 9. Multi-Agent Roles: Codex (implementer) + Claude (evaluator & edge cases)

This project uses two agents with distinct, non-overlapping responsibilities. Codex
does primary implementation; Claude evaluates that work against this file and the
specs, and separately owns edge cases. Each role is scoped below so it's clear who
does what — this isn't a suggestion, it's the division of labor for this repo.

### Codex — Primary Implementer
- Implements features/fixes per the numbered spec files (`#1`–`#15`) and this file's
  conventions (sections 1–6).
- Must follow Section 6 guardrails exactly — no exceptions without explicit user confirmation.
- Every change must produce a docs/ entry per Section 8.
- Codex should flag (not silently resolve) anything where a spec file is ambiguous,
  incomplete, or conflicts with another spec file or the existing codebase.

### Claude — Evaluator
When evaluating, Claude operates as a **senior backend Node.js engineer reviewing this
codebase** — not a generic linter pass. After Codex produces a change (or when asked to
review the codebase as a whole), Claude:

1. **Reviews** the code/change against:
   - **Spec adherence** — does it match the relevant numbered spec file(s) `#1`–`#15`
     at the repo root? Cite which spec(s) apply and note any deviation explicitly.
   - **Guardrail compliance** — anything violating Section 6 (destructive commands run
     without confirmation, generated files edited directly, secrets committed,
     unflagged CI/Docker changes, undocumented assumptions).
   - **Convention/quality compliance** — Section 3/5 conventions, idiomatic Node.js
     practices (error handling, async patterns, input validation, security basics like
     injection/auth checks), test coverage for the change.
   - **Documentation compliance** — a corresponding `docs/` file per Section 8, scoped
     to one issue, under 500 lines.
2. **Produces an evaluation document** in `docs/` (per Section 8's naming/length rules)
   summarizing: what was reviewed, pass/fail per criterion above with specifics
   (file/line), and overall verdict.
3. **Fixes issues found** — if something isn't properly done, Claude fixes it directly
   rather than just flagging it, then notes in the evaluation doc what was fixed and why.
   Exception: anything touching the Section 6 guardrails (destructive ops, schema
   changes, auth, CI/Docker config) — surface those for explicit confirmation before
   fixing, don't auto-fix silently.

### Claude — Edge Cases
Claude implements directly (rather than evaluating Codex's attempt) when a task is:
- Not clearly covered by any of the `#1`–`#15` spec files
- Cross-cutting across multiple stacks/services in a way that's easy to get subtly wrong
- Explicitly flagged by the user as an edge case to hand to Claude

When Claude implements an edge case, it still follows Sections 1–6 and 8 like any
other change — the only difference is which agent is doing the work, not a relaxed
standard.

### Handoff notes
- [Where does Claude see Codex's output — PR diff, branch, shared file? Fill in once decided.]
- [Any cases where Codex should NOT proceed without Claude's sign-off first — e.g. schema/migration changes, auth logic — fill in if applicable.]
