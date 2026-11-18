# V1 API contracts

Date: 2026-11-18. Proposed wire contract; implementation/OpenAPI generation belongs
to M02 onward. [SRS](../requirements/SRS.md), [data model](DATA_MODEL.md),
[security](SECURITY_MODEL.md). FastAPI owns business authorization; Next.js is a client.

## Conventions

Base `/api/v1`, same-origin TLS. JSON UTF-8, UUID identifiers, RFC3339 UTC timestamps,
integer revisions. Reject unknown write fields and invalid enum values. Strings have
documented limits: title200 characters, goal4000, ordinary learner message16000,
feedback4000, URL2048; code limit32KiB UTF-8. Lists default20/max100 with opaque
cursor and stable `(created_at,id)` order. Responses never include raw secret values.

Authentication: opaque HttpOnly Secure SameSite=Lax cookie; CSRF header matching
session-bound token plus Origin validation on mutations. Login uses state/nonce/PKCE,
not CSRF header for the OIDC callback. CORS denies other origins. Fresh authentication
<=10min for exports, account deletion and credential mutations. Session idle30min,
absolute7d. Unauthenticated401, authenticated foreign/missing object404 without leaking
existence, known owner permission/state denial403/409 respectively.

All creation/action POST requests except authentication carry `Idempotency-Key` (opaque
16-128 characters). Scope owner+method+route; same body hash resolves the original
operation without executing it again; different body409 `idempotency_conflict`.
Retain content-free operation references and keyed request fingerprints for24h, never
original sensitive response bodies or raw request bodies. Authenticate and authorize
every replay; regenerate the representation from currently visible state. Deleted or
revoked evidence returns safe unavailable/tombstone status, not prior excerpts. A
retrieval replay resolves its restricted evidence manifest without another model call;
stale evidence returns `source_changed` rather than rerunning billable work. Purge
dependent payloads under the deletion workflow, including legacy replay caches.
Domain uniqueness still prevents duplicate attempt settlement after expiry. PATCH/DELETE and actions
changing an existing aggregate require `If-Match: "revision"`; absent428, mismatch412.
Tombstoned deletes are idempotent for the same owner and return204 or original job.
Do not retry a conflict automatically. Rate/capacity limit429 with Retry-After; size413;
validation422; missing capability503. Transient server errors do not mean an accepted
operation was rolled back: client retrieves the prior idempotent operation.

Every mutable aggregate representation includes `revision` and `etag` equal to the
quoted revision, including list items. Single-resource reads and mutation responses
also send `ETag`; composite resume responses provide each aggregate's tag separately.
An immutable content `version` is not a concurrency revision. Header/body preconditions
that name the same aggregate must agree or return422 `conflicting_preconditions`.
Missing required precondition428; stale412; no overwrite. Authenticate/authorize before
precondition checks. An authorized same-key replay resolves the prior operation before
testing its old write revision: it performs no new write and must still recheck visibility.

| Operation | Aggregate checked by If-Match |
| --- | --- |
| Workspace PATCH, goal clarification, pathway proposal/acceptance, tutor-session creation | Workspace; pathway acceptance changes workspace accepted-pathway pointer |
| Candidate decision or acquisition from candidate | Candidate |
| Source version decision, ingestion, refresh, source DELETE | Source (not immutable source-version content); return current source revision with version metadata |
| Tutor turn submission | Tutor session; expected_session_revision must equal header |
| Job cancel/retry | Job; expected_revision must equal header |
| Assessment/run submission | Attempt |
| Credential test/revoke/delete | Credential |
| Session DELETE | Named authentication session |
| Account export/delete | Account, obtained from GET /account |
| Model-binding replacement | Binding; creation instead requires If-None-Match: * |

Other creation-only routes require idempotency but no parent If-Match; they still
validate ownership, current state, supplied immutable versions, quotas and source epoch
transactionally. Retrieval is read-like and has no If-Match. Logout is a special
idempotent revocation of the current authenticated session with CSRF, no revision.
For binding PUT require exactly one creation/replacement precondition; conflicting
headers422, missing428, existing creation target or stale replacement412. A successful
replacement increments binding revision; a missing replacement target returns404.

Error envelope: `{error:{code,message,request_id,retryable,details}}`; details contains
field paths and safe enum reasons, never stack traces, provider bodies or private IDs.
Typical codes: `approval_required`, `permission_unknown`, `source_changed`,
`budget_exhausted`, `provider_outcome_unknown`, `sandbox_unavailable`, `quota_exceeded`.
Business failures of an accepted job appear in its terminal result, not as a misleading
HTTP500 on every poll. Every response includes request ID; no body in access logs.

## Resource representations

| DTO | Required fields / meaning |
| --- | --- |
| Workspace | id,title,status(active/archived/deleting),revision,etag,goal_revision_id,accepted_pathway_version_id nullable,source_epoch,created_at |
| Account / AuthenticationSession | account id,status,revision,etag; session id,revision,etag,created_at,idle_expires_at,absolute_expires_at,revoked_at; no identity/session tokens |
| Candidate | id,revision,etag,url,title,publisher,format,relevance,coverage[],checked_at,publication_date nullable,permission_status(allowed/unknown/denied),permission_evidence,decision |
| Source | id,revision,etag,status,current_version_id,pending_version_ids |
| SourceVersion | id,source_id,source_revision,source_etag,sha256,size_bytes,media_type,acquired_at,permission_status,approval_status,processing_state,revision,etag; no blob key exposed |
| Job | id,revision,etag,kind,state,stage,created_at,updated_at,deadline,cancel_requested,result nullable,error nullable; retry_after_seconds; progress count only where a total is known |
| RetrievalHit | passage_id,source_id,version_id,section_path[],anchor,span(start,end),excerpt,rank; source_epoch,index_generation; score not a confidence probability |
| Pathway | id,version,revision,etag,goal_revision_id,source_epoch,status,objectives[],edges[]; each objective has id,outcome,prerequisites,coverage/gaps,assessment criteria; accept uses workspace tag |
| TutorSession | id,revision,etag,pathway_id,pathway_version,objective_id,active_turn_id nullable,last_turn_ordinal |
| Turn | id,session_id,ordinal,state,text nullable,citations[],source_epoch,usage_status; completed/abstained/failed/cancelled; failed is not an assistant answer |
| Citation | id,claim_ids[],version_id,passage_id,start,end,anchor; excerpt only while visibility remains valid |
| Exercise | id,version,exercise_family_id,objective_ids[],format,prompt,rubric,source_epoch,assistance_policy_version; rubric has criterion IDs, weights, mandatory flags, scoring anchors and version; hidden test content absent |
| Attempt | id,revision,etag,exercise_id,exercise_version,exercise_family_id,pathway_version,assistance_used,assistance_policy_version,status; answer/code only through authorized detail |
| Assessment | id,attempt_id,status(score/provisional/indeterminate),score nullable,rubric_version,criterion_scores[],progress_policy_version,evidence[],feedback,supersedes_id nullable |
| CredentialMetadata | id,revision,etag,provider,label,status,version,created_at,last_test_status; no key suffix/ciphertext |
| ModelBinding | capability,revision,etag,mode,provider,model,credential_id nullable,credential_version nullable,consent_revision,user_cap,status |
| Usage | scope,period,currency,limit,reserved,settled,unknown_reserved,pricing_version; decimal monetary strings |

Fields containing generated material are validated against a versioned schema before
publication. References alone do not authorize content. GET turn/exercise checks visibility
and epoch; stale evidence returns metadata plus `requires_revalidation`, not deleted excerpts.

## Endpoint register

`{w}` is a workspace owned by the authenticated account. All nested IDs are checked
against it. S=sync; A=async returning202 Job and `Location: /api/v1/jobs/{id}`.

| Method/path | Request | Success and important errors | Requirements |
| --- | --- | --- | --- |
| GET /auth/login | allowlisted return_path only |302 OIDC redirect; stores one-use transaction | FR-001 |
| GET /auth/callback | code,state |302 safe local return, session cookie; invalid flow401 | FR-001 |
| GET /session | none |200 user/session metadata and CSRF token, no identity tokens | FR-002 |
| GET /account | none |200 Account with revision/ETag | FR-002,FR-025 |
| POST /auth/logout | CSRF |204 revoke session, expire cookie | FR-002 |
| DELETE /sessions/{id} | fresh owner/session context |204 revoke selected session | FR-002 |
| GET /sessions | none |200 owner session metadata list | FR-002 |
| POST /workspaces | title,goal,experience,time_preference |201 Workspace; quota429 | FR-003 |
| GET /workspaces; GET /workspaces/{w} | cursor for list |200 list or Workspace | FR-003,FR-019 |
| GET /workspaces/{w}/resume | none |200 workspace revision, accepted pathway ID/version, current goal, progress revision, and first paginated page of tutor-session/attempt/pending-job references with independent next cursors | FR-019 |
| GET /workspaces/{w}/pathways; GET /workspaces/{w}/tutor-sessions; GET /workspaces/{w}/attempts; GET /workspaces/{w}/jobs | cursor; jobs state filter optional |200 owner-scoped paginated metadata with IDs/revisions, live visibility checks; includes pending and terminal job discovery | FR-008,FR-013,FR-019,FR-026 |
| GET /workspaces/{w}/tutor-sessions/{id} | none |200 TutorSession with current revision/ETag | FR-014,FR-019 |
| PATCH /workspaces/{w} | title and/or status(active/archived),goal revision input |200 Workspace; stale412 | FR-003 |
| POST /workspaces/{w}/goal-clarifications | answers[],diagnostic_action(start/skip/submit),diagnostic_answers optional | A proposed goal/prerequisite disposition; provider cost gated | FR-004 |
| POST /workspaces/{w}/discoveries | goal_revision_id | A candidates up to5; capability unavailable503/fallback | FR-005,FR-030 |
| GET /workspaces/{w}/candidates | discovery_job_id,cursor |200 Candidate list | FR-005 |
| POST /workspaces/{w}/candidates | manual_url,permission_evidence |201 Candidate; validation422 | FR-030 |
| POST /workspaces/{w}/candidates/{id}/decision | approve/reject,permission_basis |200 Candidate with revision; unknown rights409 | FR-006 |
| POST /workspaces/{w}/sources | approved_candidate_id | A bounded acquisition; returns SourceVersion awaiting exact-hash approval | FR-007 |
| POST /workspaces/{w}/uploads | multipart file,permission_evidence |201 quarantined SourceVersion; 50MiB413; no parsing/model use yet | FR-007,FR-030 |
| GET /workspaces/{w}/sources | cursor |200 source list with active/pending versions | FR-008,FR-023 |
| GET /workspaces/{w}/sources/{s} | none |200 Source with revision/ETag | FR-006,FR-023 |
| GET /workspaces/{w}/sources/{s}/versions/{v} | none |200 metadata/approval state; owner404 | FR-006 |
| GET /workspaces/{w}/sources/{s}/versions/{v}/preview | none |200 acquisition metadata/hash/change status only before approval; no parsed text or rendering | FR-006 |
| POST /workspaces/{w}/sources/{s}/versions/{v}/decision | approve/reject,sha256,permission_basis |200 exact version approval; changed hash409 | FR-006 |
| POST /workspaces/{w}/sources/{s}/versions/{v}/ingestions | pipeline_config_id | A processing/publish; approval missing409 | FR-008..010 |
| POST /workspaces/{w}/sources/{s}/refreshes | candidate_approval_revision | A new acquired version requiring decision; unchanged returns result no-op | FR-023 |
| DELETE /workspaces/{w}/sources/{s} | If-Match | A immediate tombstone then purge | FR-024 |
| GET /jobs/{j} | none |200 Job scoped to owner/workspace | FR-008,FR-026 |
| GET /jobs | cursor,state optional,workspace_id optional |200 owner-scoped job metadata, including account export jobs; workspace filter must be owned | FR-019,FR-025,FR-026 |
| POST /jobs/{j}/cancel | expected_revision |202 Job; already terminal200 | FR-008 |
| POST /jobs/{j}/retry | expected_revision |202 replacement job; ambiguous paid outcome409, unsafe state409 | FR-008,FR-026 |
| POST /workspaces/{w}/retrievals | query,limit(1..20),source_ids optional |200 hits+epoch or explicit empty; deletion race409 | FR-011 |
| GET /workspaces/{w}/citations/{id} | none |200 live authorized span; tombstone410 without content | FR-012,FR-024 |
| POST /workspaces/{w}/pathways | goal_revision_id,source_epoch | A proposed Pathway | FR-013 |
| GET /workspaces/{w}/pathways/{id} | none |200 versioned Pathway | FR-013 |
| POST /workspaces/{w}/pathways/{id}/accept | version,source_epoch |200 accepted version; stale412/409 | FR-013 |
| POST /workspaces/{w}/tutor-sessions | accepted_pathway_id,objective_id |201 session+revision | FR-014 |
| GET /workspaces/{w}/tutor-sessions/{id}/turns | cursor |200 durable turns/stale indicators | FR-014,FR-019 |
| POST /workspaces/{w}/tutor-sessions/{id}/turns | message,expected_session_revision,source_epoch | A turn; active turn409, no silent duplicate | FR-012,FR-014 |
| POST /workspaces/{w}/exercises | objective_id,pathway_version,format | A validated Exercise | FR-015 |
| GET /workspaces/{w}/exercises/{id} | version |200 prompt/rubric, no hidden tests | FR-015 |
| POST /workspaces/{w}/exercises/{id}/attempts | exercise_version,answer or code,submission_id,declared_assistance |201 Attempt; server derives family/policy and combines recorded assistance; duplicate resolves live authorized Attempt | FR-016 |
| POST /workspaces/{w}/attempts/{id}/assessments | rubric_version | A Assessment; coding requires successful trusted run | FR-016 |
| POST /workspaces/{w}/attempts/{id}/runs | image_profile_id from allowlist | A code_run; unavailable503; invalid profile422 | FR-017 |
| GET /workspaces/{w}/attempts/{id} | none |200 attempt/run/assessment status | FR-016..019 |
| GET /workspaces/{w}/progress | pathway_version optional |200 projection+evidence+stale markers | FR-018,FR-019 |
| GET /model-catalog | none |200 supported capabilities, limits, consent terms revision | FR-020 |
| GET /workspaces/{w}/model-bindings/{capability} | none |200 ModelBinding with revision/ETag | FR-020 |
| PUT /workspaces/{w}/model-bindings/{capability} | mode,provider,model,credential_id nullable,consent_revision,user_cap; If-None-Match:* create or If-Match replace |201 created/200 replaced ModelBinding; precondition412, unknown endpoint/model422, incompatible index409 | FR-020 |
| POST /credentials | provider,label,secret |201 CredentialMetadata, fresh auth; no logging | FR-021 |
| GET /credentials; GET /credentials/{id} | cursor for list |200 metadata with revision/ETag | FR-021 |
| POST /credentials/{id}/tests | consent_revision | A test, budget gated | FR-021,FR-022 |
| POST /credentials/{id}/revoke; DELETE /credentials/{id} | If-Match,fresh auth |200 revoked metadata / A purge | FR-021 |
| GET /usage | period(scope allowlist) |200 Usage for owner, not global private usage | FR-022 |
| POST /account/exports | fresh auth | A export; no secrets/disallowed source bytes | FR-025 |
| GET /account/exports/{id}/download | fresh auth |200 authenticated one-use download; expiry410 | FR-025 |
| DELETE /account | If-Match,fresh auth,confirmation=true |202 deletion receipt then sessions revoked | FR-025 |
| POST /workspaces/{w}/feedback | owned turn_id or assessment_id,category,description |201 report ID; foreign404 | FR-027 |
| GET /workspaces/{w}/feedback/{id} | none |200 own report/resolution | FR-027 |

Account deletion receipt uses a narrowly scoped random receipt token stored hashed,
valid24h for status-only polling at `GET /deletion-status/{receipt}`. It exposes only
pending/complete/failed and safe support reference, never content or identity; not
usable as login. User need not retain an authenticated session to observe purge status.
Receipt is excluded from URLs in logs and referrers. If receipt is lost, no unauthenticated
account enumeration endpoint is provided. Operator can investigate by safe support reference.
The content-free receipt record is exempt from account-row cascade deletion until its
24-hour expiry, even if purge completes earlier; it contains token hash, expiry, safe
status and opaque support reference only. The raw token is returned once, never stored
in a replay payload. Ordinary authenticated operation replay cannot recover it after
sessions are revoked. Expired receipts disclose no account existence.

Before exact-version approval, preview is acquisition metadata only: validated media
type, byte count, checksum, acquisition URL/time and checksum change status. No document
parsing, extracted text, PDF rendering, indexing or provider transmission occurs.
Filename/URL labels are escaped untrusted text. Content inspection requiring extraction
starts only after exact-version approval and follows parser/resource controls.

## Example accepted turn and terminal result

```json
{"message":"Why does this loop skip an item?","expected_session_revision":4,"source_epoch":7}
```

202 commits a user turn and job, returning IDs, state `queued`, and polling Location.
A completed result has `{turn_id,state:"completed",text,citations,source_epoch:7}`.
The job may instead complete with an abstention turn, or fail with `source_changed`,
`provider_outcome_unknown` or `deadline_exceeded`. Client polling uses server retry
hint, exponential backoff capped10s and visibility pause; it does not regenerate on reload.
No response body suggests an infrastructure error is a learner error.
Completion, terminal failure, timeout and cancellation each release the session's
active-turn pointer under the current job fence and increment session revision in
the same transaction as terminal status. A stale worker cannot clear a newer turn.

## Internal interfaces and consistency

Acquisition takes candidate/version approval and bounded URL/upload reference, returns
artifact+checksum metadata. Processing takes immutable approved version+pipeline config,
returns staged section/passage manifest. Retrieval takes authorized workspace context,
epoch and query, returns validated spans. Grounding takes claim/citation draft+snapshot,
returns publishable answer or abstention. Tutor orchestrates these services; it cannot
read secret storage. Assessment owns rubrics and progress effects. Gateway alone accepts
model capability requests, resolves consent/budget and returns typed output/usage.

Dispatch admission is serialized with credential revoke/delete and binding replacement
using the same credential/binding row locks. In one short transaction check active
credential and binding revisions, consent, owner state and budgets, and record the
call as dispatched with its admitted revisions. This commit is the dispatch-admission
boundary; network I/O follows outside the transaction. If revoke/replacement commits
first, the old request cannot be admitted; if admission commits first, that admitted
request may still transmit or bill after revocation. Do not imply revocation retracts
already admitted calls. Ambiguous delivery retains reservation and is never blindly retried.

Execution broker submits `{run_id,attempt_id,code_artifact_hash,image_digest,test_spec_hash,
limits,nonce,deadline}` to authenticated isolated host; result contains run/nonce/image
identity, termination reason, bounded outputs and harness observations. The trusted
controller verifies result correlation and grades; guest-supplied exit/score text is not
authority. Host has no application DB/provider key. Tests must be selected outside guest.

Contract tests cover status/error/schema plus permissions, CAS/idempotency races, and
deletion between lookup and response. OpenAPI generated from implementation must match
this register; TypeScript clients generated from that schema. `/api/v1` breaking changes
require explicit version/deprecation plan. Pagination and polling limits are admission
controls, not unlimited extraction paths. Persist source/model/rubric versions for audit.

Required regression cases (future tests, not executed): T-FR-024/T-FR-029 replay a
retrieval after source deletion/revocation and verify no prior excerpt or foreign data;
T-FR-019 clears all browser state and discovers pathway/session/attempt/job IDs from
account/workspace lists and resume; T-FR-006 confirms preview does no extraction before
approval; T-FR-014/T-FR-020 exercise stale tags, missing tags, conflicting header/body
revisions and competing binding creation/replacement; T-FR-021 races dispatch admission
with revoke in both lock orders, proving only previously admitted calls may proceed;
T-FR-025 purges account rows and verifies receipt-only status remains readable until
24h then expires; T-FR-018 verifies server-derived family/assistance/policy provenance
cannot be forged to grant progression. No application tests run in this documentation milestone.
