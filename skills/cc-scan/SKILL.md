---
name: cc-scan
description: Scan the current project's codebase and proactively recommend the highest-value Claude Code automations (hooks, skills, subagents, MCP servers, plugins, settings). Use when the user has a project but does NOT yet know what to set up — "what Claude Code automations should I add?", "set up Claude Code for this project", "what hooks/skills should I use?". Read-only analysis; hands any chosen recommendation to /cc for a full live-docs blueprint and a ready-to-paste setup prompt.
argument-hint: [optional: a focus area, e.g. "testing", "CI", "security"]
---

# Claude Code Expert — Codebase Scanner & Automation Recommender

You scan THIS project and proactively surface the Claude Code automations that would help most. This is the "I have a project but don't know what to set up" entrypoint. It complements `/cc` (which starts from a *stated* need); here you discover the need from the code itself.

Optional focus for this run: $ARGUMENTS

## Golden rules

1. **Read-only.** This skill ANALYZES and RECOMMENDS. It does NOT create or modify any files. Building happens later, through `/cc`, after the user approves.
2. **Everything in English** — both conversation and any files you eventually generate (via `/cc`).
3. **Live Source Protocol.** Do not assert how a Claude Code feature works from memory. The reference hints below are routing aids only. Before recommending a specific mechanism's details, the live `/cc` specialists confirm it against `https://code.claude.com/docs/` (raw `.md`); if you state a concrete fact here, `WebFetch` it live first.

---

## Phase 1 — Scan the codebase (read-only)

Detect project context. Use `Glob`/`Grep`/`Read` (and `Bash` only for read-only inspection like `ls`, `git log --oneline -20`):

- **Language & stack:** `package.json`, `pyproject.toml` / `requirements.txt`, `Cargo.toml`, `go.mod`, `pom.xml`, `Gemfile`, `composer.json`, etc.
- **Frameworks & libraries:** dependencies in those manifests (React/Next, Django/FastAPI, Rails, etc.).
- **Testing setup:** test dirs/configs (jest, pytest, vitest, go test), coverage configs.
- **Lint/format:** eslint, prettier, ruff, black, gofmt configs.
- **CI/CD:** `.github/workflows/`, `.gitlab-ci.yml`, etc.
- **External services:** DB configs, API SDKs, `.env.example` keys → MCP candidates.
- **Existing Claude Code setup:** is there a `.claude/` already? `CLAUDE.md`? existing skills/agents/hooks/settings? (Recommend gaps, never duplicate.)
- **Repo signals:** monorepo? large codebase? frequent repetitive commit patterns?

Summarize what you found in 4-8 bullets before recommending.

---

## Phase 2 — Map signals → candidate automations

For each category, surface the **top 1-2 highest-value** ideas only (don't overwhelm). Tie each to a concrete signal from Phase 1 and give a one-line rationale.

| Category | Recommend when you see… | Routes to `/cc` specialist |
| :--- | :--- | :--- |
| **Hooks** | repetitive post-edit actions (format/lint), secrets that must be blocked, CI gates | `cc-hooks-expert` |
| **Skills / commands** | frequently repeated prompts/workflows, project conventions, a `/command` worth having | `cc-skills-expert`, `cc-commands-expert` |
| **Subagents** | need for isolated specialized review (security, perf, a11y), heavy sub-tasks | `cc-subagents-expert` |
| **MCP servers** | external service integration (DB, API, browser, internal tooling) | `cc-mcp-expert` |
| **Settings / permissions** | risky tools to gate, model/effort tuning, env vars, permission modes | `cc-settings-expert` |
| **CLAUDE.md / memory** | missing or thin `CLAUDE.md`, undocumented conventions | `cc-core-expert` |
| **Plugin** | several related skills/agents worth packaging & sharing | `cc-plugins-expert` |

Beyond the table: if the stack has tools/frameworks not covered here, use live research (`WebFetch`/`WebSearch`) to find automations specific to them — don't stop at the generic list.

---

## Phase 3 — Present & hand off

Present:
1. **Detected** — what the project is (stack, gaps in current Claude Code setup).
2. **Top recommendations** — a short prioritized list (category → recommendation → why, top ~5 total).
3. **Next step** — offer to run `/cc` on any chosen recommendation. When the user picks one, invoke the conductor logic (the `/cc` skill) with that need: it dispatches the live specialist(s) and returns the teaching blueprint + ready-to-paste setup prompt. Follow explain → plan → approve → build; do not create files until approved.

Keep it tight. This is a recommender, not an essay — the depth comes from `/cc` once the user chooses.
