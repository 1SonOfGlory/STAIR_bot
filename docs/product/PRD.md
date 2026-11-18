# V1 product requirements

Date: 2026-11-17. M01 specification for review, not implemented capabilities.
Owner: coordinator. [Charter](../charter.md); normative requirements: [SRS](../requirements/SRS.md).

## Product outcome and audience

STAIR_bot helps a learner turn a goal into a source-grounded pathway, lessons,
practice, and a durable record of demonstrated progress. A learner should understand
what they will learn, why a resource was proposed, what evidence supports an answer,
and what to do next. They should never configure vector indexes or retrieval parameters.

Initial launch assumptions, requiring product acceptance before M02: self-directed
adults; English interface and resources; public HTML and text-based PDFs plus permitted
uploads; Python standard-library exercises as the first executable language. Tutoring
is not restricted to Python, but every supported subject needs source and assessment
coverage. No professional credential, medical/legal advice service, or verified
learning-efficacy claim is offered. These assumptions narrow release validation, not
the long-term vision. Children, institutional administration, sharing, video transcription,
OCR, authenticated crawling, arbitrary package installs, and multiple execution languages
are outside V1. Missing required V1 functionality cannot be disguised as a completed release.

## Core journey

1. Register through an approved identity provider and receive a private account. Explain
   provider data use before transmitting learning data. Login failure preserves local form input.
2. Create a workspace with a goal, current experience, available study time, and desired
   outcome. Ask targeted clarification; offer a prerequisite diagnostic and allow skipping
   with an explicit unverified prerequisite state. Do not infer sensitive traits.
3. Discover approximately five distinct credible resources. Cards show title, publisher,
   URL, relevance, language, date checked, known publication/version date, coverage,
   format, and acquisition permission basis. Unknown dates are shown as unknown.
   Fewer suitable sources are allowed with an explanation. A manual URL/upload route
   remains available when discovery is unavailable, without pretending discovery succeeded.
4. Review and approve selected sources. Download only under the recorded permission
   and approval. Changed content receives a separate version and approval gate. Show
   queued/processing/ready/failed status, actionable errors, cancellation, and safe retry.
5. Accept a proposed pathway with outcomes, prerequisites, source coverage, and assessment
   checkpoints. Unsupported objectives remain gaps to resolve, not invented lessons.
6. Study one objective with explanations and clickable source excerpts. Ask follow-up
   questions; receive a grounded answer, a clarification, or an honest insufficient-evidence
   response. Show progress while waiting, not unverified streamed answer fragments.
7. Practice with disclosed grading criteria. Run Python exercises in an isolated runner;
   use tests and explanations to revise. Distinguish tentative AI feedback from objective
   scoring. A failed runner never implies a failed learning outcome.
8. Resume on another session with the same accepted pathway, completed turns, attempts,
   and next suggested objective. Review evidence, report errors, refresh sources, export,
   or delete data. Deleting a source may invalidate affected lessons and evidence.

## User control and quality

Autonomy means executing approved, bounded workflows; it does not mean uncontrolled
web access, spending, code execution, or silently changing curriculum. Resource approval,
provider consent, pathway acceptance, and billable limits remain visible decisions.
Model keys are optional: platform inference is a V1 requirement, BYOK an additional mode.
The current $0 budget authorizes neither. A funded or genuinely available platform route
must be demonstrated before launch; BYOK alone does not satisfy the charter.

The interface exposes plain-language statuses, source evidence, remaining limits and
recovery options. Basic accessibility is required, including keyboard-only learning and
an accessible alternative to a rich code editor. Workspaces are private and owner-only.
No social features, leaderboard, badges implying certification, or engagement optimization
is required. Progress reflects assessment evidence, not time spent or message volume.

## Product acceptance and measurement

All P01-P12 charter capabilities map to SRS requirements and the [traceability matrix](../planning/REQUIREMENTS_TRACEABILITY.md).
Release requires the entire source-to-learning-to-resume journey, safe source and account
deletion, both provider modes, and isolated Python execution. A disabled feature is an
honest pre-release limitation, not a reason to mark its acceptance criterion complete.

During approved validation, measure task completion for goal-to-first-lesson, citation
opening success, acquisition failure recovery, successful resume, and understanding of
assessment feedback with consented testers. Define denominators, participant selection,
and observation windows before making user-experience claims. Do not invent conversion
or learning uplift targets without a baseline. Operational latency/capacity and quality
release thresholds are proposed in SRS/testing and must be measured, not advertised now.

## Decisions and risk ownership

The human product owner accepts initial audience/formats, provider disclosure, release
budget, and scope changes. The engineering owner proposes implementation and records
evidence; separate reviewers assess security, data validity and research claims.
Important open gates are listed in [roadmap](../planning/ROADMAP.md): hosting and funding,
identity/search/model providers, key management, execution hardware, corpus rights,
retrieval quality, and availability staffing. No dates or operating budget are promised.
