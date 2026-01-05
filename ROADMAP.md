# Team Reminder Service – Roadmap

## Background & Motivation

This service started as a small internal tool and has evolved organically as multiple contributors added endpoints, data models, and experimental features. The current codebase is functional but shows increasing signs of inconsistency and unclear ownership:

- **Inconsistent API design**: Some endpoints follow structured error responses; others return ad-hoc dicts
- **Architectural ambiguity**: Business logic is split between route handlers, `services.py`, and repositories; unclear ownership makes it hard to avoid duplication
- **Data layer uncertainty**: The repository layer supports both in-memory storage and database abstractions, but the two paths don't clearly coexist
- **Missing production features**: No background job scheduler, minimal logging, no error recovery; reminders only work via manual API calls
- **Undefined contribution model**: The contributing guide acknowledges the lack of a roadmap and leaves priorities and sequencing unclear

As the team grows and relies on reminders being sent reliably, these inconsistencies become blockers. A roadmap establishes clear phases, reduces parallel friction among contributors, and signals what is intentionally deferred.

## Philosophy

- **Small contributor pool, limited time**: We optimize for clarity and sequential impact, not for maximum parallelism
- **Living document**: This roadmap will evolve as we learn; decisions marked with `?` are open questions to revisit
- **Fix core first**: Prioritize stability, consistency, and observability over new features
- **Defer complexity**: Multi-tenancy, advanced scheduling, and admin dashboards are explicitly out of scope until core patterns are solid

## Phases & Milestones

### Phase 1: Foundation – Consistency, Clarity, Observability (2–4 weeks)

**Goal**: Establish clear patterns for APIs, data access, and logging so future work can proceed without rework.

#### Key Deliverables

1. **API Error Handling Standardization**
   - Audit all endpoints in `users.py` and `notifications.py`
   - Ensure all endpoints use structured `ErrorResponse` model
   - Define HTTP status code conventions (e.g., 400 for validation, 404 for not found, 500 for internal errors)
   - Update inconsistent endpoints (e.g., `GET /users/{user_id}`, `POST /notifications/send/{reminder_id}`)

2. **Data Layer Clarity**
   - Commit to in-memory storage as the data persistence layer for this phase
   - Rename/consolidate repository classes if needed (e.g., consistent naming)
   - Add inline documentation to `repositories.py` explaining why in-memory is chosen for now and what a migration path would look like
   - Create a placeholder interface/abstract base class for future database backend (do NOT implement the backend yet)

3. **Basic Observability**
   - Add structured logging to all route handlers and services using Python's `logging` module
   - Log request entry/exit with unique request IDs for tracing
   - Extend `/health` endpoint to include basic dependency checks (e.g., "store is reachable")
   - No external tools (Datadog, ELK) required; just file or stdout logging for now

4. **Documentation & Ownership**
   - Update `contributing.md` to specify where new code should live (APIs in `api/`, business logic in `services/`, data access in `repositories.py`)
   - Document the module ownership model (e.g., who reviews changes to each module)
   - Add a `ARCHITECTURE.md` explaining the current design and planned future evolution

#### Why This Comes First

These changes reduce friction and rework downstream. Once contributors agree on error handling and where code lives, Phase 2 can proceed without stepping on each other.

#### Parallel Work Opportunities

- Error handling standardization can happen in one PR
- Logging can be added to route handlers and services independently
- Documentation updates can happen in parallel
- **Blocker**: New features should be paused until Phase 1 is merged to avoid merge conflicts

---

### Phase 2: Reliability – Background Jobs & Scheduled Reminders (3–5 weeks)

**Goal**: Enable reminders to be sent automatically on a schedule, not just on-demand. Establish observability into reminder delivery.

#### Key Deliverables

1. **Background Job Scheduler**
   - Implement a simple in-process scheduler (e.g., using `APScheduler` or similar) that runs reminder checks at regular intervals
   - Define a `ReminderScheduler` service that fetches reminders due to run and triggers sending
   - Store last-run timestamps for each reminder to avoid duplicates
   - Add configuration for scheduler check frequency (e.g., every 5 minutes) and batch size

2. **Improved Reminder Sending**
   - Extract the sending logic from `services.py` into a dedicated `NotificationSender` class
   - Implement basic retry logic (e.g., retry up to 3 times on failure, with exponential backoff)
   - Track and log delivery outcomes (success, failure reason, retry count)
   - Update `SendResult` model to include retry metadata

3. **Enhanced Health Checks**
   - Extend `/health` to verify that the scheduler is running and has processed reminders recently
   - Add metrics or logs that expose scheduler health and delivery stats
   - Define SLAs (e.g., "reminders should be sent within 5 minutes of their scheduled time")

4. **Pagination & Filtering (Cleanup)**
   - Add `limit` and `offset` query parameters to `GET /users/` and `GET /notifications/user/{owner_id}/`
   - Add optional filtering (e.g., by active/inactive status for users, by channel or frequency for reminders)
   - This prevents the service from becoming slow as data grows

#### Why This Comes After Phase 1

The scheduler needs solid error handling (Phase 1) to avoid silent failures. It also needs a clear data access layer (Phase 1) to fetch reminders reliably.

#### Parallel Work Opportunities

- Scheduler implementation and improved sending logic can be developed in parallel (though they'll merge into `services.py`)
- Health check improvements can happen separately
- Pagination can be added incrementally to each endpoint
- **Sequence note**: Hold pagination until after scheduler is in place; prioritize reliability

---

### Phase 3: Advanced Features – Only After Phases 1 & 2 Are Stable (Future – not committed)

**Goal**: Once the core is reliable and well-understood, cautiously add complexity for advanced use cases.

#### Possible Future Work (Not Committed)

- **Database backend migration**: Introduce Postgres or another relational database, with a migration path from in-memory storage
- **Multi-tenancy**: Support multiple isolated teams or organizations sharing the service
- **Audit logging**: Full audit trail of reminder creation, updates, and delivery (required for compliance-sensitive teams)
- **Advanced scheduling**: Support cron-like rules, timezone-aware scheduling, or calendar-based reminders
- **Permission model**: Role-based access control (e.g., only reminder owner or admin can modify)
- **External integrations**: SMS via Twilio, Slack via API, email template customization

#### Gating Criteria

Before any Phase 3 work starts:
- Phase 1 & 2 must be merged, tested, and running in production for at least 2 weeks
- The team must have a clear use case or customer request for the feature
- An owner must volunteer to lead the work and maintain it long-term

---

## Collaboration & Parallel Work Strategy

### Layered Ownership Model

To minimize merge conflicts and unclear ownership:

- **API layer** (`api/users.py`, `api/notifications.py`): Route definitions, request validation, response marshaling. Any contributor can add endpoints here after Phase 1 standardization.
- **Service layer** (`services/`): Business logic (scheduling, sending, retry). One owner per service; others submit PRs with clear scope.
- **Data layer** (`repositories.py`): Data access and in-memory store. One owner; PRs require peer review to ensure consistency.
- **Models** (`models.py`): Shared Pydantic models. Backward-compatible changes preferred; breaking changes require RFC-style discussion.

### Phase 1 Execution Plan

1. **Week 1**: Error handling audit and fixes (one contributor, or pair-program to align on patterns)
2. **Week 2**: Logging implementation (add to each module independently; minimal merge conflicts)
3. **Week 3**: Documentation (`ARCHITECTURE.md`, update `contributing.md`; non-blocking)
4. **Review & merge**: One integrated PR or series of small PRs with clear dependencies

### Phase 2 Execution Plan

1. Start only after Phase 1 is merged and code is stable for 3–5 days
2. **Week 1**: Scheduler skeleton and service refactoring
3. **Week 2**: Retry logic and delivery tracking
4. **Week 3–4**: Health checks, tests, and integration
5. **Review & gradual rollout**: Test in a non-critical environment first; gradually enable for all users

### Reducing Blocking & Parallel Friction

- **Avoid long-lived branches**: Keep PRs small and mergeable within 2–3 days
- **Use feature flags**: For Phase 2 scheduler, wrap background job execution in a flag so it can be tested without disrupting current reminders
- **Clear acceptance criteria**: Every PR should have a checklist of what it enables in the next phase

---

## Key Uncertainties & Decision Points

### 1. **In-Memory Storage vs. Database**

- **Current state**: Repositories support in-memory dicts; some code assumes a database exists
- **Decision needed**: Should Phase 1 standardize on in-memory for MVP, or should we commit to Postgres now?
- **Recommendation**: **Stay with in-memory for Phases 1–2**. This is simpler, faster to test, and doesn't require infrastructure. Plan a Phase 3 migration to Postgres if the team grows or data retention requirements change.
- **Rationale**: Premature database selection introduces deployment complexity without clear benefit for a small, internal team

### 2. **Background Job Architecture**

- **Current state**: No scheduler; reminders are only sent via API calls
- **Decision needed**: Use a simple in-process scheduler (APScheduler), a task queue (Celery + Redis), or cron-based jobs?
- **Recommendation**: **Start with in-process scheduler (APScheduler)** in Phase 2. This has no external dependencies and is sufficient for a small team. Revisit after Phase 2 if we need horizontal scaling or fault tolerance.
- **Future migration**: If we outgrow a single process, Phase 3 can introduce Celery + Redis without changing the API

### 3. **API Versioning**

- **Current state**: No versioning; changes to endpoints are breaking
- **Decision needed**: Should we introduce `/v1/`, `/v2/` style versioning now, or manage breaking changes differently?
- **Recommendation**: **Not now**. Use semantic versioning for the service itself and communicate breaking changes clearly in release notes. Revisit after Phase 2 if we have external consumers who can't update immediately.

### 4. **Observability Stack**

- **Current state**: Minimal logging; no metrics or tracing infrastructure
- **Decision needed**: Should Phase 1 include Datadog, ELK, or similar, or just local logging?
- **Recommendation**: **Start with local, structured logging** (stdout or file). No external SaaS required. If the team grows or reliability becomes critical, Phase 3 can add Datadog or similar.

### 5. **Scope of "Reminder"**

- **Current state**: Reminders have a title, message, channel, time-of-day, and frequency (once/daily/weekly)
- **Decision needed**: Should we support more complex scheduling (e.g., cron rules, timezone-aware times, blackout periods)?
- **Recommendation**: **No, not now**. The current model (time-of-day + frequency) is sufficient for MVP. Phase 3 can add complexity if customers request it.

---

## Out of Scope / Not Now

### Why These Are Deferred

1. **Multi-Tenancy**
   - **Why deferred**: Adds significant complexity (data isolation, permission model, billing) without clear use case yet. Start with single-tenant.
   - **Revisit if**: Team decides to offer this service to external customers.

2. **Advanced Scheduling** (cron, timezone-aware, calendar-based)
   - **Why deferred**: Current time-of-day + frequency model covers 80% of use cases. Advanced rules add testing and maintenance burden.
   - **Revisit if**: Users request complex schedules (e.g., "every second Tuesday").

3. **Admin Dashboard UI**
   - **Why deferred**: API-only or CLI management is sufficient for a small internal team. UI adds frontend maintenance cost.
   - **Revisit if**: Non-technical stakeholders need to manage reminders.

4. **SMS & Slack Integrations**
   - **Why deferred**: Email is the lowest-common-denominator channel. Adding integrations means maintaining multiple vendor APIs and handling provider-specific failures.
   - **Revisit if**: Specific use cases require these channels, and a contributor volunteers to own them.

5. **Full Audit Logging**
   - **Why deferred**: Not required for an internal team. Add if compliance or regulatory requirements emerge.
   - **Revisit if**: Service is used for regulated workflows.

6. **Permission Model & RBAC**
   - **Why deferred**: Current model (any authenticated user can create reminders) is simple and works for small teams. Permission model adds complexity.
   - **Revisit if**: Team grows and role-based access is needed.

---

## Success Metrics & Review Checkpoints

### Phase 1 Success

- [ ] All endpoints return structured `ErrorResponse` on errors
- [ ] HTTP status codes are consistent across API
- [ ] Every route logs entry, exit, and errors
- [ ] Health check passes and includes basic dependency info
- [ ] Contributing guide and `ARCHITECTURE.md` are up-to-date
- [ ] No open questions about where code should live

### Phase 2 Success

- [ ] Reminders are automatically sent on schedule (no manual API call needed)
- [ ] Failed reminders are retried up to 3 times
- [ ] `/health` endpoint reports on scheduler status
- [ ] List endpoints support pagination (limit/offset)
- [ ] Logs include delivery outcomes and retry metadata
- [ ] Service runs in production for 2+ weeks without unexpected downtime

### Phase 3 Decision Point (End of Phase 2)

- [ ] Team aligns on whether to pursue database migration, multi-tenancy, or other features
- [ ] Clear ownership and volunteers are identified for any Phase 3 work
- [ ] Phases 1 & 2 are stable and well-tested

---

## Timeline & Resource Planning

**Assumption**: 1–2 contributors, ~10–15 hours per week each.

| Phase | Estimated Duration | Owner(s) | Risk |
|-------|-------------------|----------|------|
| Phase 1 (Foundation) | 2–4 weeks | 1 lead + 1 peer reviewer | Low; mostly cleanup |
| Phase 2 (Scheduler) | 3–5 weeks | 1 lead + 1 reviewer | Medium; scheduler logic is new; requires testing |
| Phase 3 (Advanced) | TBD; not committed | TBD | High; depends on unclear requirements |

---

## Review & Update Cadence

- **After Phase 1**: Review roadmap with team; collect feedback on API changes and observability choices
- **After Phase 2**: Decide whether to move to Phase 3 or double down on stability and documentation
- **Quarterly**: Revisit this document; update based on actual usage patterns and new requests from the team

---

## Questions & Feedback

This roadmap is a starting point. Open questions and discussion points:

- Should we introduce a database earlier than Phase 3? (Current: Phase 3+)
- Should multi-tenancy be part of Phase 3, or explicitly out of scope indefinitely?
- Are there regulatory or compliance requirements that should inform the audit logging priority?
- Should Phase 1 include integration tests, or are unit tests sufficient for MVP?

Please raise these in a GitHub issue or discussion; they'll inform the next iteration of this roadmap.
