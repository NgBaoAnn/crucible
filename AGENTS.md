# AGENTS.md

Planning + implementation workspace for **The Adversarial Crucible** (anti-sycophancy multi-agent sparring platform), Sea x OpenAI Regional Codex Hackathon – Vietnam 2026.

## Repo state (important)

- **No code exists yet.** There is no `package.json`, no workspace config, and **no git repo initialized** (`git init` still pending) — yet the plan instructs committing after each task. Initialize git and the monorepo before the first task, or skip commit steps until then.
- This is a docs-first repo; treat the plan as the source of truth, not the strategy prose (they can drift).

## Doc map

| File | Role |
|---|---|
| `docs/superpowers/plans/2026-09-24-crucible-v0-mvp.md` | **Authoritative build plan** — monorepo layout, per-task checklists, exact code snippets, test commands, 24h schedule. Execute this. |
| `hackathon_strategy_and_winning_ideas.md` | Product strategy/spec (Vietnamese). Cited by the plan as the spec. |
| `hackathon_submission.md` | English submission-form answers (Q1–Q4). |
| `text.txt` | Byte-identical copy of `hackathon_submission.md`. **Edit the `.md`, not `text.txt`**; treat `text.txt` as a throwaway export. |
| `hackathon_strategy_and_winning_ideas.docx` | Word export of the strategy doc; regenerate if the `.md` changes. |

## Language

The strategy doc and the build plan are authored in **Vietnamese**; the submission form is English. Keep existing docs in their language — do not translate or rewrite them. New working docs may be in either.

## Executing the plan (non-obvious contract)

- The plan header mandates using checkbox-tracking task execution (superpowers subagent-driven-development / executing-plans workflow): pick a task, implement, **tick its `- [ ]` boxes**, run that task's test command, then commit.
- Planned monorepo (from the plan): Turborepo + pnpm workspaces (Bun acceptable), packages `@crucible/core`, `@crucible/server`, `@crucible/app` linked via `workspace:*`, TypeScript everywhere.
- Planned commands (only exist once the monorepo is scaffolded): `pnpm --filter @crucible/core test`, `pnpm --filter @crucible/server test:tool`, `pnpm --filter @crucible/app build`.
- Architecture rules from the plan:
  - All Zod schemas and event types live in `packages/core` and are imported by server/app — never duplicate types.
  - Each module dir in `packages/server/src/` (`agent/`, `bus/`, `session/`, `tool/`, `provider/`, `server/`) exposes a single `index.ts`; **no deep imports** across module boundaries.
  - Streaming is SSE via the in-memory Event Bus; `tool/codex-runner.ts` executes generated Python in a subprocess with a **3s timeout / SIGKILL** — tests cover infinite loops and must not hang.
  - App UI: Next.js 14+, Tailwind, `@xyflow/react` for the Hypothesis Graph; survival score must stay integer in `[0, 100]`.