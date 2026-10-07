# Crucible — an honest objection before the judges give you one

**PA#1 · Proposal and plan · CSC13114**
**Team:** Ngô Gia An — 23120205 · Nguyễn Bảo An — 23120207 · Trương Nhật Đạt — 23120231
**Repository:** https://github.com/NgBaoAnn/crucible

## 1. Problem and user

**User.** Minh is a third-year IT student leading a three-person team. It is five days before the idea-pitch round of a student startup competition, and he has a one-page description of his product.

**Problem.** Minh cannot get an honest objection to his idea before the judges give him one.

**What he does today.** He sends the page to two friends, who reply "sounds good, go for it". He pastes it into ChatGPT, which praises the idea and appends generic "things to consider" he can agree with without changing anything. The first real objection — "why would anyone pay for this when X is free?" — arrives on stage.

**Product.** Crucible is a web app where Minh pastes his one-page idea, receives up to five objections, each pinned to a sentence he wrote, and has to answer them. It never praises the idea and never rewrites it.

## 2. The LLM feature, and what it costs when it is wrong

**The feature: the closure ruling.** For each objection Minh types a reply. The model reads the objection, the sentence it is pinned to and the reply, and returns one ruling:

- **CLOSED** — the reply contains evidence or a changed plan that removes the objection;
- **OPEN** — the reply engages with the objection but offers no evidence;
- **DODGED** — the reply does not address the objection.

Each ruling carries a one-sentence reason, a verbatim quote of the part of the reply it relied on, and the evidence that would close it. The survival score is closed ÷ total. Generating the objections is also an LLM call, but its errors are visible and cheap — Minh dismisses an irrelevant objection in one click. The ruling is the part he trusts.

| When the ruling is wrong | Who is affected | How much | Can it be undone? |
|---|---|---|---|
| **False CLOSED** — a confident but empty reply is accepted | Minh and his two teammates | They pitch, and may spend a semester building, on a hole the tool marked as patched. Worse than no tool: it adds confidence | No — a judge asks the question once |
| **False OPEN / DODGED** — a sound reply is rejected | Minh | 10–20 minutes re-arguing, or a sound part of the plan reworked for nothing | Yes — "Appeal" re-runs the ruling with his counter-argument and logs the case |

**How we would know.** A labelled set of 60 (objection, reply) pairs, each ruled independently by two team members before the model sees it. The headline number is the false-CLOSED rate; the target is ≤ 10%, reported at every checkpoint. In the app, every appeal is stored with the prompt version and reviewed weekly.

## 3. Scope

**In this semester:** one domain (startup and product ideas); text input up to one page; objections from three fixed lenses (stingy customer, sceptical CFO, ruthless competitor) in a single call; the reply → ruling loop with survival score and appeal; sign-in and saved sessions; a plain export of closed and open objections; the labelled set and an automated eval of the ruling.

**Deliberately out:**
- Career and life decisions — a wrong ruling there can cost someone a job, and we cannot measure it this semester.
- The critical-thinking gym: case studies, fallacy radar, systems-thinking meter.
- Image or PDF upload, and pinning objections on diagrams.
- Agents debating each other in a live group chat, and a separate supervisor agent.
- An LLM-written "final plan" — Crucible judges the user's answers; it does not write them.
- Team workspaces, mobile app, payments.

## 4. Plan and ownership

| # | Checkpoint | Work | Owner | Date |
|---|---|---|---|---|
| 1 | PA#1 — proposal | This document; repository with CI; labelling rubric and a 20-pair pilot of the ruling prompt | Trương Nhật Đạt | 14/10/2026 |
| 2 | Spec — closure ruling | Verdict schema, quote check, definition of each label; labelled set grown to 60 pairs, two labellers per pair | Ngô Gia An | 28/10/2026 |
| 3 | Walking skeleton | Deployed: sign-in, paste idea, objections pinned to sentences | Nguyễn Bảo An | 11/11/2026 |
| 4 | Core feature | Reply → ruling → score end to end; eval runs in CI and prints the false-CLOSED rate | Trương Nhật Đạt | 02/12/2026 |
| 5 | Hardening and user test | Appeal flow, appeal log, per-user spend cap; five students run a real idea through it | Ngô Gia An | 23/12/2026 |
| 6 | Final | Eval report across prompt versions, export, demo and documentation | Nguyễn Bảo An | 06/01/2027 |

## 5. Risks

**1. The ruling flatters.** If the model accepts confident replies with no evidence, Crucible is ChatGPT with a score. *This week:* write 20 pairs, 8 of them confident-but-empty, and measure the false-CLOSED rate of a first prompt before any UI exists. A CLOSED ruling whose quote does not appear verbatim in the reply is downgraded to OPEN in code, not by the prompt.

**2. We cannot agree on the right answer.** If two of us label the same reply differently, the eval measures nothing. *This week:* write a one-page rubric for the three labels, label the same 20 pairs separately, and compute agreement. Below 80%, we tighten the definitions before the set grows to 60.

## 6. Technology choices

- **Next.js (App Router) + TypeScript** — one repository for the UI and the server routes that call the model, so the API key never reaches the browser.
- **PostgreSQL + Prisma** — ideas, objections, replies and rulings are relational, and every ruling is stored with its prompt version so an appeal can be replayed.
- **Auth.js with Google sign-in** — saved sessions per student and a per-user cap on spend.
- **Anthropic Claude API, `claude-opus-5-5`** ($4 input / $20 output per million tokens), structured output for the verdict schema — the ruling is the whole product, so it gets the strongest reasoning. Estimated ≈ $0.26 per session (≈ 26k input, 8k output tokens) and ≈ $2 per 60-pair eval run. `claude-haiku-4-5` ($1 / $5, ≈ $0.07 per session) replaces it only if the eval shows no rise in false CLOSED.
- **Vitest + GitHub Actions** — the labelled set runs when the prompt file changes; a higher false-CLOSED rate fails the build.
- **Vercel + Neon free tiers** — no hosting cost at class size.
