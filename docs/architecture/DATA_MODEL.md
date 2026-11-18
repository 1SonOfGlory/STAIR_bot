# Data model and lifecycle

Date: 2026-11-18. Proposed logical model; no migrations implemented.
[SRS](../requirements/SRS.md) is normative; [API](API_CONTRACTS.md) defines representations.
PostgreSQL is the proposed authority. Object storage holds bytes, not authorization.

## Ownership and common conventions

Use UUID primary keys, UTC timestamptz, created_at/updated_at and integer revision for
mutable aggregates. Owned tables carry user_id; workspace data also carries workspace_id.
Composite unique keys `(user_id, workspace_id, id)` and matching foreign keys prohibit
cross-owner/cross-workspace references even if an application filter is omitted.
The authenticated principal is transaction-local DB context for RLS. Application roles
are neither owner, superuser nor BYPASSRLS; migration/admin access is separate and audited.
Missing context always denies access. Session/credential/account-only records
use user-level policies. Public model catalogs contain no private data.

Ownership is individual in V1. Do not add nullable organization IDs or speculative roles.
Use a workspace authorization service so future membership can be added by deliberate
migration rather than rewriting every feature. No global cross-tenant content deduplication
or shared embeddings in V1; content hashes are scoped to owner/workspace.

## Logical entities

| Aggregate / tables | Essential fields and relationships | Constraints / writer |
| --- | --- | --- |
| users, identities | user status/email; identity issuer, subject, user_id | Unique issuer+subject; no email-only linking; Identity module |
| sessions, login_transactions | hash of opaque token, user_id, idle/absolute expiry, auth_time, revoked_at; state/nonce/PKCE temporary records | No raw session token in DB/log; consumed login state cannot replay; Identity |
| workspaces, goal_revisions | owner, title, status, revision, source_epoch, current_goal_id, accepted_pathway_version_id; goal text, outcomes, experience, time preference, prerequisite dispositions | Goal immutable; workspace pointers update with revision CAS; Workspaces |
| source_candidates | discovery_job_id, URL/title/publisher, relevance/coverage, checked_at, publication_date nullable, permission basis/status, decision | Candidate approval authorizes acquisition only; Sources |
| sources | workspace, candidate_id nullable, status, current_version_id nullable, revision | At most one active version; candidate/upload provenance retained; Sources |
| source_versions | source_id, SHA-256, object_id, media type, size, permission evidence, acquired_at, approval_id, processing_state, pipeline_version | Immutable bytes, unique source+hash; permission revocation is separate mutable visibility state; Sources |
| approvals | actor, source/candidate reference, hash nullable only for candidate approval, permission snapshot, decision, decided_at, revision | Hash mandatory for publish; withdrawal recorded as event; Sources |
| artifacts | opaque object key, owner/workspace, checksum, size, purpose, staging/live/tombstone state, expiry | Private bucket; object ownership checked via DB, never inferred from key; owning module |
| sections | version_id, parent_id nullable, ordinal, original title, inferred flag, page/HTML anchor, content offsets | Same-version parent, acyclic tree, unique version+ordinal path; retain internal-node body; Knowledge |
| passages | section_id/version_id, ordinal, text artifact/span, token_count, text_checksum, lexical vector | Exact offsets in normalized text plus mapping to original; unique version+section+ordinal; Knowledge |
| index_generations, embeddings | workspace, model/provider/dimension/normalization/pipeline hash, state; generation+passage+vector | Unique generation+passage, fixed dimension per physical vector index; atomic active generation pointer; Knowledge |
| jobs, job_steps | kind, aggregate ID, actor/workspace, state, attempts, deadline, not_before, lease/token, cancel_requested, error code; step input/output hashes | Unique operation idempotency scope; step effects unique by job+step+input; Jobs |
| operation_replays | owner/method/route/key scope, keyed request fingerprint, operation ID/type, created_at, expires_at | Unique scope; 24h content-free references only, no raw requests/responses or secrets; owning command module |
| pathways, pathway_versions, objectives, prerequisite_edges | workspace, accepted version; goal/source epoch, ordered objective content/coverage; prerequisite IDs | Immutable proposal version, acyclic same-version graph; Learning |
| tutor_sessions, turns, citations | session pathway/version, revision, active_turn; turn role/text/state, request id, epoch, provider_call; citation version+passage+offsets+claim IDs | One active turn/session, ordinal unique; completed answer/citations one transaction; Tutoring |
| exercise_families, exercise_versions, rubrics | family ID and provenance; objective refs, source epoch, prompt, format, family_id, rubric/test artifact hashes, generator version, assistance_policy_version, status; criterion IDs, weights, mandatory flags and scoring anchors | Same-workspace family reused by paraphrases/retries; immutable released exercise/rubric; assessment service alone assigns family/scoring rules; Assessment |
| attempts, assessments, code_runs | exercise/pathway version, family_id, submission artifact, revision, declared_assistance, recorded_assistance, assistance_used, assistance_policy_version; assessment score/status/rubric/model provenance, criterion_scores, progress_policy_version, supersedes ID; run image/limits/test hash/result | Unique submission key; family/policy copied from trusted exercise, assistance cannot be lowered by client; no grade from untrusted guest; Assessment/Execution |
| progress_policies, progress_events, progress_projections | immutable policy version/hash and thresholds, family count, freshness and assistance eligibility; event workspace_sequence, objective/attempt/assessment ID, family_id, assistance_used, rubric/policy versions, criterion outcomes, type, evidence epoch, superseded_event_id nullable; projection revision/status/evidence IDs/policy version | Unique assessment effect and workspace sequence; append-only except privacy purge; projection rebuildable under pinned policy; Learning |
| credentials, provider_bindings | user/provider/label, encrypted key/wrapped data key, state/version; capability/model/mode/credential ref/consent revision/caps | Never export ciphertext or return key; binding must use same owner; Gateway |
| budget_ledgers, usage_reservations, provider_calls | scope/date/currency/cap/spent/reserved; maximum estimate/price version; intent key/state/provider request ID/actual usage/error | Decimal currency arithmetic; unique call settlement; consistent lock order deployment then owner then credential; Gateway |
| feedback_reports | owner, owned turn/assessment, category, optional text, state/resolution | No automatic grade change or outbound transmission; Learning |
| exports, deletion_requests, deletion_ledger | requester/scope/state/job/artifact/expiry; opaque deleted IDs, revision, timestamp | Ledger separately durable for restore suppression; no content in tombstones; Privacy |
| deletion_receipts | token_hash, expires_at, pending/complete/failed status, opaque support reference | No user/content fields or cascading user FK; survives account purge until 24h expiry; not an authentication session; Privacy |
| audit_events | event type, pseudonymous actor/object IDs, time, outcome, trace ID | Restricted append, bounded retention; no source bodies/prompts/keys; Security |

Job input payloads contain typed identifiers and versions, not credentials or arbitrary
commands. Sensitive learner text stays in its owning table/artifact. Structured JSON
is acceptable for provider-specific metadata and versioned rubric schemas, not ownership
keys or relationships that require referential integrity.

Mutable aggregates expose integer revision/ETag through both list/detail representations.
Source-version approval and ingestion compare/increment the source revision; pathway
acceptance compares/increments workspace revision; turn admission compares/increments
tutor-session revision. Immutable content versions remain separate. API defines the
remaining action-to-aggregate mapping. Workspace resume reads accepted-pathway pointer
and authorized paginated tutor-session, attempt and job indexes; it requires no
client-retained IDs. Source epochs and permissions are checked on each returned item.

## State transitions and transaction boundaries

Source: candidate proposed -> approved/rejected; approved acquisition -> awaiting_version_approval
-> approved_version -> processing -> ready, or failed/cancelled. A refresh is a separate
version following the same path. The currently ready version remains active until
atomic replacement. Rejecting a refresh does not remove old content; deleting/revoking
the source does. Uploads first enter quarantine and require exact-hash approval.

Publication transaction checks source revision, job fencing token, owner state, permission,
approval hash, object checksums and reserved passage/storage quota; installs active version
and generation-compatible embeddings; increments source_epoch. No external call occurs
inside this transaction. A late refresh fails CAS and cannot overwrite a newer version.

Job states: queued/running/retry_wait/succeeded/failed/cancelled. Lease uses database time,
30s expiry, heartbeat every10s, monotonic fencing token. At most three retries after the
initial attempt (1/5/30s minimum delays), only for known safe transient failures. Unknown
provider dispatch is a provider_call state, not an invitation to rerun a job blindly.

Call: reserved -> dispatched -> succeeded/failed/unknown. Persist intent and reservation
before network I/O; dispatched-but-no-result retains full reservation. Reconcile via
provider request ID if supported, else conservative settlement/manual investigation.
Cancellation cannot retract a transmitted call. Use fencing and unique settlement IDs
to stop duplicate local charging; exactly-once external execution is not claimed.

Persist binding revision, credential version/revision and consent revision on each
admitted call. Canonical lock order is budget ledgers (deployment, owner, credential),
then binding rows, then credential rows; multiple rows of each class use stable ID order.
Operations taking a subset preserve this order. Revoke/delete/replacement take the
same affected binding and credential locks as admission. Binding creation also locks
the referenced credential and rechecks its active state before committing, so a new
binding cannot reactivate a revoked credential. Replacement rechecks the expected
binding revision and increments it. Dispatch checks active states and
revisions under these locks and commits `dispatched` plus reservation before network
I/O. Revocation/replacement that wins this serialization blocks old admission; an
already admitted call may subsequently transmit/bill. No transaction remains open
during network I/O. Unknown dispatch retains its reservation after job termination.

Turn acceptance locks the session row and checks revision/active turn. Commit user turn
and job together; completion commits assistant content/citations and releases active turn.
Terminal failure, cancellation and timeout also clear active_turn and increment session
revision atomically with terminal job/turn status, conditional on current fence and
matching turn ID. A stale worker cannot clear a subsequent turn's admission slot.
Deleted or source-epoch-stale evidence cannot commit. At read time suppress a completed
answer whose referenced source visibility changed until it is safely revalidated; never
serve cached deleted excerpts. Operation replay references persist for24h with keyed
request fingerprints, never original sensitive bodies. Replay reauthorizes and resolves
live visibility from the existing operation without repeating its effects/provider
calls. Stale retrieval manifests return unavailable rather than new billable retrieval.
Delete dependent result payloads and legacy response caches; account deletion purges
owner replay references. Business uniqueness persists beyond the replay window.

Assessment/progress commit is one transaction keyed by assessment ID. Retrying a completed
grading job cannot append another progress increment. Regrade appends a superseding
assessment and compensating evidence transition; old provenance remains until deletion.
The trusted engine derives exercise family from content lineage and records assistance
from system events plus learner declaration; unknown assistance remains explicit and
is ineligible until resolved under the pinned policy. This does not establish absence
of external assistance. Paraphrases/retries share a family.
Mandatory criterion outcomes, rubric version and immutable progress-policy version
travel with each assessment/event so projection replay can enforce all criteria and
distinct-family thresholds. Clients cannot submit authoritative family, criterion
scores, eligibility or policy versions. Corrected assessments preserve supersession.

## Indexes and storage growth

Index owner/workspace+updated_at for listings, source active pointers, job state/not_before,
lease expiry, unique idempotency scopes, session active turn, attempt identity, and retention
expiry. Use PostgreSQL FTS for lexical search (not labelled BM25) and pgvector exact
search initially as a correctness reference; ANN is enabled only after filtered-recall
and latency evidence. Filter ownership/active-version before selecting candidates and
recheck after ranking. Benchmark query plans with representative tenant skew.

Each embedding generation pins model/dimension. Changing embedding model builds a new
generation in staging, checks completeness, then switches a pointer atomically; old
generation remains until no in-flight references require it, bounded by deletion policy.
Do not compare vectors from different models. Dimension changes require separate index
storage/migration, not coercing old rows into new vectors.

SRS quotas bound admission, not sufficient disk planning. Compute raw, extracted,
embedding, index, replica and backup bytes from measured fixtures before deployment;
reserve global capacity atomically. Orphan artifact reconciliation removes staged objects
after a grace period beyond job deadlines; checksum audit detects missing live objects.

## Deletion, export and migrations

Deletion commits denial and epoch/fence changes first, then durable purge covers blobs,
versions, passages/vectors, derived answers, exercises and evidence containing removed
content, caches, exports and scratch. Source citation tombstones retain only opaque IDs.
This inventory includes operation results/retrieval manifests and any cached replay
payload. Denial and live replay checks apply immediately, before asynchronous purge.
Account deletion purges all personal records; content-free restore suppression IDs remain
30d, after which every recoverable backup containing them must already have expired.
Operational logs14d/security metadata30d. Export omits secrets and third-party bytes
without redistribution permission. A generated answer is not exempt from source deletion.
The content-free deletion receipt is a narrowly scoped exception: retain its token hash,
safe status, opaque support reference and expiry until24h after issuance, independent
of account cascade deletion. Never retain the raw token, user identity, job payload or
content in it. Status can therefore become complete after user rows disappear. Expire
the receipt on schedule; the separate30d restore-suppression ledger is not a public
status endpoint. Lost receipt tokens cannot be recovered by unauthenticated lookup.

Deletion ledger durability must be independent of the restored backup; replay deletion
and credential revocation before serving. Unavailable ledger means restore remains closed.
RPO/RTO and purge timing are proposed targets in SRS. External provider erasure is outside
local guarantees. Migration strategy: expand/backfill/verify/contract, compatibility with
current and previous app build during rollout, explicit rollback or forward repair for
destructive changes. Pin schema versions in exports and evaluation manifests.
