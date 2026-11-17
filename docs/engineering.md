# Engineering operating process

The permanent rules are in [AGENTS.md](../AGENTS.md). This document makes their
execution and evidence requirements explicit.

## Milestone record

Every milestone record includes approval and scope, mapped requirement IDs, owner,
acceptance criteria, minimal plan, inspected revision, changes, executed checks,
results, review findings, decision links, commit/PR references, limitations, and a
stop condition. Unexecuted checks must be labelled unexecuted. A milestone can remain
blocked on publication even when its local artifact is finished.

Keep changes small enough to review. Use unit tests for domain behavior, integration
tests for persistence/provider boundaries, and end-to-end tests for critical learner
journeys as those capabilities are introduced. Include error, permission, and isolation
cases. Select and pin actual Python/TypeScript tooling when implementation is approved;
do not scaffold placeholder applications or claim a static-check suite exists now.

## Git and review

Inspect status and diff before editing and before staging. Stage only owned files.
Fetch and inspect remote changes before publication. Verify origin points to
1SonOfGlory/STAIR_bot, the intended feature branch is checked out, and the patch
contains no credentials, private learner data, or unlicensed source materials.
Use descriptive commits and PRs explaining the problem, resulting behavior, checks,
limitations, and decisions. Disclose AI-assisted authorship. Preserve existing work.

Require a PR for main, disallow force pushes/deletions where supported, and require
review before merge. Required CI checks should be added only once real checks exist.
Record any platform limitation; unsupported enforcement does not waive the workflow.
A repository owner may need another human collaborator to approve a PR they authored.
Internal specialist review supplies evidence but is not a GitHub approving review.

## Design and release gates

Before enabling a capability, define failure behavior and verify it:

- Acquisition: resource approval, rights/provenance, permitted fetch paths, SSRF
  protection, redirect and size limits, malicious file handling, and retry behavior.
- Knowledge: immutable source versions and stable internal IDs, hierarchy and source
  anchors, extraction quality, incremental replacement/deletion, and access filtering.
- Teaching: distinguish retrieved evidence from instructions, prevent tool authority
  from being delegated to source content, cite source passages, and abstain or clarify
  when evidence is inadequate. Retrieval validity alone is not answer correctness.
- Identity and secrets: authenticated ownership, authorization on each data boundary,
  encryption and controlled server access to stored keys, revocation/deletion, log
  redaction, and explicit consent for content sent to providers.
- Execution: separate sandbox trust boundary; no credentials or application database
  access, default-denied network, filesystem/process isolation, time/memory/output
  quotas, termination, and adversarial escape testing before learner execution.
- Operations: migrations and rollback, tested backups/restores, bounded queues and
  retries, cancellation/idempotency, observability without sensitive payloads, incident
  handling, provider outages, and measured latency/cost limits.
- Learning: prerequisite and outcome alignment, reproducible scoring where possible,
  progression based on evidence rather than activity alone, and user recovery paths.

These are design obligations, not implemented guarantees or choices of infrastructure.
Product-specific release thresholds and threat models must precede the relevant build.

## Evidence and decisions

Use [the decision log](decisions/README.md) for consequential choices, alternatives,
evidence, tradeoffs, and reversibility. Research evidence must pin source revisions.
An approved evaluation protocol must fix outcomes, splits, baselines, uncertainty
methods, runtime limits, and decision criteria before confirmatory results are seen.
Do not infer learning efficacy, fairness, causation, or production reliability from
retrieval benchmarks alone. Preserve dissent and narrow unsupported claims.
