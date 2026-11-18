# Learning engine specification

Date: 2026-11-17. Status: proposed M01 specification; not implemented, measured,
or an approved scientific protocol. Owner: delegated learning/testing author;
separate reviewer and human acceptance pending. Base: `f8c967f`.

Normative requirement identifiers are in [SRS](../requirements/SRS.md). Architecture
and trust boundaries are in [system architecture](../architecture/SYSTEM_ARCHITECTURE.md)
and [security model](../architecture/SECURITY_MODEL.md). Acceptance cases are in
[test strategy](../testing/TEST_STRATEGY.md). The [charter](../charter.md) establishes
purpose; these proposed decisions resolve previously open details only upon approval.

## Audience and boundaries

V1 proposal: English-language, adult, self-directed learning from permitted HTML
and text-based PDF resources. Scanned PDFs, OCR, audiovisual lessons,
institutional credentials, certification, and minors' learning are outside the initial
acceptance envelope. Python exercises use the pinned standard-library environment;
arbitrary dependency installation is excluded. The application itself is not restricted
to Python's standard library. Learner age is an eligibility declaration, not a verified
identity claim. Audience, supported formats, and progression thresholds need human
acceptance. No diagnosis of learning ability or validated proficiency score is offered.

The learner sees goals, lessons, source references, practice, and evidence of progress.
They do not configure retrieval internals. Opening a lesson, elapsed time, chat length,
self-reported confidence, and model praise never count as mastery evidence.

## Goal and pathway construction (FR-003, FR-004, FR-013)

Store each goal revision with desired outcome, prior knowledge, scope, constraints,
learner confirmation, and unresolved questions. A model may suggest these fields;
the learner confirms the interpretation before accepting a pathway. If the objective
is ambiguous, ask one focused clarification at a time. A skipped diagnostic produces
`unknown` prerequisite evidence, never an assumed pass or a failing grade.

Diagnostic questions align to explicit prerequisite objectives and use the assessment
rules below. They can propose a starting point but do not silently waive prerequisites.
Provide a readable explanation and allow the learner to study prerequisite lessons.

A pathway version contains an immutable goal revision, objective nodes, prerequisite
edges, lesson ordering, supported source-version spans, estimated effort labelled as
an estimate, and assessment criteria. An objective specifies an observable action,
scope, expected evidence, and rubric version, for example: implement a function that
handles documented empty-input behavior and explain that choice. Avoid objectives
such as "understand Python" without observable criteria.

Before presenting a proposal:

1. Check all objective IDs and prerequisite edges reference this pathway version;
   reject self-edges, duplicate edges, and cycles using topological sorting.
2. Verify each teaching objective has at least one accessible, approved, published
   source version; flag unresolved prerequisite coverage and contradictory sources.
3. Verify every assessed objective has an assessment plan and rubric criteria; require an available aligned exercise before assessed progression;
   block automatic progression where reliable assessment is unavailable.
4. Present sequence, outcomes, gaps, and prerequisites for learner acceptance.

Acceptance atomically selects the pathway version with expected workspace revision.
Concurrent edits return a conflict for review. Regeneration creates a proposal; it
does not overwrite an accepted pathway. Transfer prior evidence only through explicit
objective-equivalence mapping reviewed by the learner, preserving original objective,
rubric, attempt, and pathway IDs. Changed assessment criteria default to no transfer.
Manual prerequisite bypass permits navigation but leaves the prerequisite evidence
unknown and shows its implications; it cannot mark mastery.

## Teaching and source grounding (FR-011, FR-012, FR-014)

Each turn binds workspace, learner, accepted pathway version, session revision,
objective, authorized source epoch, provider binding, and budget reservation. Permit
one active turn per session. Queue work transactionally, return a durable job ID,
and let the learner resume polling after reconnecting. Duplicate request keys return
the original turn. Reject stale session revisions instead of overwriting newer work.

Retrieve only currently approved, published versions in that workspace. The proposed
baseline combines full-text and dense candidates with reciprocal-rank fusion, then
uses section ancestry to provide context within the prompt budget. Record candidate
IDs, scores, index generation, selection policy, source spans, and exclusion reasons
in restricted evaluation traces without retaining raw prompts in operational logs.
Do not describe this as trained STAIR or assume it improves learning.

Treat passages and learner code as untrusted data. They cannot override system policy,
enable tools, access credentials, or authorize resource acquisition. For every substantive
source-dependent teaching claim, bind a citation to an immutable version, passage,
and page/heading/character span. Validate ownership, version visibility, and span
existence mechanically. A separate support check assesses whether the passage supports
the claim; mechanical citation validity alone is insufficient. Support checking is a
fallible product control and must pass human evaluation before release.

If evidence is absent, conflicting, outdated for the question, or fails support checks,
clarify, explain the limitation, or abstain. A limited answer may state supported facts
while identifying unanswered parts. Do not fill gaps with uncited model recollection
presented as source-grounded teaching. Separate source claims from learner-directed
questions and clearly labelled illustrative examples. Unsupported generated examples
must not become assessment answer keys without validation.

Recheck source epoch and authorization before committing the answer and before rendering.
If a source became unavailable, withhold affected text/citations and mark regeneration
required. No partial model output reaches the learner before these checks. Persist the
turn, citations, final status, and session revision in one transaction. A gateway timeout
with unknown billing status yields a recoverable failed/unknown operation requiring
reconciliation, not an automatic duplicate model call.

## Exercise and assessment contract (FR-015, FR-016, FR-017)

An exercise version stores objective mapping, supported source spans, instructions,
answer schema, criterion IDs, criterion weights, grading method/version, feedback rules,
and assistance policy. Weights must be nonnegative and sum to one; missing mandatory
criteria invalidate the rubric. Fix the rubric before an attempt starts. Never change
the answer key after observing a learner's answer without a visible correction event.

Use objective grading where appropriate: explicit normalized choices, documented
numeric tolerances, or deterministic code assertions. Avoid string equality for answers
with multiple valid formulations. A free-response rubric uses anchored criterion scores
in {0, 0.5, 1}; score is the weighted sum. Store criterion scores, explanatory evidence,
grader identity/version, and uncertainty/status. A model assessor must cite relevant
response portions and may return `needs_review`; it cannot convert an unscorable
response into zero. Self-grading or an unvalidated model grade is practice feedback
only and ineligible for automatic mastery.

Release of automatic free-response mastery requires an independently reviewed human-
labelled scoring audit with frozen rubric, disagreement policy, and accepted error
thresholds. Until that gate, free-response scores remain provisional. Learners can
report or dispute feedback (FR-027); a correction supersedes the assessment, retains
an audit trail, and deterministically recomputes progress. No claim of a human review
service is implied; a dispute can remain pending with progression withheld.

Code attempts run only on the separate isolated execution host described in the
security model. Proposed limits: 32 KiB submitted code, 2 s CPU/5 s wall, 256 MiB RAM,
16 processes, 10 MiB writable space, 64 KiB combined output, no guest network, and no
application credentials/database. Sandbox unavailability disables execution. Distinguish
assertion failure, syntax/runtime error, quota violation, infrastructure error, and
cancelled run; infrastructure failure gives no assessment score. Pin interpreter,
image, locale, test fixture, and harness. Tests must encode task requirements and
reasonable boundary cases, not a single preferred implementation. Passing tests is
evidence for tested behavior, not proof of general program correctness.

## Evidence ledger and progress (FR-018, FR-019)

Durable records: immutable exercise version, attempt, optional code run, assessment,
and progress event; a replaceable projection supplies current UI state. Every record
has workspace ownership and stable IDs. An attempt records submission time, objective
and pathway versions, assistance used, exercise-family ID, and terminal status.
Group paraphrases or regenerated variants of the same question into one exercise
family to prevent repeated memorized answers masquerading as independent evidence.

Proposed objective states: `unknown`, `practising`, `demonstrated`, `review_due`, and
`needs_reassessment`. These are interface states, not psychometric ability estimates.
Use the following deterministic policy, versioned with the projection:

1. Start `unknown`; a submitted relevant attempt sets `practising` unless stronger
   valid evidence already exists. Views and abandoned attempts do not change state.
2. Eligible evidence requires completed grading, validated rubric/grader, correct
   ownership and objective mapping, no unresolved dispute, no revealed answer or
   solution-level assistance, and no infrastructure failure. Assisted practice remains
   visible, but is excluded from progression. Record assistance rather than implying
   the platform can detect all external help.
3. Propose `demonstrated` when the latest eligible assessment in each of two distinct
   exercise families scores at least 0.80, each mandatory criterion scores 1, and
   prerequisites are demonstrated. Keep the two-family requirement explicit; it does
   not establish statistical independence or validated proficiency.
4. A later eligible failing assessment after demonstration sets `needs_reassessment`.
   Re-enter `demonstrated` only after qualifying evidence from two distinct families
   submitted after that failure. An invalidated or superseded assessment triggers
   recomputation; never preserve a status supported only by invalid evidence.
5. After 30 days without qualifying evidence, show `review_due` and offer retrieval
   practice. This is a scheduling rule, not an estimated forgetting curve. Preserve
   the historical demonstration date and permit continued study.

The 0.80, two-family, and 30-day choices are proposed product defaults requiring
acceptance and later validation; they are not paper-derived results. Display the
supporting assessments and dates, missing evidence, and next action. Workspace
completion is demonstrated objectives divided by all required objectives in the
accepted pathway; report numerator and denominator and count review-due objectives
separately. Never imply that completion establishes workplace competence.

Commit an assessment and its uniquely keyed progress event together; deduplicate by
attempt/assessment revision and use optimistic concurrency on the projection. Replay
events ordered by per-workspace sequence, not client timestamps. Corrections reference
superseded event IDs. Recovery rebuilds the same projection from the ledger with a
pinned policy version. Cross-session resume retrieves the accepted pathway, current
objective, completed turns, outstanding jobs, and current evidence state after fresh
authorization; client storage is never the authoritative ledger.

## Change, deletion, and graceful failure

Source refresh (FR-023) produces a new approval/version and a pathway-review proposal;
it cannot silently rewrite teaching evidence or assessment provenance. Source deletion
(FR-024) removes content access immediately and leaves only permitted citation tombstones.
Mark dependent lessons unavailable and affected objectives `needs_reassessment` where
the retained grading evidence cannot be verified. Preserve minimal historical outcome
metadata only while account retention permits, never source excerpts in tombstones.
Account deletion (FR-025) also deletes attempts and evidence under the privacy schedule.

When model budget is zero, provider credentials are invalid, ingestion is incomplete,
or the sandbox is disabled, explain the blocked capability and retain completed work.
Do not manufacture lessons, grades, or progress. Allow viewing authorized retained
material and manual resource submission where supported. Pathway edits and retries
must not erase previously accepted versions or duplicate evidence.

## Required evidence before release

T-FR-004, T-FR-012 through T-FR-019, T-FR-023, T-FR-024, T-FR-027, and T-NFR-010
exercise these rules with deterministic fixtures, persistence tests, and reviewed
human judgements. Retrieval/answer quality, grading reliability, accessibility, and
source suitability are separate release gates in the test strategy. Neither passing
software tests nor retrieving relevant passages establishes learning efficacy. Any
study of learning outcomes needs a separately approved protocol, participant safeguards,
appropriate comparison and sampling, and independent scientific review.


## Ambiguity, follow-ups and corrections

For FR-004/012/014 distinguish unclear learner intent from unavailable source evidence.
Ask a targeted clarification for the former; disclose missing evidence for the latter.
Resolve follow-up references using bounded session history; topic changes must not
inherit unrelated assumptions. On a challenged earlier answer, retrieve again and
correct it when warranted. Never cite the assistant's earlier answer as evidence.
These behaviors are motivated by [MTRAG-UN](../research/ibm-rag-additional-evidence.md)
and have explicit conversation regression cases in the test strategy.
