# V1 implementation roadmap

Date: 2026-11-17. Planning output only. M01 documents are prepared for review;
no following milestone is authorized by this roadmap. No dates or effort estimates
are presented without team/capacity evidence. [SRS](../requirements/SRS.md) owns
acceptance; [traceability](REQUIREMENTS_TRACEABILITY.md) owns coverage.

## Dependencies, ownership and gate rules

M02 foundation -> M03 knowledge -> M04 retrieval -> M05 tutoring -> M06 assessment.
M07 code execution depends on M02 and M06; M08 release depends on M02-M07.
Security/privacy/cost controls are built with each feature, not postponed to M08.
Each milestone has one assigned implementer, separate reviewer when risk warrants,
human scope approval, feature branch, executed tests/static checks, reviewed diff,
decision updates, verified commit/push/PR, and a stop before the next milestone.
Expected paths below are implementation deliverables, not files already created.

Milestones may be split into reviewable PRs with the same acceptance scope. Failures
block dependent capabilities. An unsupported sandbox/provider must remain disabled;
the product cannot be declared V1 complete while mandatory requirements remain unmet.

## M02: foundation, identity and contracts

Prerequisites: human accepts M01 scope assumptions and selects identity/hosting/key
management approach; approved budget remains explicit, with local test doubles permitted
only under a subsequent implementation authorization. PR1 charter and M01 specs need
review/merge through the protected workflow before dependent production release.

Owner: implementation engineer; separate security review. Tasks:

- Establish Python/FastAPI module boundaries, TypeScript/Next.js client and same-origin
  reverse-proxy contract. Pin compatible dependencies and documented local setup.
- Implement OIDC/session/CSRF and owner-only workspace creation/revisions/archive;
  RLS/composite FKs, migration roles, transaction-local pool context and limits.
- Implement durable jobs/fencing/idempotency primitives and provider gateway interfaces,
  encrypted credentials/consent, usage reservation ledger with dispatch disabled by default.
- Generate OpenAPI/client types; add CI lint/type/unit/integration and secret checks.
  Define telemetry redaction and migration rollback patterns immediately.

Expected files: `backend/pyproject.toml`, lockfile, `backend/app/{identity,workspaces,jobs,gateway}`,
`backend/migrations/`, `frontend/package.json`, lockfile, `frontend/src/`,
`backend/tests/{unit,integration,contracts}/`, `.github/workflows/checks.yml`, `infra/local/`.
Tests: T-FR-001..003,020..022,026,029; T-NFR-001,007,008,011 at foundation scope.
Deliverable: authenticated private workspace, tested durable command and provider admission
boundaries using doubles. Gate: cross-owner/pool leakage zero, duplicate mutation zero,
all listed checks execute; no real provider route enabled without separate consent/capacity.

## M03: source discovery, approval and ingestion

Prerequisites: M02 passed; select permitted discovery service and parser; configure byte,
storage and passage quotas from measured fixtures. Acquire only permissioned test materials.
Owner: knowledge engineer; separate acquisition/security review.

- Implement discovery and manual fallback cards with date/permission provenance; two-phase
  candidate/exact-content approval, quarantine uploads and safe inert inspection.
- Implement SSRF-safe bounded fetch, isolated PDF/HTML extraction, hierarchy/span mapping,
  staged artifacts, source version refresh, cancellation and atomic publication primitives.
- Implement source deletion, dependent-content invalidation and object reconciliation now;
  embedding stage may be disabled until M04 but must not mislabel a lexical-only index.

Expected files: `backend/app/{sources,processing,knowledge,privacy}/`, new migrations,
`frontend/src/features/sources/`, `backend/tests/fixtures/documents/`,
`backend/tests/integration/test_source_lifecycle.py`, `infra/parser/`.
Tests: T-FR-005..010,023,024,030; applicable T-FR-026,029 and T-NFR-001..003.
Deliverable: approved supported sources become versioned searchable-ready content with
source spans and visible errors. Gate: malicious/oversize/revoked content cannot publish;
crash and refresh races preserve old complete version; active deletion purge demonstrated.

## M04: retrieval and grounded evidence

Prerequisites: M03; independent protocol review and human authorization for bounded
evaluation; fixed corpus rights, labels/splits/budgets and model adapters. No inherited
permission to train original STAIR or buy compute.
Owner: retrieval engineer; separate ML and data/research reviewers.

- Implement lexical and exact-vector reference retrieval, compatible embedding generations,
  hybrid candidate, hierarchy ablations, access filtering and context budgeting.
- Implement source-span citation resolver and structured grounding output validation.
- Run predeclared quality/performance evaluation; retain manifests, uncertainty and failures.
  Select simplest passing configuration; ANN/reranking only when justified by evidence.

Expected files: `backend/app/{retrieval,grounding}/`, knowledge migrations,
`backend/tests/integration/test_retrieval_isolation.py`,
`evaluation/{protocol.md,manifests,configs,run_retrieval.py,reports}/`.
Tests: T-FR-011,012,020,024,029; T-NFR-005,006,010; bounded load subset.
Deliverable: independently assessed retriever and grounding pipeline, documented choice.
Gate: approved quality gates met with uncertainty, deletion/ownership zero violations,
model migration atomic, measured costs within approved cap. Failure triggers narrower
scope or design revision before tutoring, not a claim that STAIR's reported score transfers.

## M05: goal interpretation, pathway and tutoring

Prerequisites: M04 accepted; goal/prerequisite and pathway rubrics approved; platform
inference capability feasible within explicitly approved limits.
Owner: learning engineer; separate product/security review.

- Implement clarification/diagnostic dispositions, objective DAG and coverage-gap checks,
  immutable pathway proposals/acceptance and plain-language learner flow.
- Implement durable tutor sessions, one-active-turn revision control, asynchronous polling,
  final grounded answers/abstention, source epoch invalidation and feedback reporting.
- Implement cross-session resume and accessible citation/lesson navigation.

Expected files: `backend/app/{learning,tutoring}/`, `frontend/src/features/{goals,pathways,tutor}/`,
`backend/tests/integration/test_tutor_concurrency.py`, `frontend/tests/e2e/learning.spec.ts`.
Tests: T-FR-004,012..014,019,027,028; T-NFR-002,005,009,010.
Deliverable: approved resources support a resumable pathway and evidence-backed lesson.
Gate: unsupported content abstains, no unvalidated output released, source deletion races
do not leak old excerpts, ambiguous goals/gaps are visible, accessibility review passes scope.

## M06: assessments and durable progress

Prerequisites: M05; accepted objective/rubric and provisional-grading policy; independent
review of scoring/projection invariants.
Owner: assessment engineer; separate data scientist review.

- Implement versioned exercise/rubric generation and validation, objective and provisional
  free-response grading, attempt deduplication and regrade provenance.
- Implement immutable evidence events, deterministic projections, invalidation/staleness,
  next-objective recommendation and resume without conflating activity with mastery.
- Provide progress/evidence views and reviewable reports; coding remains unavailable until M07.

Expected files: `backend/app/{assessment,progress}/`, new migrations,
`frontend/src/features/{practice,progress}/`, `backend/tests/unit/test_progress_replay.py`,
`backend/tests/integration/test_assessment_idempotency.py`.
Tests: T-FR-015,016,018,019,027,028; T-NFR-003,009,010.
Deliverable: reproducible assessment/evidence progression without unsupported efficacy claims.
Gate: same ledger rebuilds same projection, failures are indeterminate rather than zero,
regrade cannot double-count, source/rubric change marks evidence stale as specified.

## M07: isolated Python exercises

Prerequisites: M02 and M06; approved separate Linux/KVM execution host and image supply
chain; operator and sandbox threat review. If no suitable funded capacity exists, blocked.
Owner: execution engineer; independent security reviewer.

- Build broker/host protocol, fresh read-only image per run, external trusted harness,
  quotas, no guest network/secrets, bounded output, teardown and orphan reaping.
- Connect attempts to trusted run results and grading; show termination reason and retry
  without overwriting earlier attempt evidence. Provide accessible plain-code editor.
- Test escape/resource/identity/replay scenarios and host crash/lease recovery.

Expected files: `backend/app/execution/`, `executor/{controller,images,policies}/`,
`infra/executor/`, `backend/tests/contracts/test_executor.py`, `executor/tests/security/`,
`frontend/src/features/code/`.
Tests: T-FR-017,016,022,026,029; T-NFR-001,003,005..007,009,011.
Deliverable: bounded isolated Python execution with trusted scoring evidence.
Gate: all isolation probes fail safely, no host fallback, quotas measured at host boundary,
no cross-run state, no guest authority over grade, kill/teardown proven.

## M08: operational release qualification

Prerequisites: M02-M07 gates passed, all critical decisions closed, numeric budget/capacity
approved, named operator/incident coverage and provider/retention terms accepted.
Owner: release engineer; separate security, ML, data and research reviews for final claims.

- Complete owner export/account deletion UI and receipt, backup/restore deletion ledger,
  migration rollback, alerts/runbooks, provider outage/unknown billing reconciliation.
- Execute E1 load/burst/soak, source deletion under load, disaster restore, accessibility
  and end-to-end learner acceptance. Confirm both platform and BYOK modes.
- Reconcile all requirement evidence, unresolved defects, release notes and rollback.
  Require human release authorization; no automatic deployment from a passing document.

Expected files: `backend/app/privacy/`, `frontend/src/features/account/`,
`tests/{load,disaster,acceptance}/`, `infra/{deploy,backup,monitoring}/`,
`docs/releases/v1-readiness.md`, `docs/operations/runbooks/`.
Tests: all T-FR/T-NFR suites, with T-FR-025 and T-NFR-004,012 completed end-to-end.
Deliverable: evidence-backed release candidate and operational handoff.
Gate: no unresolved critical/high security/privacy defects, all mandatory requirements
have actual evidence, measured targets pass or human-approved revised targets are recorded
before retest; seven-day soak does not certify the monthly SLO. Human approves release.

## Unresolved decisions and risk register

| Decision / owner | Evidence needed and blocking point | Consequence / fallback |
| --- | --- | --- |
| Initial adult/English/HTML/PDF/Python scope / product owner | Accept PRD assumptions before M02 | Broader audience/formats requires scope/test revision |
| Identity provider and authentication accessibility / engineering+security | Terms, integration proof, verified-email/recovery behavior before M02 | No insecure local-password shortcut |
| Hosting, private object store, key management and backups / operations | Cost/capacity, encryption/restore feasibility before activation | $0 does not establish free reliable hosting |
| Discovery and model providers / product+ML | Permitted use, data terms, capability/pricing/limits before M03/M04 | Manual discovery fallback; mandatory platform inference still blocks release |
| Exact model/embed dimension and quality gates / ML+data+research | Independently reviewed protocol, development evidence, fixed confirmatory choices before M04 | Hybrid remains candidate; no result imported from paper |
| Source permission and extraction quality / knowledge+product | Allowed acquisition policy and fixture fidelity before M03 | Reject uncertain rights/unsupported documents, no DRM bypass |
| Progress defaults / product+data | Accept evidence threshold, family independence, expiry and provisional grading rules before M06 | Status means product evidence rule, not validated proficiency |
| Sandbox host and harness / execution+security | KVM support, hardening, limits and adversarial results before M07 | Disable execution; cannot release full V1 |
| Operator, incident and retention policy / human owner | Named responsibility, jurisdiction/provider terms and restoration drill before M08 | Release blocked; no invented compliance certification |
| GitHub eligible reviewer / repository owner | Independent approving account with write access | Protected PRs remain unmerged; no bypass by agent |

No unresolved item licenses unsafe implementation. Resolve in a dated decision with
alternatives, consequences, evidence, reviewer and approving authority. Changes to
requirements update SRS, contracts, tests and traceability in the same reviewed PR.

M04's evaluation-plan deliverable must include conversation-stratified last-turn/history/
rewrite comparisons and bounded development-only HPO, as specified in TEST_STRATEGY.
Its protocol, budget, sample size and uncertainty gates require separate review before
any run. The additional IBM sources authorize neither experiments nor a storage purchase.
