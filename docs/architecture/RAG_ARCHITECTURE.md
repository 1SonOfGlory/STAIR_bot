# Retrieval and knowledge architecture

Date: 2026-11-17. M01 proposed V1 specification. Owner: M01 RAG/operations author.
Baseline: `f8c967f`. Separate review and human acceptance pending. No retriever has
been implemented, benchmarked, or selected as a scientific winner.

## Contract and method choice

[SRS](../requirements/SRS.md) owns FR-005–012, FR-023–024, FR-029–030 and the
canonical acceptance targets. This document elaborates their knowledge pipeline.
See [system architecture](SYSTEM_ARCHITECTURE.md), [data model](DATA_MODEL.md),
[security](SECURITY_MODEL.md), [learning](../learning/LEARNING_ENGINE.md), and
[test strategy](../testing/TEST_STRATEGY.md) for adjacent contracts.

The leading engineering candidate combines explicit document hierarchy, PostgreSQL
full-text search, and pgvector dense retrieval. It uses no corpus-specific model
training. PostgreSQL full-text ranking is not claimed to be BM25. The
[pinned STAIR assessment](../research/stair-review.md) distinguishes this adaptation
from STAIR's supervised section-ID generator and constrained decoding. Its reported
invalid-ID rate is not an answer hallucination rate. Original STAIR remains a
reference requiring separate training, update/deletion, and compute feasibility
approval. Generated summary trees and learned reranking are deferred until a measured
gap justifies their additional inference, maintenance, and provenance costs.

V1 supports English HTML and text PDFs; no OCR, recursive crawling, multilingual
quality claim, or arbitrary formats. Poor extraction produces an explicit unsupported
or review-needed result, not a silently incomplete searchable document. All model
transmission, including embeddings, passes provider consent and budget admission.

## Discovery, approval, and immutable acquisition

FR-005 proposes approximately five relevant resources with publisher, URL, topic
coverage, format, freshness evidence if available, access/permission basis, and a
concise recommendation rationale. Credibility signals are evidence for inspection,
not a truth certification. Search snippets are discovery metadata, never answer
evidence. Explain fewer results; do not fabricate missing resources. FR-030 offers
manual public URL entry or permitted upload if search is unavailable or declined.

Candidate approval plus permission evidence authorizes bounded acquisition only.
Present acquired metadata, SHA-256 and change status for exact-version approval
before extraction, indexing or model transmission. A previously known exact hash
can combine the approvals if the acquired bytes match. Uploads remain quarantined
until exact-hash approval. No changing remote URL inherits approval indefinitely.
Approval is independent of acquisition rights; refresh repeats these gates.

Acquire bounded bytes into private staging, compute SHA-256, and preserve requested
and final URL, media type, byte count, acquisition time, permission evidence, and
parser input hash. Uploads get the same checks. Security owns destination validation,
redirect limits, type checks, and isolated resource bounds. Never execute HTML,
embedded code, or source instructions. Remote modifications create a new immutable
version; the old active version remains available during permitted refresh failure.

## Hierarchy, passages, and provenance

Each version contains a rooted, acyclic section tree with stable IDs within that
version, parent ID, sibling order, original heading, heading path, and extraction
method/confidence. Prefer explicit HTML headings or a validated PDF ToC; reconcile
ToC destinations with text locations. When missing, infer bounded sections from
layout/heading cues and mark them inferred; otherwise use a flat root with ordered
passages. Never invent a publisher-supplied hierarchy. IDs are not reused across
versions merely because headings match.

Preserve paragraphs, lists, code blocks, and tables as semantic units where possible.
Initial configurable candidate: target 400 tokens per passage, hard limit 800, and
at most 80 overlap tokens, all measured with a pinned tokenizer. These are proposed
evaluation settings, not established optima. Split oversized units at recorded
boundaries with continuation markers; do not silently drop trailing text. A table
or code block that cannot be faithfully represented is flagged for inspection.
Overlap retains original offsets so duplicated text is not treated as new evidence.

Every passage records workspace, source/version, section, ordered position,
normalized-text offsets, original page/HTML anchor mapping, content hash, extraction
configuration and parser version. PDF pages use explicit one-based display numbering
and a stored mapping to file pages. HTML anchors refer to the stored snapshot; a live
site may change. Preserve a normalization-to-source mapping for quotations; offsets
into normalized text alone are insufficient proof of an original quotation.

Original text and heading breadcrumbs remain distinct fields. Embedding input may
prepend ancestor headings, but this synthetic prefix is never presented as quoted
body evidence. Generated summaries, if later approved, carry source-span lineage and
cannot replace original authoritative spans. All derived records remain private.

## Index generations and publication

A generation pins embedding provider/model revision, dimension, distance function,
tokenizer, normalization and chunking versions, lexical configuration, and pipeline
revision. Query embedding uses the same compatible generation; vectors from different
models/dimensions never share a ranked distance space. Record provider limits and
batch sizes explicitly before enabling ingestion. Retrying a batch must reuse durable
step IDs and respect gateway unknown-outcome handling.

Stages are acquisition, extraction, structure validation, passage construction,
lexical indexing, embedding/index validation, and publication. Persist stage outputs
and redacted failures for FR-008 status/retry/cancel. Selected hybrid publication
requires complete lexical and dense coverage; no partial version becomes searchable.
A separately evaluated lexical-only mode could be selected explicitly later, but
provider outage must not silently change the active retrieval contract.

Publication verifies immutable object existence/checksum, complete stage manifest,
ownership, permission, version approval, source expected revision, deletion state,
quota reservation, and job fencing token. One database transaction swaps the active
version pointer and increments workspace source epoch. Rows and objects staged by a
failed or losing concurrent refresh remain invisible and are garbage-collected.

Embedding migrations build a complete shadow generation, validate it against the
same approved content inventory, and switch its pointer atomically. Keep the old
generation only within authorized retention and storage limits for rollback. A
rollback may never reactivate revoked/deleted content. Model/provider changes require
new consent where applicable and funded capacity for both generations; do not claim
zero-cost migration or mix partially converted sources.

## Query execution and context selection

1. Authenticate the caller and resolve workspace ownership. Capture active version
   inventory, source epoch, generation, and retrieval configuration. A deleted or
   unauthorized workspace denies before search; a workspace without usable material
   returns a useful abstention or source-approval action.
2. Form a bounded query from learner question and accepted objective/session context.
   Treat all text as data. Do not send entire history or unrelated private documents.
   Query embedding requires active binding, consent and budget admission.
3. Search authorized active versions using lexical ranking and dense similarity.
   Initial candidate takes top 50 from each with deterministic ID tie-breaking.
   Authorization is part of candidate eligibility, not merely an output filter.
   Start with exact dense search for the bounded baseline; introduce approximate
   indexes only after profiling and recall checks with realistic ownership filters.
4. Deduplicate by passage ID and combine using reciprocal rank fusion:
   `score(p) = sum(1 / (60 + rank_i(p)))`, ranks starting at one; absent passages
   contribute zero. Constant 60 and candidate counts are tunable protocol settings,
   not similarity thresholds or calibrated confidence scores.
5. Select up to eight fused passages, then add directly related parent/neighbor
   context only within the same authorized version. Deduplicate overlapping spans;
   cap total evidence at 6,000 tokens and the selected model's remaining context
   budget, whichever is smaller. Preserve question/instructions/output allowance
   first. Expansion cannot bypass authorization or crowd out evidence without a
   recorded selection reason. All counts and tie-breaks are configuration-versioned.
6. Record the selected evidence manifest in restricted application state: IDs,
   spans, source epoch, generation/config revision and ranks. Do not put source
   text or query text in operational telemetry. Recheck eligibility before model
   dispatch and again before committing/serving an answer.

No raw rank/score is shown as a probability of truth. The default does not invoke
an LLM to select section IDs, rewrite arbitrary queries, or perform autonomous
multi-hop browsing. Candidate starvation, embedding failure, and source changes
produce explicit failures/abstentions rather than an unannounced retrieval fallback.

## Grounding and citation delivery

FR-012 answers use selected evidence with claim-linked citation references. Model
output must conform to a typed result containing answer/abstention and citation span
IDs from the supplied manifest. Deterministically validate membership, ownership,
active version, offset bounds, and quoted-text agreement. Reject invented anchors,
foreign sources, and source instructions that attempt privileged actions. Citation
links resolve through authenticated application endpoints, not public storage URLs.

Structural validity alone cannot prove entailment. A separate support check must
assess whether each substantive teaching claim follows from its cited evidence;
the implementation/threshold and independently labeled evaluation must be approved
at M04 before tutoring is enabled. Any model-based checker is fallible, requires its
own budget reservation, and cannot replace human evaluation. If checks fail or
evidence is insufficient/conflicting, return a bounded explanation of the gap and
ask for clarification or an approved source. Do not present unsupported facts as
grounded instruction. One repair attempt may be configured only within the original
deadline and fresh cost admission; otherwise abstain.

Tutor results are buffered until checks finish; no unchecked text streaming. Source
epoch changes before commit invalidate pending evidence. Historical rendering checks
each cited version against current eligibility; unrelated epoch changes trigger
revalidation, not automatic deletion of otherwise eligible history. Replaced sources
show the prior answer as unavailable/stale under the active-version policy, with an
explicit regeneration action; never silently attach new spans to an old claim.

## Refresh, deletion, and derived-content invalidation

FR-023 refresh produces an inspectable candidate version and requires reapproval.
An unchanged checksum reuses the existing authorized version without redundant reindex.
Concurrent refresh uses expected source revision; a loser cannot overwrite the
winner. Revoked rights/approval immediately stop access even if a replacement fails.
FR-024 deletion atomically denies access, increments source epoch and fences pending
jobs before background cleanup. Retrieval and citation caches include owner,
workspace, epoch and generation and are invalidated on change; no shared answer cache.

Purge raw bytes, snapshots, section/passage text, vectors, staged artifacts, generated
excerpts, exports and scratch within 24 hours. Use dependency lineage to redact or
purge affected tutor turns, pathway text, exercises, assessment explanations and
other retained derivatives; if span-level removal is uncertain, purge the entire
affected content artifact. Preserve only content-free citation tombstone IDs. Mark
dependent learning artifacts unavailable and require regeneration/reacceptance;
retaining progress metadata must not retain deleted text or claim evidence remains
verifiable. Account deletion removes the account's learning records altogether.
Backup expiry and restore deletion replay follow security and operations; local
purge cannot guarantee erasure by external providers or downloaded copies.

## Evaluation and acceptance gates

This is a bounded future evaluation specification, not an approved experiment.
Before M04 execution, obtain independent protocol review, human authorization and
acceptance of the test strategy's proposed numeric quality gates/sample-size justification. No datasets, training,
paid calls, or experiments are authorized by this document. Use the shared evidence
and reproducibility workflows at that gate.

Compare lexical-only, dense-only, hybrid without hierarchy, and hybrid with hierarchy
using identical corpus snapshots, permissions, query sets, embedding model, evidence
token budget and answer model. Freeze development-tuned parameters before held-out
evaluation. Include human-relevant questions, weak/no structure, ambiguous headings,
cross-resource questions, multi-section answers, unanswerable questions, and source
updates/deletions. Group near-duplicates and related sections/questions across splits;
hold out entire documents for transfer evaluation. Record any same-book split
separately; it does not demonstrate unseen-document generalization. Independent
annotators check support/relevance, adjudicate disagreement, and preserve unresolved
label ambiguity without inspecting system identity where practical.

Use the test strategy's Recall@5, sufficient-evidence@5 and nDCG@5 definitions as
the primary retrieval measures; any other k is a separately declared diagnostic.
State denominators and aggregation. Report invalid citations, claim support,
answer correctness, abstention on answerable/unanswerable questions separately.
Include uncertainty clustered by source/document, per-format/structure results and
failure examples; no single answer metric establishes learning benefit or fairness.
Record ingestion/update/delete times, query latency, memory, storage, provider calls,
tokens and cost. Pin corpus hashes, permissions, labels/splits, configuration,
dependencies, model revisions, hardware, commands and raw result provenance.

| Canonical test | Required evidence / disposition |
| --- | --- |
| T-FR-006 / T-FR-007 | Changed bytes cannot inherit approval; acquisition permission and safe-fetch fixtures pass |
| T-FR-009 | Golden HTML/PDF fixtures preserve section order, code/table handling and source-span mapping; inferred structure is labeled |
| T-FR-010 / T-FR-026 | Crash at every stage or competing refresh produces either old complete version or new complete version, never partial search |
| T-FR-011 / T-FR-029 | Foreign, unapproved, revoked, staged and deleted versions return no evidence, including with pooled DB connections and index filtering |
| T-FR-012 | Invalid spans rejected deterministically; independently scored support/correctness/abstention meet predeclared thresholds before release |
| T-FR-023 / T-FR-024 | Refresh/delete during retrieval, generation and rendering prevents stale release; derivative inventory purged within deadline |
| T-NFR-005 / T-NFR-006 | Retrieval p95 ≤2s under operations workload; record exact/approximate recall and capacity before any scale claim |
| T-NFR-007 / T-NFR-010 | Reproduce ranking/config lineage and resource accounting; unknown cost or insufficient budget blocks dispatch |

Select the simplest candidate meeting all accepted quality, isolation, latency,
update/deletion and cost gates. A quality regression or unmet threshold blocks
promotion; no presumed superiority of structure or STAIR excuses it. Open decisions
include embedding/provider feasibility, support checking, extraction quality,
annotation/sample budget, acceptance of proposed quality thresholds and statistical
decision rules, and measured storage/latency.

## Conversation handling and bounded tuning

The [additional IBM evidence](../research/ibm-rag-additional-evidence.md) motivates
these proposed M04/M05 checks, without adopting its model rankings as product results.
For FR-012/014, resolve references from bounded authorized conversation history, but
retrieve evidence again for the current question. Distinguish ambiguous intent
(ask a targeted clarification) from missing evidence (abstain or qualify). A learner's
correction triggers fresh retrieval and an explicit correction when warranted; previous
assistant claims never count as documentary evidence. Topic switches reset irrelevant
history, while the immutable turn records retain the actual input and selected context.

Compare last-turn retrieval, deterministic bounded history and optional model rewrite
on the same conversation-held-out tasks. Charge rewrite calls through the gateway;
no rewrite runs by default before its quality, latency and cost gate passes. Freeze
history/token limits and rewriting rules before confirmatory evaluation. Record retrieval
and answer results separately by ambiguity, answerability, nonstandalone reference,
clarification/correction and topic switch, including wrong earlier assistant answers.

When approved, tune on development data with a predeclared small search and fixed
spend ceiling. Compare model-first greedy and random search under equal total resource
budgets, including reused indexing, inference and judging. Keep human-audited grounding
and abstention constraints alongside answer adequacy. Do not select configurations on
confirmatory test results or promise the HPO paper's convergence on our search space.

Keep backend-independent passage IDs, source versions and index generation metadata.
IBM's specialized 100-billion-vector prototype does not alter the E1 PostgreSQL choice;
a backend change requires measured tenant-filtered performance and migration evidence.
