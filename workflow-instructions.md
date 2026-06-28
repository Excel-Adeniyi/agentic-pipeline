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
- **What this is:** [1-2 sentence description of the project's purpose.]
- **Detected stack(s):** [e.g. "Node.js/Express API, Flutter mobile app" — fill in from step 0.]
- **Repo layout:**
  ```
  /[folder]    -> [what lives here, which stack]
  /[folder]    -> [what lives here, which stack]
  ```

## 2. Environment Setup
- Runtime version(s): [e.g. Node 20.x via nvm, Python 3.12 via venv, Flutter 3.x via FVM]
- Package manager: [npm / yarn / pnpm / pip / poetry / pub / etc.] — don't mix lockfiles
- Required env vars: see `[.env.example or equivalent]`. Never commit secrets.

## 3. Per-Stack Commands
> Duplicate this block for each stack/service detected in step 0 (e.g. one
> for the backend, one for the mobile app, one for a separate worker service).

### [Stack/Service name, e.g. "Backend (Node.js)"]
- Install: `[command]`
- Run (dev): `[command]`
- Run tests: `[command]`
- Lint / format: `[command]`
- Type check: `[command, if applicable]`
- Build: `[command]`

**Conventions:** [project-specific patterns to follow, e.g. folder conventions, state management approach, validation library, naming rules.]

### [Repeat for next stack/service…]

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

## 8. Documenting Fixes
- Every fix must be documented as a markdown file inside `docs/`.
- One file per fix, tied to a single, unique issue (e.g. `docs/issue-123-login-timeout.md` or `docs/<issue-id>-<short-slug>.md`).
- Each file must stay under 500 lines. If a fix needs more than that, split it
  into multiple linked files rather than writing one giant doc.
- Do not combine multiple unrelated fixes into the same file.