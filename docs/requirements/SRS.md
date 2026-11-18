# Software requirements specification

Date: 2026-11-17. Owner: coordinator. Status: proposed V1 baseline for review.
No application exists. Every numeric threshold below is a proposed acceptance target,
not measured performance. Human acceptance precedes the relevant implementation milestone.

## Interpretation and precedence

This file is the canonical requirement register. FR identifiers define functionality;
NFR identifiers define cross-cutting acceptance. Architecture documents elaborate these
requirements, not a second independent requirement list. Tests use matching T-FR/T-NFR
identifiers in [test strategy](../testing/TEST_STRATEGY.md). [Traceability](../planning/REQUIREMENTS_TRACEABILITY.md)
maps every requirement to an owner module, milestone and evidence. Any conflicting
detail is a specification defect to resolve before implementation, not permission to
choose silently. Current authorization covers documents only, with $0 spending.

Terms: an approved candidate authorizes bounded acquisition of a named resource;
an approved source version authorizes use of exact acquired bytes. A source version
is immutable. A source epoch is a monotonic workspace counter invalidating derived
evidence when visibility changes. A job is a durable operation, not a network request.
Owner means the authenticated account owning the workspace. All requirements apply
to the initial PRD scope unless stated otherwise. Archive is read-only, not deletion.

## Functional register

Each row is mandatory for V1. Acceptance includes its named negative case; mocks alone
do not establish integration, provider capability or security evidence.

| ID | Requirement | Measurable acceptance criteria |
| --- | --- | --- |
| FR-001 | Register/sign in using approved OIDC identity | Valid issuer/subject with verified email creates exactly one account; repeated login reuses it. Invalid signature/issuer/audience/nonce, unverified email, replayed state/code are rejected. Email equality alone never links accounts. |
| FR-002 | Manage sessions and logout | Session idle timeout 30 minutes, absolute lifetime 7 days; logout/revoke invalidates the server record immediately. Missing/expired cookies and foreign CSRF fail. Fresh authentication within 10 minutes is required for export, account deletion and credential mutations. |
| FR-003 | Create, revise and archive learning workspaces | Owner creates within quota; title/goal revision persists. Stale expected revision returns conflict with no overwrite. Archive makes learning mutations unavailable while owner can read, export, unarchive or delete. |
| FR-004 | Clarify objective and prerequisites | Before pathway proposal record goal, target outcome, experience and prerequisite disposition. Ambiguous goals produce clarification; skipped diagnostics are marked unverified. Each proposed prerequisite is justified by objective/source coverage or explicitly labelled assumption. |
| FR-005 | Discover relevant current resources | Return up to five distinct eligible recommendations per discovery request, normally five; fewer include a reason. Each card contains publisher/URL/coverage rationale, checked-at date, known or unknown publication date, permission status and format. Fabricated or unreachable URLs fail source checks. |
| FR-006 | Approve/reject resources and exact content | No content acquisition without candidate approval and permission evidence; no extraction/model indexing before acquired SHA-256 version approval. UI shows acquired metadata/hash and change status; rejection never publishes content. Pre-known exact hashes may combine the two approvals. |
| FR-007 | Acquire permitted HTML/PDF or upload | Enforce URL/network/format/byte/page/token limits; checksum exact bytes; no cookies or ambient secrets forwarded. Denied rights, SSRF fixtures and unsupported files fail with safe reasons; forbidden connections do not occur. |
| FR-008 | Track, cancel and retry ingestion | A committed request returns a durable job ID. Polling exposes stage and terminal result/error. Queued cancellation prevents execution; running cancellation fences publication. Retry creates no duplicate active version and cannot revive a deleted source. |
| FR-009 | Extract hierarchical content with provenance | Every searchable passage has source/version, section/ancestor path, original page or HTML anchor and exact text offsets. Fixtures retain heading order, code blocks and parent content; inferred headings are flagged; empty/scanned/unsupported documents fail explicitly. |
| FR-010 | Publish a complete searchable version atomically | During publish/crash tests readers see the old or the complete new active version, never partial sections/embeddings. Missing objects, revoked approval, quota breach or stale fencing token prevent activation. |
| FR-011 | Retrieve within authorized knowledge | Search returns only owner-visible active approved versions with resolvable spans. Deleted/foreign IDs yield no content. Empty/insufficient results are explicit; retrieval quality is evaluated separately under NFR-010/test strategy. |
| FR-012 | Ground explanations and abstain | Every source-dependent substantive claim references validated source spans or is marked unsupported and withheld. Fabricated IDs/offsets fail deterministic checks. Unsupported questions return insufficient evidence/clarification; semantic support/correctness are measured independently, not inferred from ID validity. |
| FR-013 | Propose/version/accept a learning pathway | Pathway objectives and prerequisite edges form an acyclic graph, have coverage or explicit gaps, and reference the goal/source epoch. Acceptance uses expected revision; regeneration creates a new version and never erases accepted attempts. |
| FR-014 | Tutor interactively with durable turns | One active turn per session; request persists user message/job atomically. Completed answer and citations persist atomically; stale epoch, deleted data or conflicting revision blocks completion. Timeout preserves user input and returns actionable terminal status; reload never silently resubmits. |
| FR-015 | Generate aligned exercises and rubrics | Each exercise version references one or more pathway objectives, source epoch, allowed answer format, rubric and grading version. Uncovered objectives or invalid generated schemas fail publication. Rubric visible before submission; hidden test details may remain withheld. |
| FR-016 | Score and explain attempts | Objective answers use pinned answer/test keys; free responses receive rubric-based provisional assessment, not deterministic truth claims. Duplicate submission does not double-count. Runtime/provider failure is indeterminate, not a failing score; regrade preserves original and replacement provenance. |
| FR-017 | Execute bounded Python exercises | Only authorized attempts run on isolated executor; enforce 32KiB code, 2 CPU seconds, 5 seconds wall, 256MiB RAM, 16 processes, 10MiB writable storage and 64KiB combined output. Escape/network/persistence probes fail. No safe runner means explicit unavailable status and unmet release gate. |
| FR-018 | Persist evidence-based progress | Accepted assessed attempts append uniquely keyed evidence; deterministic projection can be rebuilt identically. Views distinguish unverified, practicing and demonstrated outcomes under the documented rubric. Visits/chat volume do not increase mastery; invalidated evidence marks affected results stale. |
| FR-019 | Resume across sessions | After logout/browser restart, same account sees accepted pathway version, completed turns, attempts, progress and pending jobs. Another owner cannot resume it. Lost network cannot cause duplicated attempts or lost committed progress. |
| FR-020 | Select platform/BYOK model bindings with consent | Both chat and embedding capabilities have explicit provider/model/billing mode and data-use consent revision. Unauthorized routing, silent fallback and incompatible embedding-model mixing are rejected. Release demonstrates both platform and BYOK paths using approved capacity. |
| FR-021 | Secure credential lifecycle | Save/test/revoke/delete returns only metadata; key absent from all read responses, exports, logs and jobs. Cross-owner use rejected; revocation blocks new dispatch admission using the same binding/credential transaction locks; already-admitted calls may still transmit or bill and are disclosed. Tests themselves require cost admission. |
| FR-022 | Bound and record model spending | Concurrent calls atomically reserve upper-bound local cost before dispatch; default deployment budget $0 blocks paid calls. Unknown pricing or unknown call outcome never releases funds optimistically. Settle once; show estimated/actual/unknown usage distinctly. |
| FR-023 | Refresh sources with reapproval | Refresh stages a new immutable version and requires exact-hash approval; old version stays active until replacement is complete unless permission/approval was revoked. Concurrent refreshes cannot overwrite each other; unchanged checksum causes no redundant reindex. |
| FR-024 | Delete a source and dependent content | In the deletion transaction deny reads, increment epoch and fence jobs. Purge source/derivatives and dependent generated excerpts within 24h; show content-free citation tombstones only. Late workers and restored backups cannot republish deleted content. |
| FR-025 | Export/delete an account | Fresh-auth owner export contains their goals, sources metadata, pathways, conversations, attempts/progress in versioned JSON, excluding credentials and content without redistribution permission. Deletion immediately revokes access and follows 24h active/30d backup bounds; jobs expose completion/failure. |
| FR-026 | Recover durable work | Killing worker before/after claim, external call and DB commit results in bounded retry or an explicit failed/unknown outcome. Effects are deduplicated; stale workers fail fencing. Ambiguous billable dispatch never triggers blind retry. |
| FR-027 | Report incorrect teaching/assessment | Owner can attach a report category and optional explanation to an owned turn/assessment; submission persists and returns an ID. Reports do not alter grades automatically or transmit private content externally. Resolution/outcome is visible when processed. |
| FR-028 | Provide accessible learner workflows | Registration through progress/export can be completed keyboard-only with visible focus, labelled controls, announced errors/status and non-colour-only meanings. Coding has a plain-text alternative; automated and manual checks follow NFR-009. |
| FR-029 | Enforce ownership everywhere | Cross-owner matrix covers records, jobs, search, blobs, citations, exports, credentials and pooled DB connections: no foreign content or mutation. Missing auth/context denies by default; submitted owner IDs are never trusted. |
| FR-030 | Recover from discovery failure | Search outage/disabled provider shows a reason and offers permitted manual URL/upload; user can continue approval/ingestion from that route. Discovery success is never recorded for the fallback. |

## Proposed capacity and workload definitions

Validation envelope E1: 100 provisioned accounts, 20 concurrent active learner sessions;
5 workspaces/account, 20 active sources/workspace; each source <=50MiB, <=1,000 pages
where applicable, <=500,000 extracted tokens; <=100,000 passages/workspace and
<=1,000,000 deployment-wide. Independent ceilings apply: not all per-user maxima
are simultaneously admitted. Reserve storage before upload and passages before publish.
Persisted bytes cap is an additional operator setting fixed from measured storage and
approved budget before M03; absent configuration blocks admission, never means unlimited.

E1 test workload: 10 ordinary API requests/second for 30 minutes, then 30/second for
60 seconds; simultaneously five active model turns, two ingestions and two code runs,
with up to 100 pending jobs. Ordinary API mix: 60% reads of workspace/progress/job,
20% non-model mutations, 20% session/metadata requests; retrieval measured separately
at 2 queries/second. Use fixed synthetic corpus manifests at both small and maximum
passage scale, real query distributions specified in test strategy. Fix measured
hardware, database/index size, model/provider, network and cache state in the report.
These bounds are a test scenario and admission limits, not a commercial capacity promise.

## Non-functional register

| ID | Area | Proposed measurable acceptance |
| --- | --- | --- |
| NFR-001 | Security | All ownership, session, SSRF, secret-redaction and sandbox negative cases pass; no unresolved critical/high exploitability finding in release scope after independent review. TLS and least-privilege deployed configuration inspected. Severity rationale recorded, not inferred from a scan alone. |
| NFR-002 | Privacy | Denial takes effect at deletion commit; active payload purge <=24h; backup expiry <=30d including WAL/replicas; redacted operational logs 14d, security metadata 30d. Restore suppression and provider disclosure tests pass. No training/advertising reuse of private data. |
| NFR-003 | Reliability | Fault tests at every durable transition produce no lost acknowledged mutation, duplicate graded attempt/progress event, or partial publication. 100 repeated kill/retry schedules pass; unknown external outcomes remain explicit. |
| NFR-004 | Availability | Initial objective 99.5% successful critical-journey probes over a rolling 30d at 1-minute intervals; success means valid login/session read, knowledge retrieval and a tutor outcome other than infrastructure failure. Insufficient-evidence is a valid outcome. Count provider failures in end-to-end figure; report internal-only figure separately. Planned maintenance is not silently excluded. Before launch, a 7d soak is evidence only, not proof of 30d SLO. |
| NFR-005 | Latency | Under E1, ordinary API p95<=500ms server time; retrieval p95<=2s; tutor accepted/status response<=1s p95 and completed grounded answer<=20s p95 including queue/provider, hard deadline60s. Supported 100-page text-PDF ingestion p95<=120s from exact-version approval including queue; worst-case-size files bounded by resource deadlines, not this percentile. Report error/timeout fraction, not only successes. |
| NFR-006 | Scalability | E1 completes with <1% infrastructure errors, no data isolation/integrity failures, bounded configured memory/storage/queue and predictable 429 admission at overload. Limits enforced globally across >=2 workers; do not use a per-process semaphore as the sole global limit. |
| NFR-007 | Cost | Default approved spend is $0. Every external billable operation, including discovery/testing/embeddings, is admitted against numeric limits and recorded price version. Concurrent stress never exceeds local reserved+settled caps. Future caps/funding need explicit approval; provider-wide bills are not guaranteed by local accounting. |
| NFR-008 | Maintainability | Reviewed migrations, generated API schema/types, strict TypeScript checks, Python type/lint checks, unit/integration tests and module-boundary checks pass in CI. Public contract breaking changes require version/migration plan; new developer can reproduce documented checks in a clean pinned environment without production credentials. |
| NFR-009 | Accessibility | Critical flows conform to applicable WCAG 2.2 AA criteria, verified by automated scans plus keyboard, zoom/reflow and screen-reader review. No critical/serious automated violation or unresolved applicable manual failure. Third-party identity flow must be assessed too; do not claim full conformance from scans alone. |
| NFR-010 | Reproducibility and quality | Fixtures/configuration/parser/index/embedding/model/prompt/rubric/image versions and hashes accompany every evaluation. Deterministic extraction/scoring/progress replay matches golden outputs; nondeterministic model runs retain settings/raw secured evaluation outputs and uncertainty. Retrieval/answer quality gates in test strategy must pass independently of identifier validity. |
| NFR-011 | Observability | Each sampled request/job/provider call has correlated trace ID, stage, duration and redacted outcome; 100% failed jobs and security/budget events emit metadata. Synthetic incident reaches responsible operator within 5min. Canary private payloads/keys never appear in telemetry; retention expiry verified. |
| NFR-012 | Recovery | Restore exercise achieves RPO<=15min and RTO<=4h, measured from interruption to verified service recovery. Deletion/credential revocation replay precedes access; reconciliation covers DB/object versions and unknown calls. Record evidence; backups alone do not satisfy this criterion. |

WCAG source: [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/), consulted 2026-11-17.
No accessibility or legal certification is asserted. Funding, hosting, workload realism
and staffing may require changing proposed targets through a recorded decision before
validation, never retroactively hiding a failed test.
