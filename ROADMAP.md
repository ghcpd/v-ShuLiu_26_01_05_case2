# ROADMAP: Documentation Site (high-level)

## Background — why a roadmap is needed now

The documentation site has grown organically and is actively used, but contributors have added pages and features unevenly. Symptoms: inconsistent information architecture, duplicated or orphaned pages, lack of clear ownership, and ad-hoc editing patterns (see `docs/contributing.md` and `docs/issues_snapshot.md`). With a small contributor pool and limited time, we need a concise, practical roadmap to reduce friction, enable parallel work, and make the repository easier to maintain.

> **Goal:** Stabilize the documentation so contributors can work safely and independently, and then add lightweight discoverability and governance features.

---

## Principles

- Keep changes incremental and low-cost.
- Prioritize high-impact, low-effort work first (quick wins).
- Make decisions explicit and reversible (consensus + small experiments).
- Treat this roadmap as a living document and adapt based on outcomes.

---

## Phases / Milestones

### Phase 1 — Audit & Stabilize (2–4 weeks) ✅
Goals:
- Perform a content inventory (which pages exist, what's duplicated, what’s outdated).
- Establish a minimal taxonomy and simple front-matter metadata convention (title, section, owner, tags).
- Create or update **content templates** and a short, actionable editing guide inside `docs/contributing.md`.
- Implement quick-win cleanup PRs for the top 5–10 high-traffic pages.

Why this first: an agreed map and templates reduce friction and make later automation safer.

Deliverables / Acceptance:
- Content inventory spreadsheet or `docs/content-inventory.md` completed.
- Templates added to `docs/templates/` and applied to a sample set of pages.
- 5–10 high-impact cleanup PRs merged.


### Phase 2 — Collaboration & Workflow Improvements (3–6 weeks) 🔧
Goals:
- Add PR and issue templates, clear **labels** and a lightweight review process (who approves what).
- Introduce CI checks for docs (linkcheck, front-matter validator, simple linting).
- Establish a small editorial ownership model and an onboarding checklist for new contributors.
- Create short, focused contributor training (1–2 documents / a short video or walkthrough).

Why this second: tooling and governance reduce recurring merge conflicts and improve quality with little ongoing overhead.

Deliverables / Acceptance:
- Templates and labels in `.github/`.
- CI scheduled checks run successfully in PRs.
- Documented editorial owners and a triage schedule.


### Phase 3 — Discoverability & Lightweight Governance (4–8 weeks) 🔎
Goals:
- Add search and tagging (e.g., Algolia, Lunr, or built-in static search) and standardize tags in metadata.
- Implement minimal analytics and a feedback mechanism for readers.
- If needed, introduce a lightweight permission model (maintainers / editors) and documented escalation paths.

Why this later: search and governance are higher-impact once content is organized and workflows are stable.

Deliverables / Acceptance:
- Search index built and validated for key pages.
- Tags applied to a majority of active pages.
- Simple analytics dashboard and a feedback flow for content issues.

---

## How this helps contributors work in parallel

- Content work (Phase 1) can be split by section owners; multiple volunteers can run audits and small fixes concurrently.
- Developer/Infra work (Phase 2 and 3) — CI, PR templates, and search setup — can be implemented in parallel with content cleanup because they are orthogonal.
- Sequencing: agree on taxonomy/metadata before large-scale renames or tag application; CI and PR tooling may be rolled out earlier and enforced gradually (soft-enforced at first).
- Small working groups (2–4 people) limit coordination overhead and enable fast decision cycles.

---

## Key uncertainties & decision points

- Site generator / UI: Keep the current static markdown approach, or adopt a framework (MkDocs, Docusaurus) — this affects search and plugin options.
- Search provider: hosted (Algolia) vs self-hosted (Lunr) vs native GitHub Pages solutions (cost and maintenance tradeoffs).
- Permissions & hosting: whether to introduce access controls (SSO, RBAC) or keep repo-centric GitHub workflows.
- Scale & multi-tenancy: whether we need to support separate product docs or multi-tenant docs (adds complexity).
- Contributor availability and priorities — timelines assume limited part-time contributors.

Plan to resolve these via small experiments and a decision checklist prepared during Phase 1.

---

## Out of scope / Not now

- Full visual redesign of the site (too costly and disruptive).
- Replacing the repo with a proprietary CMS or a heavy migration.
- Multi-tenancy, SSO, or complex RBAC systems (high cost and unclear benefits for current user base).
- Large paid search solutions until we validate value from lightweight search.

Rationale: These items either require significant engineering effort or introduce ongoing costs that are not justified at this stage.

---

## Quick wins and suggested metrics (to track progress)

Quick wins:
- Add templates and a short editing guide (Phase 1).
- Add a link-checking CI job (Phase 2).
- Fix top 5 pages for clarity and canonical links (Phase 1).

Suggested metrics:
- Template adoption rate (percentage of new PRs using templates).
- Number of duplicate/orphan pages removed.
- PR review time for docs (target: <48 hours for small edits).
- Search coverage and average result relevance score (once search is implemented).

---

## Next steps (first 30 days)

1. Convene a **working group (3–5 people)**: one infra/engineer, one editor/maintainer, one or two content volunteers.
2. Run a 2-week content audit (Phase 1 kickoff): generate `docs/content-inventory.md` and propose taxonomy choices.
3. Create templates and submit the first cleanup PRs.
4. Add one CI job (linkcheck) to catch regressions early.

---

> **Note:** This roadmap is intentionally prescriptive but lightweight—prioritize consensus on a few decisions, iterate quickly, and revisit after each milestone.


---

*Document created as a living plan. Update this file as decisions are made and milestones complete.*
