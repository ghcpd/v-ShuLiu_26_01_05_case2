# Project roadmap — Team Reminder Service

> Short, practical plan to stabilize documentation, collaboration, and developer workflows for the next phase of the project.

## Background — why a roadmap now

- The codebase and docs have grown organically; there is no single source of truth (`docs/contributing.md` explicitly notes "no official roadmap").
- Known pain points in `docs/issues_snapshot.md` (mixed storage, inconsistent error handling, missing background worker, limited observability) are creating developer friction and making coordinated changes risky.
- A small contributor pool and limited time mean we must prioritize low-risk, high-impact improvements that let multiple people work in parallel without blocking core delivery.

## Principles

- Prioritize stability and clear ownership over new features.
- Make incremental, reversible changes (small PRs, migration paths) so users and integrators aren’t blocked.
- Improve collaboration and DX (developer experience) first — that reduces future coordination cost.

---

## Milestones (3 phases)

Each milestone is scoped for a small team (1–3 contributors) and designed to be completed in ~2–6 weeks depending on scope.

### Phase A — Stabilize foundations (safety & consistency) ✅
Goal: Eliminate structural inconsistencies that block contributors and put a firm baseline in place.

Deliverables / scope:
- Standardize storage approach and repository interfaces
  - Decide short-term default (e.g., keep in-memory for tests + add a canonical `Postgres`-backed `Repository` implementation behind a clear interface in `app/repositories.py`).
  - Add migration plan / wrapper so both can coexist during transition.
- Consistent error handling and API contract hygiene
  - Introduce a shared exception model / error handler and apply to high-traffic endpoints in `api/`.
  - Add minimal API contract tests.
- Update contributor docs and ownership
  - Add CODEOWNERS, PR template, CONTRIBUTING updates (expand `docs/contributing.md`) to show where to file infra vs content changes.
- Quick wins for observability
  - Add structured logging and a basic metrics counter (minimal, pluggable).

Success criteria:
- `app/repositories.py` exposes a stable interface with at least one DB-backed implementation and unit tests.
- No remaining endpoints that return inconsistent HTTP error shapes for the same class of errors.
- Contributors can open small, focused PRs using templates and see immediate CI feedback.

Why first: These are low-level blockers that cause merge conflicts and break client expectations.

---

### Phase B — Collaboration & editing experience (workflow & process) 🔧
Goal: Make it safe and fast for contributors to add/edit docs and code without stepping on each other.

Deliverables / scope:
- GitHub workflow and CI improvements
  - Add/linting, pre-commit (black/isort/ruff), and test runs on PRs.
  - Enforce branch protection rules and PR checklist.
- Documentation information architecture (IA) cleanup
  - Audit `docs/` and `README.md` to define canonical sections (Getting started, Architecture, How-to guides, API reference).
  - Move/rename pages to follow the IA; add a small navigation index page.
- Clear ownership and small-scope tasks
  - Create a lightweight roadmap + milestone labels in the issue tracker; tag issues that block others vs. independent.
- Editing UX
  - Add a documented local dev flow (how to run app/docs locally) and a `docs` PR checklist.

Success criteria:
- New contributors complete a docs + code change in <1 hour from first clone to PR.
- Reduced merge conflicts on `docs/` and `app/` changes.

Why second: Once foundations are stable, improving processes multiplies contributor throughput.

---

### Phase C — Enable advanced features incrementally (scale & polish) ✨
Goal: Add higher-value capabilities that improve discoverability and reliability while avoiding heavy maintenance burden.

Deliverables / scope (incremental / opt-in):
- Search & metadata for docs
  - Add simple static-site-friendly search (e.g., Lunr) or a lightweight server-side index; add `tags`/`categories` in docs front-matter.
- Scheduling & background processing
  - Implement a simple scheduler (e.g., APScheduler) or a task-queue adapter with a clear contract; keep worker optional behind a feature flag.
- Observability and reliability
  - Add request metrics, structured traces (lightweight), and alerting playbook.
- API governance (optional, gated by consumer needs)
  - Decide on API versioning approach and add changelog + deprecation policy.

Success criteria:
- Search improves discoverability for common docs by measurable queries.
- Reminders can be scheduled reliably in non-manual flows in staging.

Why third: These features are higher-impact but require the stability and collaboration improvements from earlier phases.

---

## How this roadmap enables parallel, non-blocking work

- Layered ownership:
  - Infra & core (storage, error-handling, background worker) — small infra owner(s).
  - Docs & IA — content maintainers; can refactor pages independently using a migration checklist.
  - Developer Experience (CI, pre-commit, CODEOWNERS) — single owner to apply changes that benefit everyone.
- Parallelism guidance:
  - Phase A tasks that touch different layers can proceed in parallel (e.g., add DB-backed repository implementation while docs team reorganizes `docs/`).
  - Sequencing required when a dependency exists (e.g., API contract changes must land before consumer-facing version bumps).
  - Use feature branches + integration branch for larger infra changes; break large migrations into incremental PRs.
- Daily/weekly lightweight syncs:
  - Maintain a short weekly milestone board and require owners to add status to issue comments so others can choose unblocked work.

---

## Key uncertainties & decision points

- Storage backend choice (keep in-memory + DB adapter vs. full DB migration). Impact: affects `app/repositories.py`, testing strategy, and deployment.
- Background job approach (in-process scheduler vs. task queue). Impact: operational complexity and SLA for reminders.
- API versioning policy (do now or defer). Impact: how we introduce breaking changes and notify consumers.
- Observability depth (minimal logging vs. full tracing). Start small and expand.

Each decision should be made with a short ADR (architectural decision record) and a migration/rollback plan.

---

## Intentional "Not now" / Out of scope

- Full UI re-write or move to a new frontend framework — low ROI for current contributor size and adds large maintenance burden.
- Multi-tenancy or large-scale permissioning system — requires product/requirements clarity and significant infra changes.
- Real-time collaborative editing features (live cursors, CRDTs) — high complexity and not required by current use-cases.
- Large third-party integrations (enterprise SSO, billing) unless a clear downstream requirement arises.

Rationale: keep the next phase focused on stability, collaboration, and low-friction contributor growth.

---

## Suggested milestone labels / timeline (example)

- Phase A (3–6 weeks): `foundation/stable-storage`, `foundation/errors`, `chore/contrib-docs`
- Phase B (2–4 weeks): `process/ci`, `process/ia`, `process/onboarding`
- Phase C (4–8 weeks incremental): `feature/search`, `feature/scheduler`, `feature/observability`

Aim to ship minimal, test-covered changes every 1–2 weeks.

---

## Next steps (short-term, first 7 days)

1. Create lightweight ADRs for storage and background jobs (owners: infra lead).
2. Add CODEOWNERS, PR template, and a `roadmap` label on the repo.
3. Triage `docs/issues_snapshot.md` items into the Phase A milestone and create 1–2 small starter issues.

---

## Closing

This roadmap treats the repository as a living plan: prefer small, reversible improvements that reduce friction and let contributors work in parallel. Follow the milestones above to move from brittle, ad-hoc contributions to a predictable, maintainable project.
