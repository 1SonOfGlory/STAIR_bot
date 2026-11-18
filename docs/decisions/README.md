# Decision log

Record significant decisions here. Each entry includes ID/date, artifact revision,
owner/reviewer, decision or claim, evidence, consequence, alternative/check, response,
disposition (pass/revise/blocked), limitations, and approving authority. A pass within
current scope does not approve implementation or certify production readiness.

## D001: Charter-only boundary

- Date: 2026-11-17. Owner: coordinator. Reviewer: research-scientist, M0-REVIEW (pass).
- Artifact: charter and AGENTS.md in this milestone's commit.
- Decision: document the platform and inspect the paper; stop before implementation.
- Evidence/authority: user's Prompt 1 and inherited $0 constraint.
- Consequence: no application scaffold, dependency installation, experiments, or paid use.
- Alternative: begin building immediately; rejected as outside approved scope.
- Response/disposition: pass for documentation scope. Next milestone needs human approval.

## D002: Dedicated repository and preserved lab

- Date: 2026-11-17. Owner: coordinator. Reviewer: research-scientist, M0-REVIEW (pass
  for documentation; remote operations verified by coordinator).
- Artifact: repository setup and milestone 0 report.
- Decision: use 1SonOfGlory/STAIR_bot, preserving the separate parent lab and existing MIT license.
- Evidence/authority: explicit user repository instruction and subsequent STAIR_bot name;
  inspected remote initial commit a20f5c24fa6b29eb508c5638793ed10218a52b6e.
- Consequence: feature branch and PR; no parent-lab files or supplied PDF published.
- Alternative: reuse parent lab repository; rejected due to unrelated untracked work.
- Response/disposition: pass for destination; enforcement/publication evidence in milestone report.
- Limitation: public visibility and MIT license were selected by the user; external
  source content retains its own rights.

## D003: Preserve structure; defer retriever selection

- Date: 2026-11-17. Owner: coordinator. Reviewer: research-scientist, M0-REVIEW (pass);
  reviewed artifact hash and limitations in [milestone evidence](../milestones/000-charter.md).
- Artifact: [STAIR review](../research/stair-review.md), pinned by PDF hash therein.
- Decision: recommend retaining hierarchy/provenance as first-class information;
  compare hybrid adaptation with simpler baselines before choosing a retriever.
- Evidence: supplied paper sections 4-8, Table 4; dynamic/unseen-corpus work is prospective.
- Consequence: no claim of exact STAIR reproduction, zero factual hallucination, or
  experimentally established product superiority.
- Alternative: adopt corpus-fine-tuned STAIR immediately; lacks dynamic-corpus and
  budget feasibility evidence. RAPTOR remains a conditional comparison candidate.
- Response/disposition: pass as a documented hypothesis; architecture selection blocked
  pending approved requirements, evaluation protocol, evidence, and human decision.
- Limitations: no local benchmarks or training, no verified runtime costs, code not audited.
- Approving authority: human project owner; no final retrieval architecture approved.

## D004: Proposed V1 engineering baseline

- Date: 2026-11-18. Owner: coordinator; separate engineering, security and research
  reviews recorded in [milestone 1](../milestones/001-specifications.md).
- Artifact: twelve specifications in the milestone commit; authority: user's explicit
  requirements/architecture assignment, not approval to implement.
- Decision: modular monolith, asynchronous database jobs, isolated execution host,
  versioned provenance and provider-independent model access are the candidate design.
- Alternatives: microservices, broker, dedicated vector database and trained STAIR remain
  conditional on measured constraints; do not add infrastructure speculatively.
- Evidence: product requirements, STAIR review and supplemental IBM evidence; no local
  performance or learning-efficacy results. Numeric targets are proposed release gates.
- Disposition: pass for reviewable specification scope; human acceptance and outstanding
  roadmap decisions precede their implementation gates. No experiments authorized.
- Consequence: source-byte approval, tenant isolation, deletion, admission accounting and
  recovery are mandatory contracts, not optional later hardening.
- Limitation: operational feasibility, providers, costs, model quality and sandbox safety
  require future execution evidence. Approving authority remains the project owner.
