# Security and privacy model

Date: 2026-11-17. M01 author: safety-researcher specialist. Status: proposed V1
specification; separate review pending. Base: `f8c967f`. This is a design, not an
implemented control, security certification, or completed penetration test.

Canonical requirements and test IDs are in [SRS](../requirements/SRS.md).
This document elaborates FR-001–002, FR-006–012, FR-017, FR-020–026, FR-029 and
NFR-001–002, NFR-007, NFR-011–012. The [charter](../charter.md) establishes the
product boundary. M01 authorizes specifications only; implementing or validating
these controls needs the corresponding approved milestone.

## Assets, actors, and trust boundaries

Protect identity/session bindings, API credentials, private learning goals and
conversations, submitted materials/code, assessment evidence, provider budgets,
source approval/version records, and the integrity of explanations and citations.
Vectors and extracted text are private derivatives, not anonymous data.

V1 has individual workspace owners; sharing and institution administrators are
excluded. A learner can be malicious. Source publishers, uploaded documents,
retrieved passages, model output, exercise code, browser parameters, and job payloads
are untrusted. Operators have exceptional infrastructure powers; their access is
audited and cannot be described as cryptographically impossible. Identity, search,
model, hosting, and backup providers are separate trust domains.

| Boundary / plausible abuse | Required control | Remaining exposure |
| --- | --- | --- |
| Browser → API: stolen cookie, CSRF, guessed object IDs | Server session, CSRF checks, ownership authorization on every operation | Compromised browser or active XSS can act as learner |
| API/workers → data: cross-workspace joins or confused job ownership | RLS, composite ownership keys, explicit worker scope, negative tests | Privileged DB operator can bypass normal application controls |
| Public URL/upload → acquisition/parser: SSRF, parser exploit, resource exhaustion | Egress filtering, pinned destination, bounded isolated parsing | Parser/kernel vulnerabilities need patching and operational response |
| Source/model → tutor: injection, false citations, hostile links | No model authority, source-span checks, safe rendering | Grounding checks do not prove truth or eliminate persuasive misinformation |
| Gateway → providers: key leakage, excess spending, data retention | Fixed adapters, scoped credentials, consent, atomic reservations | Provider behavior/billing outside local control |
| Learner code → executor: escape, fork bomb, persistence | Dedicated microVM host, no guest network, resource limits, teardown | Hypervisor/host compromise remains possible |
| Backup → restore: deleted content resurrection | Deletion ledger replay before serving, retention verification | Externally held copies cannot be recalled by application deletion |

Source publishers also have acquisition rights; third parties may appear in uploads.
Operators face incident and cost exposure. Learners can suffer privacy loss, incorrect
feedback, or loss of access. These are plausible consequences, not measured harm rates.

## Identity, sessions, and ownership

FR-001 uses an allowlisted external OIDC issuer, authorization code flow with PKCE
S256, exact configured redirect URIs, single-use state and nonce bound to the login
transaction, and server-side token exchange. Validate signature, allowed algorithm,
issuer, audience/authorized party, expiration, and nonce before creating a session.
Require verified email but key identity by `(issuer, subject)`; never merge accounts
on matching email alone. No local passwords or automatic provider-account linking.
OIDC token validation and nonce semantics follow [OIDC Core, sections 2 and 3.1.3.7](https://openid.net/specs/openid-connect-core-1_0.html#IDTokenValidation);
the flow selection follows [RFC 9700, section 2.1](https://www.rfc-editor.org/rfc/rfc9700.html#section-2.1).

FastAPI owns opaque, cryptographically random server sessions. Store a hash of the
session secret; issue a host-only `__Host-` cookie with `Secure`, `HttpOnly`,
`SameSite=Lax`, and `Path=/`. Rotate after login/re-authentication. Proposed expiry:
30 minutes idle and 7 days absolute; enforce both server-side. Logout invalidates
the session before clearing the cookie. Account deletion invalidates all sessions.
Credential mutations (save/replacement/revoke/delete), export, and account deletion require authentication within
the last 10 minutes. These timeouts are design choices requiring usability review,
not values mandated by the cited standards.

All authenticated mutations require a session-bound CSRF token plus same-origin
Origin validation (or strict Referer fallback); reject missing or foreign evidence.
OIDC callback is a narrowly exempt endpoint protected by its login transaction.
GET/HEAD do not mutate user data. UI/API share an origin; no wildcard credentialed
CORS. Configure trusted reverse proxies explicitly. Never place session/provider
tokens in local storage, URLs, telemetry, or browser-readable application state.
These controls draw on [OWASP Session Management, cookie and lifecycle sections](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html).

FR-029 requires authorization at API admission, worker execution, retrieval, result
commit, and result delivery. Resolve ownership from the authenticated principal,
never a supplied `user_id`. Object existence must not leak through status/error
differences across owners. UUID unpredictability is not authorization.

Every workspace-owned row carries `workspace_id`; composite foreign keys prevent
cross-workspace references. RLS runs under a non-owner, non-superuser role without
`BYPASSRLS`, using transaction-local authenticated user context reset by transaction
end. Migration/administration roles are separate; `FORCE ROW LEVEL SECURITY` is
defense in depth where applicable, not protection against superusers. Missing
context denies access. Queue maintenance uses a separately scoped role; workers
derive owner/workspace from persisted records, then establish a bounded context.
Do not trust a job's embedded owner assertion. RLS exceptions are described in
[PostgreSQL row security documentation](https://www.postgresql.org/docs/current/ddl-rowsecurity.html).

Retrieval filters ownership and active source/version before ranking and again
before returning citations. Cache keys include owner, workspace, source epoch,
index generation, and authorization-sensitive revision; no shared answer cache.
Private blob buckets disallow public access. Serve downloads through authorization
checks, not reusable public object URLs. Export readiness does not bypass ownership.

Idempotency replay also rechecks current ownership, deletion state and source visibility.
Store content-free operation references and regenerate authorized representations;
never replay stored excerpts after their source is deleted or revoked. Any retained
sensitive replay payload is included in immediate denial and the 24-hour purge inventory.
Replay does not rerun a completed operation or initiate a new provider call.

## Safe acquisition and parsing

FR-006–010 bind approval to a source/version and recorded permission evidence.
Approval is not a license. A changed source requires review under FR-023; redirect
to a materially different source cannot silently inherit approval. Record canonical
URL, fetched URL, checksum, time, format, and permission basis. Reject unknown
acquisition rights; offer user-supplied permitted material. V1 excludes authenticated
website crawling, browser automation, DRM bypass, and arbitrary recursive crawling.

The acquisition service accepts only HTTP(S) URLs without embedded credentials;
restrict ports to 80/443, reject local file and other schemes, and never inherit
ambient proxy credentials. Parse with a single standard URL parser, canonicalize
host/IP forms, and reject malformed/ambiguous forms. Resolve all A/AAAA answers;
reject non-public destinations including loopback, private, link-local, multicast,
unspecified, reserved, IPv4-mapped bypasses, and cloud metadata endpoints. Pin the
validated destination for the connection while retaining proper TLS hostname/SNI
verification. Revalidate every connection/retry and redirect; never automatically
follow unchecked redirects. Allow at most five redirects. Network egress rules
independently block internal services/metadata, including DNS rebinding outcomes.
These measures implement the layered validation approach in
[OWASP SSRF Prevention, application and network controls](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

Do not forward cookies, authorization headers, or user identifiers to source sites.
Search receives only a disclosed, minimized goal query; manual URL fallback avoids
requiring external discovery. Remote requests reveal server IP and requested URL to
the destination; disclose this boundary rather than promising anonymous acquisition.

Proposed fetch deadlines: 10 seconds to connect and 60 seconds total, including
redirects; enforce streaming compressed and decoded size limits of 50 MiB. Parsing
accepts only supported text PDF/HTML after MIME and content checks, not filename
alone. Reject archives, encrypted PDFs, executable formats, embedded scripts,
unsupported active content, and unsupported encodings explicitly. HTML extraction
does not execute JavaScript or fetch images/subresources. Do not invoke shell strings,
unsafe object deserialization, or document-supplied plugin commands.

Acquisition and parsing use separate restricted subprocesses; parser has no network,
DB credentials, provider keys, or host filesystem access beyond its input and private
scratch directory. Proposed per-document parser limits: 1 GiB memory, 60 CPU seconds,
90 seconds wall time, 16 processes, 100 MiB scratch, with process-tree termination.
Additionally cap 1,000 pages and 500,000 extracted tokens. Parent enforces limits even
if parser crashes; reject/truncate nothing silently. These supplemental limits need
feasibility verification against the SRS ingestion latency target before activation.
Delete scratch on completion/failure; startup reaper handles abandoned directories.
Publish immutable checked output only after a complete successful transaction.

## Untrusted content, explanations, and rendering

Source and model text never grant permissions, select credential IDs, override
budgets, run code, issue arbitrary HTTP requests, or change system policy. Separate
instruction and source fields, label provenance, and validate structured outputs.
Prompt-injection detection can quarantine/report suspicious sources but is not the
authorization boundary. The model has no direct DB, filesystem, credential, or
network tool. Any proposed action passes typed deterministic application checks and
the learner consent required by its feature. Persistent memory/pathways are subject
to the same rule. This follows the defense-in-depth posture of
[OWASP LLM Prompt Injection Prevention](https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html).

FR-012 binds citations to authorized active version spans; nonexistent anchors,
deleted versions, and unsupported answers fail closed to abstention or revision.
Recheck source epoch before turn commit and again on rendering. A citation proves
traceable attribution only; it does not establish source truth, causal learning
benefit, or faithfulness of a model's internal reasoning. Show concise source-based
rationale rather than claiming access to internal reasoning. A report mechanism
lets learners contest incorrect teaching/assessment without surrendering credentials.

Render sanitized Markdown with raw HTML disabled; escape code/output, prohibit
script/data/javascript URL schemes and embedded frames, disable remote images, and
use a restrictive CSP without arbitrary inline script. Treat source links as external
navigation, not server-side fetch permission. Redact sensitive URL query parameters
from logs. Do not put raw retrieved material in exception pages or analytics.

## Provider gateway and BYOK lifecycle

FR-020–022 use a fixed provider/endpoint adapter allowlist; users cannot configure
arbitrary base URLs. Provider choices distinguish chat and embedding capabilities.
Before first transmission or provider change, show which provider receives which
content, whether platform or learner is billed, retention terms, and local caps.
Sending document text for embeddings needs the same disclosure as chat prompts.
No silent paid/provider fallback; lack of consent or approved capacity blocks dispatch.

Credential lifecycle:

1. Accept a key once over TLS into a dedicated write-only endpoint. Disable body
   capture in reverse proxy, application, tracing, error reporting, and gateway logs.
2. Encrypt using authenticated envelope encryption with per-credential data keys
   wrapped by a deployment key held separately from the database/backups. Bind
   credential ID, owner, and provider as authenticated context. Select the concrete
   supported key-management mechanism before release; do not invent cryptography.
3. Return only key ID, provider, user label, timestamps, and status. No plaintext
   retrieval endpoint, secret-derived preview, or export of encrypted/plaintext keys.
4. Test only through the fixed provider adapter. Disclose if testing is billable and
   reserve budget first; never imply a free test when provider terms are unknown.
5. Gateway resolves the owner's active binding and decrypts just before dispatch;
   minimize plaintext lifetime. No key in jobs, prompts, code runners, or parser
   environment. Python process memory cannot promise provable zeroization.
6. Replacement creates a new credential version and revokes local use of the old
   version atomically. Revocation/deletion blocks new dispatch admission immediately;
   queued work rechecks the version. A previously admitted request may still transmit,
   finish or bill after revocation; cancellation of that request is best effort.
7. Purge stored ciphertext and wrapped data keys under the deletion schedule.
   Local revocation does not revoke the provider's key: direct the learner to the
   provider for upstream revocation, especially after suspected compromise.

Encryption does not protect keys from a compromised authorized gateway or an
operator holding both ciphertext and wrapping access. Separate duties and audit
decrypt/rotation/revocation metadata. Backups retain only encrypted secrets until
expiry; do not claim immediate cryptographic erasure while recoverable wrapped keys
exist. Lifecycle controls are informed by
[OWASP Secrets Management, lifecycle and backup sections](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html).

Budget reservation is atomic across concurrent calls and retries, scoped to owner,
credential/provider, and platform daily caps. Estimate upper bounds using prompt,
maximum output, and configured pricing version; block unknown pricing. Reconcile
actual usage; retain reservations on ambiguous outcomes until reconciled. A timeout
is `UNKNOWN`, not authorization for another billable request. Local limits cannot
guarantee provider bills when pricing, account activity elsewhere, or usage reporting
differs. Initial deployment daily budget is $0 and dispatch fails closed until the
human approves numeric caps and provider capacity.

Dispatch admission and revocation share transaction locks on the capability binding
and credential version. Lock budget ledgers in deployment/owner/credential order,
then affected bindings and credential rows in stable ID order; mutations taking a
subset preserve that order. Under these locks, admission checks current consent,
binding/version and revocation state, reserves cost and commits the durable call's
`dispatched` state. This commit is the admission linearization point, not proof of
network transmission. Revocation commits under the same binding/credential locks:
if it wins, admission fails; if admission wins, its request may transmit later.
Do not hold a DB transaction across network I/O. Never redispatch an ambiguous
admitted call; preserve its reservation and reconcile. Test both race orderings.

## Learner code and resource abuse

FR-017 execution is disabled until a separately approved Linux KVM microVM host
passes isolation tests. Never run learner code on the API/worker host. A plain Python
subprocess, restricted builtins, or container alone is not the chosen isolation
boundary. Use a pinned read-only Python image, ephemeral per-run scratch, no guest
network device, no host mounts, no cloud metadata, no DB/provider keys, and no reuse
of writable state. The host accepts authenticated narrowly scoped run requests; it
does not accept arbitrary VM configuration from the learner.

Per run: 32 KiB code, 2 CPU seconds, 5 seconds wall time, 256 MiB memory, 16 processes,
10 MiB writable space, and 64 KiB combined stdout/stderr. Host watchdog kills the
entire microVM on expiry; truncate output with an explicit marker. At most two
simultaneous runs deployment-wide. Tear down guest and scratch even after crash;
return only bounded text/status and grading evidence. Hidden tests are assessment
integrity aids, not secrets guaranteed against adversarial programs sharing a guest.
Do not expose host paths or exception internals to learners.

MicroVM choice still needs hardened host configuration, patching, jailer/seccomp and
host resource controls, trusted images, and maintenance ownership. The provider's
[Firecracker production host guidance](https://github.com/firecracker-microvm/firecracker/blob/main/docs/prod-host-setup.md)
supports this operational dependency; it does not establish this design's security.
If suitable hosting is unavailable at the approved budget, execution stays disabled
with a visible capability limitation; V1 execution acceptance remains unmet.

Enforce SRS quotas before admission: 100 accounts, 20 active sessions, 5 workspaces
per account, 20 active sources/workspace, 100,000 passages/workspace, 1 million total,
100 pending jobs, 5 model turns, 2 ingestion jobs, and 2 code runs concurrently.
Rate-limit login, discovery, upload, export, credential tests, and run submission
separately by principal and network signal; never use IP alone as identity. Numeric
per-route limits must be fixed in configuration and tested under NFR-006 before
release. Fair queues and clear retry timing prevent one owner monopolizing capacity.

## Privacy, retention, deletion, and recovery

Collect only identity linkage/verified email and learning data needed for requested
features. Do not infer sensitive traits from topics, code, language, or performance.
No training use, advertising profiling, or cross-user content retrieval is authorized
by using the product. Age/audience and deployment jurisdiction remain launch
decisions; do not claim suitability for children or jurisdictional compliance.

Assessment scores and clicks describe observed behavior under available resources;
they do not directly measure ability, preference, opportunity, or learning benefit.
Source visibility is exposure, not endorsement or preference. No universal group
fairness claim follows from equal exposure or score parity. Any later fairness study
requires a justified population, construct, lawful data basis, and approved protocol;
this specification authorizes no such study or sensitive-data collection.

| Data | Retention / deletion contract |
| --- | --- |
| Active workspace content and derivatives | Retain for requested service until deletion; deny access immediately, purge active copies within 24 hours |
| Credentials | Block new local dispatch admission immediately; active ciphertext/wrapped keys purge within 24 hours |
| Redacted operational logs | 14 days; no prompts, answers, source bodies, keys, cookies, or raw code |
| Security metadata | 30 days; bounded event type, pseudonymous actor/object ID, time, decision, correlation ID; no retained content |
| Backup copies | Expire within 30 days of creation; all replica/version/WAL/snapshot tiers must obey the bound |
| Deleted-user restore tombstones | Content-free opaque IDs only for 30 days; replay before any restored system serves traffic |
| Export staging | Delete after download or 24 hours, whichever occurs first; active deletion cancels export |

FR-024–025 deletion runs as a durable idempotent workflow: authenticate recent
intent; transactionally deny access/revoke sessions or affected source, increment
source epoch and fence jobs; enumerate relational records, blobs and versions,
vectors, derived chunks, caches, outputs, export staging, and scratch; purge; verify
inventory; expose completion or retry status. Late workers cannot publish because
the commit checks ownership, deletion state, epoch, and fencing token. Source
citation tombstones retain only opaque references, never deleted titles, URLs, spans,
or quotations. Retained turns containing deleted-source excerpts must be redacted
or purged, including generated answers/citation payloads and cached views; do not
retain the text under the label of provenance. Account deletion removes the turns.

Backups are not immediately rewritten. Keep a separately durable, content-free
deletion ledger available across restoration. Restore to an isolated environment;
replay deletions and credential revocations and expire old sessions before access.
Reject restores older than the retained deletion ledger. At the 30-day boundary,
expire corresponding recoverable snapshots before expiring tombstones, including
replicas and transaction logs. Verification of this ordering is a release gate.
RPO ≤15 minutes and RTO ≤4 hours are unverified targets; no approved backup service
or operational staffing currently establishes them.

External providers, identity accounts, downloaded exports, and learner-held copies
are outside application purge control. Explain which external requests were sent,
available provider deletion mechanisms, and unavailable guarantees. Deleting local
data does not prove external erasure. A provider lacking acceptable retention terms
must remain unavailable for the affected data use.

## Operations, evidence, and release gates

Use TLS for external and service connections, encrypted storage/backups, least
privilege service identities, pinned reviewed dependencies/images, and no secrets in
repository or build artifacts. Production data is not test fixtures. Administrative
access requires named accounts, MFA, time-bounded authorization, and auditable reason;
do not log inspected content. Emergency access and key rotation require a documented
operator. Alert on isolation failures, unusual credential use, repeated SSRF blocks,
budget reconciliation failures, and overdue deletion jobs. Incident response first
revokes affected access, disables impacted adapters/features, preserves minimal
security evidence, and assesses exposure; notifications depend on verified facts and
applicable obligations, not an invented universal deadline.

The following are future acceptance cases, not executed tests. Each inherits its
canonical ID; multiple cases under an ID are intentional.

| Test | Required adversarial evidence / pass condition |
| --- | --- |
| T-FR-001 / T-FR-002 | Wrong issuer/audience/nonce, replayed code/state, session fixation, expired and logged-out cookies rejected; foreign/missing CSRF rejected; recent-auth actions enforce age |
| T-FR-029 / T-NFR-001 | Two owners, guessed IDs, cross-workspace FK, pooled connections, jobs, search, caches, citations and exports: no foreign payload; missing RLS context denies; admin bypass remains documented |
| T-FR-007 | Controlled HTTP fixtures cover redirects, rebinding, IPv6/mapped addresses, metadata, credentials, nonstandard ports, oversize/decompression and slow bodies; no blocked destination connection or ambient secret forwarding |
| T-FR-008 / T-FR-010 | Malformed PDF/HTML and parser resource breach produce bounded failure and scratch cleanup; no partial searchable version; cancelled/deleted jobs cannot publish |
| T-FR-012 / T-NFR-001 | Injected passages request secrets, tool use and foreign citations; deterministic boundary rejects privileged actions; unsafe HTML/URLs inert; unsupported content abstains; model-quality limits recorded separately |
| T-FR-017 | Network probe, host-file read, process storm, infinite loop, memory/disk/output overflow and persistence attempt confined/terminated; subsequent guest sees no prior state; no fallback to application execution |
| T-FR-020 / T-FR-021 | No-consent admission blocked; key absent from responses, exports, traces, crash logs and jobs; cross-owner key rejected; revocation-first race prevents admission, admission-first retains disclosed possible transmission/billing; encryption context tampering fails |
| T-FR-022 / T-NFR-007 | Concurrent reservations cannot exceed local cap; unknown price/$0 cap blocks; timeout retains reservation with no blind retry; provider/local billing discrepancy displayed |
| T-FR-024 / T-FR-025 / T-NFR-002 | Delete during retrieval, generation, ingestion and export; access immediately denied including idempotent replay; inventory empty within 24h including answer excerpts and replay payloads; external limits visible |
| T-NFR-012 | Restore oldest permitted backup with deleted owners/credentials; deletion replay precedes traffic; all backup tiers expire by 30d; record measured RPO/RTO and failures |
| T-NFR-011 | Canary nonproduction key/content never enters telemetry; retention expiry works; security events retain actionable metadata without content |

Tests use synthetic local fixtures and provider doubles unless separately approved;
no scanning public systems or live billable calls is authorized. Release blocks on
failed isolation, secret exposure, unconstrained execution/acquisition, or deletion
resurrection. Model answer-quality tests cannot substitute for these deterministic
controls. Independent review must inspect code, deployed configuration, and test
evidence at implementation; this document alone provides none of that audit access.

Open decisions: identity/provider selection and terms; hosting/KVM feasibility;
key-management and backup implementation; operator/incident ownership; supported
audience/jurisdiction; per-route rate limits; parser performance at its proposed
limits. These are implementation/release gates, not reasons to assert completed
security. The human approves deployment and spending; the author cannot approve
their own security specification.
