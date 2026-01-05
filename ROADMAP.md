# Roadmap — Team Reminder Service

_Last updated: 2026-01-05_

## Purpose — why a roadmap now ✅
The repository is in active use but has grown organically (see `README.md`, `docs/contributing.md`, and `docs/issues_snapshot.md`). Symptoms: inconsistent information architecture, mixed storage implementations (in-memory vs DB), uneven API/error handling, and no agreed collaboration workflow. These cause duplicated effort, risky parallel edits, and slow, ad-hoc fixes.

A focused, short roadmap will:
- Reduce blocking and merge conflicts by enabling parallel, low-risk workstreams.
- Prioritize low-effort, high-impact fixes (stability, contributor experience).
- Create clear decision points for risky/platform-level choices (DB, background worker, API versioning).

> This is a living, low-friction plan for a small contributor pool — not a feature backlog.

---

## Principles (constraints) ✨
- Small team / limited time: prefer incremental, reversible changes and short spikes (1–2 week).
- Minimize user disruption: prefer non-breaking migrations, feature flags, and compatibility shims.
- Ship governance early (labels, templates, CI) so contributors can work safely in parallel.

---

## Phases & milestones (3) — 8–12 week horizon
Each phase is 2–4 weeks with clear acceptance criteria and small, parallel workstreams.

### Phase 1 — Stabilize & govern (Weeks 0–3) — goal: stop the rot
Primary outcomes:
- Clear doc IA and contribution rules so contributors stop duplicating pages.
- Fast, automated safety checks to catch style/typing/docs regressions.
- Small, visible quality wins to lower contributor friction.

Key deliverables (example issues):
- Content audit + canonical IA (map existing pages under `docs/`) and top-level nav in `README.md`.
- Add `ROADMAP.md`, `ISSUE_TEMPLATE.md`, `PULL_REQUEST_TEMPLATE.md`, and suggested labels (`area:docs`, `area:infra`, `area:migration`).
- CI: markdown lint + docs build + basic unit/test smoke (fail fast on PRs).
- Quick wins: unify CONTRIBUTING.md with explicit next steps for docs edits and ownership recommendations.

Success metrics:
- <72h review SLA for doc PRs; new PRs must include a template.
- CI prevents merge if docs build or lint fails.

Why first: governance and automated checks unblock parallel work and reduce churn.

---

### Phase 2 — Developer experience & core correctness (Weeks 3–8) — goal: reduce technical debt
Primary outcomes:
- Shared backend patterns so features can be built safely and reviewed quickly.
- Low-risk infra choices framed as experiments (spikes) rather than big rewrites.

Key deliverables:
- Shared error-handling layer + one migration PR that standardizes responses for a small set of endpoints (`users` or `notifications`).
- Repository interface hardening: add interface tests for `repositories.py` and a migration plan (spike) for persistent storage (SQLite PoC → Postgres later).
- Add basic observability: structured logging and a lightweight metrics endpoint; add tests for logging behavior.
- Introduce a simple scheduler PoC (in-process cron or lightweight worker) and acceptance tests for scheduled reminders.
- Expand test coverage for critical endpoints (target: +20% coverage on core paths).

Parallelism notes:
- Docs/content cleanup (Phase 1) can continue in parallel with these engineering efforts.
- Storage spike and error-handling may run concurrently but agree on an interface contract first (API and repository interfaces).

Acceptance criteria:
- One endpoint fully migrated to the shared error model and covered by tests.
- A documented storage migration spike (with sample migration code and README) in `docs/`.

---

### Phase 3 — Incremental polish & capability rollouts (Weeks 8–12+) — goal: enable scale
Primary outcomes:
- Lightweight advanced features that improve contributor and consumer experience without large maintenance burden.

Candidate deliverables (prioritized):
- Search/indexing for docs (simple full-text or Algolia free-tier PoC).
- Tagging and improved metadata for docs to improve discoverability.
- Define API versioning policy and apply to any breaking changes (minor semantic version + changelog practice).
- Basic role/permission model for critical write operations (opt for simple token-based gating first).
- Improve observability (metrics -> Prometheus format + dashboards as a stretch goal).

Gate for wider rollout: successful PoCs, documented migration paths, and CI + tests in place.

---

## Collaboration & parallel-work strategy 🔀
Who works on what (small-team, concurrent-safe):
- Docs maintainers (content + IA): canonicalize pages, add templates, and implement search metadata.
- Backend owners: error model, repo interface tests, and the storage spike.
- Infra/CI: add docs build, linters, and basic observability artifacts to CI.
- PM/maintainer: triage, reconcile cross-cutting decisions, and publish milestone notes.

How to avoid blocking:
1. Ship governance (Phase 1) immediately so contributors use the same templates and labels.
2. Use well-scoped spikes for risky choices (DB, queue). Spikes produce a recommendation (+ pros/cons) and a small PoC branch.
3. Enforce a compatibility contract for `repositories.py` — anyone changing storage implements the interface and adds tests.
4. Prefer backward-compatible small PRs (feature-flagged where applicable).

Suggested labels and rules:
- `area:docs`, `area:api`, `area:infra`, `area:migration`, `help-wanted`, `good-first-issue`, `priority:high`
- Require at least one `area:` label and an ISSUE for any >500-line or cross-cutting PR.

Review cadence:
- Weekly triage for new issues and PR grooming.
- End-of-milestone demo + 1-page retro.

---

## Key uncertainties & decision points ⚠️
- Storage backend: keep in-memory (temporary) → PoC SQLite → Postgres for production. Decision needs data volume and retention input.
- Background processing: lightweight cron vs. task queue (RQ/Celery). Choose cron or small worker first to reduce maintenance.
- API versioning: do we need v1 endpoints now? If consumers are few and internal, defer strict versioning — but require changelog and deprecation policy.

Recommended de-risking steps:
- Two-week spike for storage (implement SQLite-backed `repositories` with migration README).
- Spike scheduler options with a single acceptance test that simulates scheduled delivery.

---

## Out of scope / Not now 🚫
- Full multi-tenant architecture — high cost and unclear ROI for current users.
- Large UX/front-end rewrite or migration to a new frontend framework.
- Heavy-weight observability (distributed tracing, APM) until core reliability and CI are in place.

Rationale: these items are high effort, carry long maintenance costs, or depend on decisions we haven’t made yet.

---

## Quick 2‑week starter sprint (concrete, assignable) ⏱️
1. Content: Create `docs/IA.md` — map current pages and propose canonical nav (owner: docs). (1–2d)
2. CI & governance: Add `ISSUE_TEMPLATE.md`, `PULL_REQUEST_TEMPLATE.md`, labels, and a markdown-lint + docs-build CI job (owner: infra). (2–3d)
3. API hygiene spike: implement shared error handler + standard response for one endpoint (owner: backend). (3–4d)
4. Storage spike: SQLite PoC for `repositories.py` + migration README (owner: backend). (4–6d)
5. Triage: schedule milestone planning + invite contributors to a 60‑min kickoff demo (owner: maintainer). (half day)

Acceptance for sprint: PRs open for items 1–4, CI green for docs, and a short kickoff meeting recorded in `docs/`.

---

## How we'll measure progress (signals) 📏
- Process: % of PRs that use templates and pass CI on first run.
- Quality: reduction in `docs` duplication; one endpoint standardized to shared error model.
- Velocity: number of small, reviewable PRs merged per week (target: 3–6).

---

## Next actions (today) — immediate low-effort wins
- Merge this `ROADMAP.md` and reference it from `README.md`.
- Open the four starter issues (IA, CI, error-handler spike, storage spike).
- Schedule a 60‑minute milestone kickoff (share agenda + desired outcomes).

---

### Contacts / maintainers (suggested)
- `@docs-maintainer` — docs & IA
- `@backend-maintainer` — API / storage
- `@infra-maintainer` — CI / observability

(Consider adding a small `CODEOWNERS` after Phase 1.)

---

This roadmap is intentionally short, prescriptive, and reversible — designed to help a small team make steady, low-risk progress while avoiding large, blocking rewrites.