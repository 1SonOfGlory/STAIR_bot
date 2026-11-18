# Operations and release readiness

Date: 2026-11-17. M01 proposed V1 specification. Owner: M01 RAG/operations author.
Baseline: `f8c967f`. Separate review/human acceptance pending. No service is deployed;
all performance, recovery and staffing targets below remain unverified.

[SRS](../requirements/SRS.md) owns NFR-003–008 and NFR-010–012, with FR-008,
FR-020–026 and FR-029 dependencies. [System architecture](../architecture/SYSTEM_ARCHITECTURE.md),
[security](../architecture/SECURITY_MODEL.md), [RAG](../architecture/RAG_ARCHITECTURE.md),
and [test strategy](../testing/TEST_STRATEGY.md) define adjacent contracts.
The present milestone permits specifications only, with $0 expenditure. Hosting,
provider activation, installation, load tests, deployment and incidents on live
systems require later authorization; these procedures do not grant that authority.

## Deployment boundary and configuration

Deploy a same-origin TLS proxy/UI/API, separate durable worker processes from the
same modular Python codebase, private PostgreSQL/pgvector, and private S3-compatible
storage. Parsing/acquisition use restricted subprocesses; learner code uses a
separate approved Linux KVM host. No execution fallback to the API or worker host.
The proposed topology does not by itself supply high availability, free storage,
24-hour response staffing, or a secure KVM environment.

Before deployment record named service/incident owner and backup contact, hosting
region, data jurisdiction, provider/identity dependencies, capacity, backup/key
management, renewal/patch ownership and funding. Missing operational ownership
blocks release. Pin dependency locks, image digests, migrations, configuration and
adapter/model revisions; store secrets outside images, source and logs. Separate
production from test identities, buckets, keys and databases. Health/readiness
endpoints expose no private identifiers or configuration secrets.

Liveness checks process responsiveness. Readiness checks essential DB/schema and
storage connectivity without expensive or billable calls. Provider/executor health
is a separate capability status: disable affected operations while permitting safe
history/account access. Do not generate billable requests as routine health probes.
Readiness failure removes traffic; it must not repeatedly restart a healthy process
solely because a downstream provider is unavailable.

## Admission, queues and concurrency

The initial envelope is a proposed release test boundary, not measured capacity:

| Resource | Proposed bound |
| --- | --- |
| Accounts / simultaneous active sessions | 100 / 20 |
| Workspaces per account / active sources per workspace | 5 / 20 |
| File / source extraction | 50 MiB; 1,000 pages; 500,000 extracted tokens |
| Passages | 100,000/workspace; 1,000,000/deployment |
| Concurrent work | 5 model turns; 2 ingestion jobs; 2 code runs |
| Pending application jobs | 100 across queued and retry-wait work |
| Ordinary API load | 10 requests/s for 30 minutes; 30 requests/s for 60 seconds |

Reserve quotas transactionally across owners, workers and replicas; process-local
semaphores alone cannot enforce deployment limits. Maintain durable capacity slots
with job/fence ownership and reconcile releases. Enforce byte/storage quotas as well
as record counts. Per-owner fairness and bounded admission prevent a single learner
from occupying every queue slot; rate limits and fair scheduling parameters must be
fixed/tested before release. Deletion/security maintenance needs a reserved control
path so a full learner queue cannot block immediate deny or required purge.

On queue/capacity rejection return a stable overload error and retry guidance; no
202 unless command and job committed. A client retry uses the original idempotency
key. One active tutor turn per session plus expected revision prevents concurrent
tabs from creating inconsistent histories. Tutor polling contributes to API load;
use bounded polling/backoff and stop on terminal status. Queue cancellation is
immediate; running cancellation requests termination and fences future commits.

Workers claim eligible jobs in short transactions with `FOR UPDATE SKIP LOCKED`,
database-time 30-second leases, a new fencing token per claim, and 10-second
heartbeats. Persist job owner, attempts, not-before, lease expiry, step manifests,
input revision and cancellation flag. Never keep a DB transaction open during a
network call, parsing, embedding or execution. Queue semantics follow the
[architecture's PostgreSQL source rationale](../architecture/SYSTEM_ARCHITECTURE.md#durable-work-and-concurrency-fr-008-fr-014-fr-026).

States are queued, running, retry_wait, succeeded, failed and cancelled. Retry only
classified transient failures, at most three retries after the initial attempt,
with persisted delays 1/5/30 seconds; a longer provider Retry-After is respected
within the operation deadline. Validation, authorization, rights and unsupported
format failures are terminal. Exhaustion is visible as failed with a redacted cause,
not an infinite retry loop. New user retry is an explicit operation after rechecking
current rights, state, quota, consent and budget.

Lease expiry permits reclaiming logical work, but does not prove the previous
process/provider call stopped. Fences prevent stale DB publication, not duplicate
external computation. Terminate/reconcile old parser or execution processes before
releasing their physical slots; host watchdogs bound orphan lifetime. A model slot
with ambiguous completion remains occupied or conservatively quarantined until
reconciliation/bounded provider behavior is established. Do not exceed resource or
spending limits by treating lease expiry as proof of capacity. Shutdown drains new
claims, requests cancellation where safe and persists state; crash recovery follows
the same rules without relying on graceful shutdown.

## Recovery and idempotency matrix

| Failure boundary | Required recovery |
| --- | --- |
| Before command/job transaction commits | No accepted job; same-key retry may create it |
| Response lost after transaction commits | Same key/hash returns existing job; changed payload conflicts |
| Object written before DB reference | Staged orphan remains private; reconciler deletes after grace period exceeding the job deadline |
| Parser/embedding stage crashes | Resume verified immutable step outputs; never publish incomplete coverage; clean orphan scratch |
| Lease expires and old worker returns | Reject old fence on all durable effects; reconcile external completion before dispatching again |
| Version publish races refresh/delete | Lock/check expected source revision, current epoch, approval, deletion and fence; losing worker cannot publish |
| Provider timeout/crash after dispatch intent | Mark call `unknown`; retain cost reservation; reconcile, never blind billable retry |
| Tutor output ready before durable commit | Recheck source epoch/session revision, atomically commit result/status; duplicate completion cannot add another turn |
| Assessment/progress projection crashes | Unique evidence-event IDs prevent duplicate credit; replay projection from authorized immutable events |
| Storage reference disappears | Deny affected artifact; alert and recover verified object or mark unavailable; do not invent replacement content |

Provider call accounting states (reserved/dispatched/succeeded/failed/unknown) are
separate from job states. A failed/cancelled job may still contain an unknown call.
Set dispatched before I/O; a crash even before actual transmission is conservatively
ambiguous. Reconciliation uses provider request IDs/idempotency guarantees only when
the selected adapter demonstrably supports them. Otherwise retain or conservatively
settle the maximum reservation with an auditable operator decision, disclose billing
uncertainty and do not resend the ambiguous call. Exactly-once provider execution
is not promised. Durable step uniqueness and DB fencing provide idempotent internal
effects; they do not reverse external charges.

## Measurement and release performance gates

| Journey | Proposed target and measured boundary |
| --- | --- |
| Ordinary API | Server p95 ≤500 ms from request receipt to response; exclude separately classified asynchronous completion, not errors |
| Retrieval | p95 ≤2 s from authorized query admission through query embedding, ranking and evidence manifest |
| Tutor | Accepted/durable status p95 ≤1 s; complete grounded answer p95 ≤20 s including queue/provider/checks; operation timeout 60 s |
| Ingestion | Supported 100-page text PDF p95 ≤120 s from exact-version approval including queue through atomic searchable publication |
| Availability | 99.5% critical-journey probe success over rolling 30 days, at one-minute intervals; provider-inclusive and internal views separate |
| Recovery | RPO ≤15 minutes; RTO ≤4 hours, demonstrated by timed restore |

For NFR-005/006 testing, use the SRS E1 ordinary API mix: 60% workspace/progress/job
reads, 20% non-model mutations and 20% session/metadata requests, with retrieval
separately at two queries/second. Exercise both small and maximum passage scales
and at least two workers. Require <1% infrastructure errors, no isolation/integrity
failures, and predictable 429 overload admission. Freeze a workload manifest: deployment hardware/software,
database/index sizes and owner distribution, source formats/tokens, warm/cold cache,
endpoint mix, arrival schedule, concurrent turn/ingestion/execution counts, provider
adapter and latency distribution, quotas and test duration. Include the envelope
above, simultaneous jobs, retries, polling, disconnects and delete/refresh races.
Do not select an easier input set after seeing latency results. Include completion
rate, queue time, timeouts and rejected load alongside percentiles so dropping slow
requests cannot make a failed service appear fast. Define a sufficient completion
sample count in the approved test plan before interpreting p95.

Provider doubles establish internal correctness and controlled-load behavior only;
they do not substantiate real provider-inclusive latency/availability. Live evidence
requires separately approved access and spending. Fetch/parser maxima are individual
safety limits, not a promise that every combination completes within the 100-page
target. Report infeasible combinations and narrow limits through approved SRS change.

Availability probes cover valid login/session read, retrieval and tutor outcomes
other than infrastructure failure; insufficient-evidence is a valid tutor outcome.
Numerator counts successful probes; denominator includes all scheduled probes,
with provider failures retained in the inclusive view and missing probes reported
without being counted as successes. A seven-day soak is preliminary evidence only,
not proof of the 30-day objective. Real-traffic success is reported separately.
Internal diagnostics may
separate downstream outages, but the user-facing number must not hide them.
Scheduled maintenance remains in the canonical SRS denominator; no exclusions apply.

Profile CPU, peak RSS, DB connections/locks, query plans, disk/IO, index sizes, worker
slots and queue age before optimizing or increasing limits. At one million passages,
raw float32 vectors alone require `1,000,000 × dimension × 4` bytes, excluding rows,
indexes, replicas and backups; dimension must be selected before sizing. At all source
count/file maxima, 10,000 × 50 MiB is about 488 GiB of raw files alone. These arithmetic
bounds are not measured storage or workload estimates. Shadow generations and staged
refresh temporarily duplicate data. Two parser limits permit up to 2 GiB parser memory
alone; two 256 MiB guests require 512 MiB guest memory plus host/hypervisor overhead.
Account for these costs before admitting full maxima or selecting hosting.

## Cost and provider controls

Current authorized execution spend is $0. Future deployment daily cap D defaults to
$0; dispatch fails closed until numeric limits and provider capacity are approved.
Open-source components do not remove hosting, storage, egress, backup, identity,
search, embedding, monitoring or operator costs. No automatic paid fallback.

For each provider call reserve conservative input/output cost using a versioned
price record and output cap, atomically against relevant owner/provider/platform
ledgers. Embedding batches, discovery, credential tests, support checks and repair
attempts also require accounting where billable. Unknown price or unsupported usage
limits blocks dispatch. Settle actual usage once; keep unknown reservations intact.
Compare billed usage where available with ledger totals and surface discrepancies.
Currency/timezone/budget reset semantics must be pinned in the gateway contract;
use UTC accounting windows and do not reset pending reservations at midnight.

BYOK requires explicit data/billing consent, capability-specific bindings and local
reservation caps. A local cap cannot guarantee a provider-wide bill, especially for
requests made elsewhere. Revocation blocks new dispatch admission; previously admitted requests may still transmit, complete or bill. Admission and revocation serialize using the binding and credential locks defined in SYSTEM_ARCHITECTURE; network I/O follows commit. Binding changes do not silently change embedding generations.

## Telemetry and incident response

Emit structured redacted events, bounded-label metrics and linked traces for admission,
queue claim, stage execution, gateway dispatch, publication and cleanup. Correlation
IDs link a request/job/call without recording raw prompts, answers, documents, code,
keys, cookies or full URLs. Keep object IDs in access-controlled event records, not
unbounded metric labels. Record duration, stage/error category, attempt, generation,
token counts, cost estimate/settlement and resource utilization where available.
User content and credential endpoints disable body capture across proxy/APM/crash
reporting as well as application logs. Retention: redacted logs 14 days, security
metadata 30 days. Verify expiry and synthetic-secret/content canaries before release.

All failed jobs and security/budget events emit metadata; other traces may be sampled.
A synthetic incident must reach the responsible operator within five minutes.
Alert on isolation/secret exposure, sandbox escape signals, deletion overdue at 24 h,
backup/ledger failures, unknown provider calls, reservation discrepancies, missing
published objects, worker lease churn, old queue items, exhaustion and latency/error
budget breaches. Numeric alert thresholds and accountable responders must be fixed
in release configuration; an unstaffed alert is not an incident response capability.

On a suspected incident: isolate the affected capability or tenant path, stop new
provider dispatch if credentials/cost are implicated, revoke sessions/keys where
needed, preserve minimal security metadata, establish scope with verified evidence,
and restore only after the responsible operator and reviewer approve. Record times,
affected revisions, containment, unresolved exposure and corrective actions. Avoid
copying private data into tickets. Notifications depend on confirmed scope and
applicable obligations; no universal legal deadline is asserted here.

## Deletion, backup and restoration

Deletion first denies access transactionally, increments source epoch, revokes affected
sessions/credentials and fences jobs; purge active copies within 24 hours. Inventory
DB rows, vectors, blobs/versions, staging, caches, scratch, exports and generated
derivatives. Verify inventory before declaring complete; failed purge remains visible
and retried idempotently. Export staging expires after download or 24 hours, whichever
comes first. Citation tombstones contain opaque IDs only, never source content.

Encrypted backups and every recoverable replica/snapshot/object version/WAL tier
expire within 30 days of creation. Keep an independently durable content-free deletion
and revocation ledger; restoration must not roll that ledger back with the database.
Maintain recoverable coverage no older than the retained ledger. At expiry, remove
recoverable snapshots before corresponding tombstones. Storage-provider deletion
behavior, key recovery and expiry must be verified; this is not proof of external
provider erasure or recall of learner-downloaded exports.

Restore procedure: provision isolated environment; select eligible consistent DB and
object artifacts; verify checksums/schema/key availability; replay the latest deletion
and credential-revocation ledger; invalidate old sessions; reconcile missing/staged
objects, jobs, fences, capacity slots and unknown calls; verify ownership and deleted
content absence; run read/write smoke checks; then authorize traffic. Never rerun
unknown provider requests during replay. Missing ledger, irreconcilable ownership,
or expired backup fails restoration closed. Measure data loss from latest durable
recoverable business transaction to incident and elapsed time until safe traffic,
including replay/verification, for RPO/RTO evidence. Test the oldest permitted backup
as well as a recent one. Current staffing/hosting does not establish either target.

## Change control and acceptance

Use reviewed migrations with backup/restore evidence. Prefer compatible expand/migrate/
contract steps; mixed API/worker revisions must agree on job/schema versions before
rolling deployment. Drain incompatible workers. Rollback restores a compatible app
revision and pointers only after verifying deletion/consent state; destructive schema
rollback is not assumed safe. A failed migration leaves readiness off until recovery.
Index changes require shadow-build space and RAG validation. No production fixtures
or secrets enter development artifacts.

| Canonical test | Future acceptance evidence |
| --- | --- |
| T-FR-008 / T-FR-026 / T-NFR-003 | 100 repeated kill/retry schedules and duplicate delivery at durable boundaries; stale fences rejected, effects deduplicated, bounded terminal failure |
| T-FR-022 / T-NFR-007 | Concurrent budget admissions, midnight pending reservations, unknown outcomes, retries and revocation cannot bypass local caps |
| T-FR-024 / T-FR-025 / T-NFR-002 | Queue saturation and worker crash do not defeat immediate deny or 24-hour purge; no retained content in tombstones |
| T-NFR-004 / T-NFR-005 / T-NFR-006 | Pinned load manifest, full completion/error distributions and measured resources meet declared targets; provider scope explicit |
| T-NFR-008 / T-NFR-010 | Fresh environment can follow pinned build/migration/run records; compatibility and rollback evidence retained |
| T-NFR-011 | Telemetry canaries absent; actionable metadata and retention expiry verified |
| T-NFR-012 | Timed isolated restore meets RPO/RTO, replays deletion first and cannot resurrect credentials/content |

M02 establishes transactions/configuration; M03 durable ingestion; M04 retrieval
measurement; M05–06 durable tutoring/progress; M07 execution host controls; M08
combined load, privacy, recovery, accessibility and operational readiness. Each needs
human approval. No tests in this table have been executed in M01. Release remains
blocked without hosting/funding, operators, provider terms/capacity, tested isolation,
backup/deletion proof and independent review of substantive implementation evidence.
