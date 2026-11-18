# Test and evaluation strategy

Date: 2026-11-17. Status: proposed M01 specification; all application tests and
evaluations below are unexecuted. Base: `f8c967f`. Owner: delegated learning/testing
author; independent review and human acceptance pending. This file defines future
acceptance work, not permission to implement, download data, call models, recruit
participants, or run experiments. Current execution budget remains $0.

Canonical requirements: [SRS](../requirements/SRS.md). Related contracts:
[architecture](../architecture/SYSTEM_ARCHITECTURE.md),
[security](../architecture/SECURITY_MODEL.md), and
[learning engine](../learning/LEARNING_ENGINE.md).

## Test ownership and gates

Each milestone owner implements relevant automated tests and records commands, code
revision, fixture hashes, environment, results, and omitted checks. A separate reviewer
assesses substantive acceptance evidence. Human authority accepts scope and residual
risks. An unrun test, unavailable sandbox/provider, or missing evaluation labels is
`blocked`, never a pass. A waiver must name the affected capability and disable it or
narrow its claim; it cannot waive tenant isolation, secret protection, or deletion.

Unit tests cover pure domain rules; integration tests use real persistence and job
transactions; contract tests cover provider adapters with deterministic fake services;
end-to-end tests cover browser journeys, reconnects, and authorization. Simulations
prove handling of simulated failures only. Provider/hosting integration and operational
targets need later authorized verification in the intended environment. No paid or
live external calls in ordinary CI; adapters fail closed unless explicitly configured.

Fixtures should be small, synthetic or rights-cleared, versioned, and include expected
outputs. Secrets must be dummy values. Test at least two owners/workspaces and forged
cross-owner IDs at every endpoint, job, blob, search, and citation boundary. Declare
test data provenance and disposal; never copy private learner content into CI.

## Functional acceptance matrix

Each ID is a test suite, not necessarily a single test. Every suite includes success,
invalid input, unauthorized access where relevant, and persisted-state assertions.

| Test / requirement | Required scenarios and observable pass condition | Gate |
| --- | --- | --- |
| T-FR-001 / FR-001 | Valid OIDC login with verified email; reject invalid state/nonce, unverified identity, replay and failed callback; no local password route | M02 |
| T-FR-002 / FR-002 | Session rotation/expiry/logout/delete; revoked session rejected on next protected request, cookie and CSRF behavior verified | M02 |
| T-FR-003 / FR-003 | Create/edit/archive goal; stale revision conflicts, archive hides active navigation while preserving authorized history | M02 |
| T-FR-004 / FR-004 | Ambiguous goal asks clarification; skipped diagnostic remains unknown; confirmed scope and prerequisite evidence persist | M05 |
| T-FR-005 / FR-005 | Five eligible source candidates or explicit fewer-results reason; deduplication, relevance/rights/freshness metadata, empty and failed search | M03 |
| T-FR-006 / FR-006 | Inspect/approve/reject exact candidate version; changed content needs renewed approval and rejected resource never enters fetch queue | M03 |
| T-FR-007 / FR-007 | Permitted fetch/upload succeeds; redirect-to-private-address, oversized, malformed, unsupported and unapproved content rejected safely | M03 |
| T-FR-008 / FR-008 | Durable ingest status, cancel and bounded retry; disconnect does not lose job and cancellation prevents later publication | M03 |
| T-FR-009 / FR-009 | Fixtures with nested headings, tables, repeated headers and missing TOC; extracted spans resolve to original content and uncertainty is exposed | M03 |
| T-FR-010 / FR-010 | Crash at each publish boundary; partial version never searchable; complete version becomes atomic active version, old version survives failed refresh | M03 |
| T-FR-011 / FR-011 | Authorized retrieval excludes foreign, rejected, staged, deleted and wrong embedding-generation passages; empty results explicit | M04 |
| T-FR-012 / FR-012 | Valid citations and human-supported claims; missing/conflicting evidence causes appropriate abstention; hostile source instructions have no authority | M04/M05 |
| T-FR-013 / FR-013 | Reject cyclic/dangling prerequisite graph; require source coverage; accept a version with conflict detection; regeneration does not overwrite acceptance | M05 |
| T-FR-014 / FR-014 | Durable turns, one active turn/session, duplicate key and stale revision; reconnect resumes; source deletion during generation withholds answer | M05 |
| T-FR-015 / FR-015 | Exercise maps to objective and fixed validated rubric; malformed weights, missing criteria and unsupported answer keys rejected | M06 |
| T-FR-016 / FR-016 | Golden objective scores, alternate valid answers, provisional free response, grading error and correction; unscorable never silently zero | M06 |
| T-FR-017 / FR-017 | Valid stdlib code; timeout/memory/process/output/filesystem/network attacks; no host credentials; infrastructure failure produces no grade | M07 |
| T-FR-018 / FR-018 | Unknown/practice/demonstration/review transitions; duplicate families, assistance, correction and replay; only eligible evidence changes progress | M06 |
| T-FR-019 / FR-019 | Fresh login and interrupted browser recover accepted pathway, evidence and outstanding work; no stale client state overwrites server | M06 |
| T-FR-020 / FR-020 | Explicit model mode and data-sharing consent; invalid capability/binding rejected; zero budget never switches to billable fallback | M02/M05 |
| T-FR-021 / FR-021 | Save/test/revoke/delete dummy key; no retrieval endpoint or key in logs/UI; revoke during queued work prevents new use | M02 |
| T-FR-022 / FR-022 | Concurrent reservations cannot exceed cap; reconcile actual usage/refund unused reservation; unknown provider outcome cannot trigger blind retry | M02/M05 |
| T-FR-023 / FR-023 | Changed source requires approval; failed refresh retains active version; accepted replacement causes explicit dependent-pathway review | M03/M06 |
| T-FR-024 / FR-024 | Delete while fetching/searching/teaching; immediate deny, purged content, safe citation tombstones and fenced late workers | M03/M08 |
| T-FR-025 / FR-025 | Export ownership and completeness; delete account including queued jobs, blobs, credentials and progress; restore reapplies deletion records | M08 |
| T-FR-026 / FR-026 | Worker death before/after commit, duplicate delivery, expired lease and stale fencing token; one publication/evidence effect and bounded retries | M02/M08 |
| T-FR-027 / FR-027 | Report unsupported teaching or disputed grading with permitted references; persist acknowledgement; report itself never marks grade corrected | M05/M06 |
| T-FR-028 / FR-028 | Keyboard and assistive-technology journeys for goal, approval, tutor, assessment and resume; meaningful focus/status/error handling | M08 |
| T-FR-029 / FR-029 | Cross-owner object/job/search/blob/citation/export requests denied, including direct IDs, stale sessions and queue context substitution | Every gate |
| T-FR-030 / FR-030 | Search unavailable/empty leads to manual URL path; same approval, permission and fetch controls apply; no bypass | M03 |

## Nonfunctional acceptance matrix

Targets are proposed engineering gates, not observed properties. Hardware, deployment,
provider, corpus distribution, warm/cold conditions, and workload must be pinned before
testing. Privacy and security tests inspect storage and traces as well as responses.

| Test / requirement | Evidence and proposed pass condition | Gate |
| --- | --- | --- |
| T-NFR-001 / NFR-001 | Threat-model attack cases, dependency review, RLS with non-owner role, session/CSRF/SSRF tests, execution isolation; no unresolved critical/high exploitable finding | M02/M07/M08 |
| T-NFR-002 / NFR-002 | Immediate access denial; active-copy purge <=24 h, backups expire <=30 d, operational logs 14 d/security metadata 30 d; restore cannot resurrect deletion; provider limitations disclosed | M08 |
| T-NFR-003 / NFR-003 | Crash/fault injection across jobs, budget and publication; no duplicate logical effects, silent loss, or partial visible versions | M02/M08 |
| T-NFR-004 / NFR-004 | Proposed 99.5% successful critical-journey probes over rolling 30 days at one-minute intervals; report internal and provider-inclusive availability separately; a short load test cannot establish 30-day attainment | M08/operations |
| T-NFR-005 / NFR-005 | Ordinary API server p95 <=500 ms, retrieval p95 <=2 s; tutor first status <=1 s, complete answer p95 <=20 s with 60 s timeout; supported 100-page text PDF ingest p95 <=120 s including queue | M04/M08 |
| T-NFR-006 / NFR-006 | Envelope below passes limits and correctness; above envelope rejects/queues with bounded resources and understandable status | M08 |
| T-NFR-007 / NFR-007 | Default platform cap $0 fails closed; concurrent usage reservation/reconciliation and adapter token ceilings; no automatic paid fallback; disclose BYOK bill-estimate limits | M02/M08 |
| T-NFR-008 / NFR-008 | Module-boundary checks, typed API contract checks, reviewed migrations and rollback, reproducible documented local setup; no unsupported maintainability score | Every gate |
| T-NFR-009 / NFR-009 | WCAG 2.2 AA target: automated checks plus manual keyboard, zoom, contrast and screen-reader review of critical journeys; documented scope and unresolved failures | M08 |
| T-NFR-010 / NFR-010 | Fixture/index/provider/image/config hashes and commands reproduce deterministic outputs; nondeterministic outputs retain versioned inputs and variability evidence | M04/M06/M08 |
| T-NFR-011 / NFR-011 | Correlate request/job/provider-call IDs, queue age and failure metrics without keys/content; alert injection reaches approved operator destination, redaction verified | M08 |
| T-NFR-012 / NFR-012 | Restore from isolated backup and verify ownership, deletion and checksums; measured RPO <=15 min/RTO <=4 h on approved environment | M08 |

Load envelope: 100 provisioned accounts, 20 concurrent active sessions, 5 workspaces per
account, 20 active sources/workspace, 50 MiB/file, 1,000 pages/source, 500,000 extracted
tokens/source, 100,000 passages/workspace, and 1 million passages/deployment. Exercise
ordinary API at 10 RPS for 30 minutes and 30 RPS for 60 seconds, with 5 concurrent model
turns, 2 ingestion jobs, 2 code runs, and 100 pending jobs. Include skewed ownership and
large-source cases. The source/workspace/deployment caps apply simultaneously; do not
claim their individual maxima can all be multiplied without the deployment cap.

Report achieved throughput, latency distribution, queue age, errors, timeout rate,
CPU/memory/storage, corpus size, and cost. Never compute latency only on successes:
report success latency and all-attempt completion/timeout outcomes side by side.
Fix request mix before execution. Availability denominator is all scheduled
critical-journey probes at one-minute intervals over a rolling 30 days, as specified
by NFR-004. Success requires valid login/session read, knowledge retrieval, and a
tutor outcome other than infrastructure failure; insufficient evidence is a valid
outcome. Record missing probes explicitly without removing them from the denominator;
they are not demonstrated successes. Include planned maintenance and provider outages
in the provider-inclusive figure, and report internal-only availability separately.
A seven-day soak supplies preliminary evidence, not proof of the 30-day objective.

## Human-relevant retrieval and teaching evaluation proposal

M04/M05 need a separately reviewed and human-approved evaluation protocol before
execution. This section specifies required design fields and proposed gates, not an
approved research study. No corpus, model or dataset has been acquired or evaluated.

Use rights-cleared English HTML/text-PDF sources representative of proposed
learning tasks: introductory explanations, procedural tasks, comparisons, multi-section
questions, ambiguous queries, outdated/conflicting sources, and unanswerable questions.
Include broken headings, long sections, tables, refreshes and deletion. Record source
origin, license/permission, acquisition date, checksum, document family, format, language,
topic, structural quality, exclusions and extraction failures. Synthetic fixtures are
useful for invariants but cannot establish quality for real learners or arbitrary domains.

Human authors construct realistic learner queries and objective-specific answer/evidence
rubrics without inspecting candidate-system answers. Use two qualified independent
raters for graded relevance/support on a predeclared subset and all disputed cases;
record raw agreement and adjudication. Model-generated labels alone cannot pass the
human-quality gate. Missing labels, unavailable sources and unscorable questions remain
in a disposition table with counts/reasons; never silently drop system failures.

Define split units as document families (editions, mirrors and near-duplicates grouped)
and learner/task families. Keep paraphrases and dependent question variants in one split.
Create development and locked test partitions before tuning chunking, fusion, prompts
or thresholds. Query labels/answers from test cannot inform prompts, ranking or exercise
generation. Test source documents may be indexed: retrieval requires them; test queries,
answers and relevance judgments remain held out. Document near-duplicate checks and
manual inspection. Record likely public-model pretraining overlap as unresolvable without
provider evidence; do not claim complete contamination exclusion.

Compare full-text-only, dense-only, and proposed structure-aware hybrid with identical
source snapshots, authorization, extraction, candidate/output budgets and query sets.
Define the hierarchy-free hybrid with flat passage construction under the same token
limits, body-only embedding input, and no parent/neighbor context expansion. Compare
it with the full structural configuration: hierarchy-aware semantic chunks,
ancestor-heading prefixes in embedding input, and authorized parent/neighbor expansion.
This contrast estimates the combined configuration change, not any individual mechanism.
Predeclare separate one-change ablations if attributing gains to chunking, heading
prefixes, or expansion: hold the other two settings fixed and name the tested change.
Rebuild and version indexes when chunking or embedding input changes. Use the same
source-span relevance judgments mapped to each configuration's passages and document
that mapping, since changed passage boundaries alter the recall denominator. Preserve
the same evidence-token budget and account for expanded context in answer evaluation.
Claim gains only for the tested contrast. Fix tuning budget on development data and
disclose configurations and failures for every comparison.
Do not label PostgreSQL full-text ranking BM25 or claim reproduction of trained STAIR.

Metrics and denominators must be fixed before evaluation:

- Recall@5: relevant labelled passages retrieved in five results divided by all known
  relevant passages for that answerable query. Report annotation-pool incompleteness;
  supplement with any-sufficient-evidence@5, the fraction of answerable queries with
  at least one retrieved evidence set sufficient for the rubric. A multi-passage
  evidence set must be fully present to count as sufficient.
- nDCG@5: DCG = sum over ranks 1..5 of `(2^relevance - 1) / log2(rank + 1)`, divided
  by the ideal DCG from the labelled pool; relevance uses anchored grades 0/1/2.
  Queries without relevant evidence have undefined nDCG, reported separately rather
  than coerced to zero. Average per query, then report task/source-family distributions.
- Citation validity: accessible, correctly owned, resolvable span references divided
  by all emitted citations. Gate: 100% on acceptance fixtures. Human citation support:
  supported source-dependent claims divided by all emitted source-dependent claims,
  including uncited claims as unsupported. Report claim counts and query-macro averages.
- Answer adequacy: proportion of answerable queries whose answers meet the complete
  objective-specific rubric, not merely a topical match. Proposed gate: >=90%; proposed
  sufficient-evidence@5 gate: >=90%; proposed human claim-support gate: >=95%.
- Abstention sensitivity: correct abstentions divided by unanswerable queries. False
  abstention rate: abstentions divided by answerable queries. Proposed gates: >=95%
  sensitivity and <=10% false abstention. Report both so always-abstain cannot pass.

These numeric quality gates are product proposals, not validated cutoffs. Sample size,
domain mix, minimum important improvement, acceptable noninferiority margins and whether
confidence bounds must clear each threshold require approval before the protocol locks.
Until those are fixed, quality release remains blocked even if a point estimate looks
favorable. Do not choose a sample size by stopping when a favorable result appears.

For paired system comparisons use the same queries and report absolute percentage-point
differences as well as relative change, quality/latency/cost tradeoffs, and failure slices.
The statistical unit is the independent document/task family, not each passage, claim,
paraphrase or repeated model run. Predeclare family-cluster resampling for 95% intervals
on paired differences; preserve pairing and within-family dependencies. If dependencies
cross families, group their connected components or use a justified multiway method.
With too few independent groups, report descriptive ranges and the inferential limit
instead of precise-looking intervals. Repeated generations assess model variability,
not extra independent learners. Predeclare primary endpoints and handle multiplicity
for confirmatory comparisons; exploratory slices remain labelled exploratory.

Report all planned systems and outcomes, exclusions, missingness, uncertainty, and
adverse examples. Evaluate permission/access failures as release-blocking correctness
failures separately from relevance. A gain matters only if its magnitude exceeds the
predeclared practical threshold without violating cost, latency, or safety constraints.
No significance test establishes learning efficacy, fairness, or broad generalization.
Results apply only to the tested language, task/source mix, model versions and deployment.

## Assessment and reproducibility audit

T-FR-016 and T-FR-018 require independently hand-worked golden examples for rubric
weights, thresholds, distinct-family evidence, corrections and replay. For free-response
grading, a future approved audit compares model and human rubric decisions, disagreement,
false demonstration and false failure rates, including language/answer-style variation.
Keep exercise families separate between grader development and evaluation. Any subgroup
comparison needs suitable consent, sampling and uncertainty; do not infer fairness from
synthetic demographic names or missing demographic data. Provisional grades remain
practice-only until this separate gate is met.

Before critical calculations, apply the repository's reproducible-experiment-audit
workflow and verify formulas/splits against an independent reference calculation.
Before reporting major results, apply adversarial-scientific-review. Neither workflow
authorizes experiments or replaces human approval. Pin source and query manifests,
split membership, annotation rubric, adjudications, extraction/index generation,
model/provider identifiers, prompts, seeds, runtime/image/dependency versions, hardware,
commands, raw per-query outputs and aggregation policy under appropriate access controls.
Seeds alone do not guarantee identical provider outputs. Record missing provider-version
information and bound reproducibility claims accordingly. Create accessible figures only
after underlying calculations are verified; preserve their scripts and data references.

## Milestone evidence and stopping rule

M02 establishes identity, persistence, ownership, gateway/budget contracts and durable
jobs; M03 acquisition/provenance/versioning; M04 retrieval evaluation; M05 pathways and
tutoring; M06 assessment/progress; M07 sandbox execution; M08 privacy, operations,
accessibility and release validation. Each milestone requires separate human approval.
Record test ID, revision, environment, input hashes, expected/actual outcome, owner,
reviewer, severity, disposition (`pass`, `revise`, `blocked`) and remaining limitations.
The current milestone checks document consistency and links only; it must not claim
that any of the application or scientific acceptance gates above have passed.

## Conversation regression acceptance

T-FR-004/012/014 must include ambiguous referents requiring a clarification, genuinely
missing evidence requiring abstention, nonstandalone follow-ups, topic changes, and
learner corrections of a wrong prior assistant statement. Assert the response action
and current citation visibility, not exact prose. Earlier assistant output is never
gold evidence. Split whole conversations together, report each stratum separately,
and audit clarification quality independently of answer support. Compare last-turn,
bounded-history and optional rewrite variants under identical corpus and context budgets.

The [supplemental evidence](../research/ibm-rag-additional-evidence.md) motivates a
bounded development-only HPO comparison, not a benchmark execution authorization.
Before M04 experiments, reviewers must fix the objective, search space, trial/resource
ceiling, sampling plan, confidence rules and stop conditions. Include indexing and
judge costs. Freeze the selected configuration before the single confirmatory run;
failed gates require a new versioned protocol and fresh holdout, not repeated tuning
on the same test set. Do not infer platform learning efficacy from retrieval scores.
