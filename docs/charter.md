# Project charter

Date: 2026-11-17. Owner: project coordinator. Human authority: repository owner.
Status: milestone 0 documentation; implementation not authorized.

## Purpose

STAIR_bot will turn a learner's stated objective into a persistent, source-grounded
learning experience. The learner controls resource approval and should not need to
configure retrieval infrastructure. Intermediate Python proficiency is an example
goal, not a restriction to programming or a verified definition of proficiency.

## V1 capability boundaries

| ID | Required capability | Evidence required before claiming implementation |
| --- | --- | --- |
| P01 | Interpret goals and establish prerequisites | Goal clarification and prerequisite cases, including ambiguous objectives |
| P02 | Discover credible, relevant, current online resources | Provenance, relevance rationale, freshness checks, and discovery failure handling |
| P03 | Recommend approximately five resources for review and approval | User can inspect, approve, reject, or replace resources; explain fewer suitable results |
| P04 | Acquire permitted approved materials automatically | Permission evidence, approval checks, safe download and failure cases |
| P05 | Preserve semantic and hierarchical structure | Source-to-section fidelity checks, page/heading anchors, and handling of missing structure |
| P06 | Persistent searchable knowledge base | Persistence, ownership isolation, search, update, deletion, and recovery tests |
| P07 | Create a learning pathway | Prerequisite ordering and links to approved resource versions |
| P08 | Interactive source-grounded teaching | Verifiable citations, unsupported-question behavior, and answer support evaluation |
| P09 | Practice and assessments | Outcome alignment, scoring checks, useful feedback, and attempts retained |
| P10 | Executable programming exercises when relevant | Isolated execution, bounded resources, abuse cases, and deterministic grading checks |
| P11 | Progress across sessions | Authenticated resume, durable state, and concurrency/recovery behavior |
| P12 | Platform models and user API credentials | Provider configuration, secure key lifecycle, limits, failure handling, and no key leakage |

These are acceptance directions, not a complete requirements specification or claims
of implemented behavior. Measurable targets must be approved in the relevant milestone.

V1 must be a maintainable consumer application with real persistence, security,
operability, and evidence of working behavior. Institutional administration is excluded.
Ownership and authorization boundaries must permit future organization membership
without implementing institutional features now. Cosmetic polish follows core function;
basic usability and accessibility remain release concerns.

## Authorization and constraints

Milestone 0 covers the charter, permanent instructions, documentation organization,
decision log, and complete inspection of the supplied paper. The user subsequently
authorized a repository under 1SonOfGlory and selected STAIR_bot. The user created it
public with an MIT license; those choices are preserved. No application implementation,
dataset downloads, experiments, model training, deployment, paid usage, or package
installation belongs to this milestone. The inherited spending limit remains $0.
Platform-provided inference is a product requirement, not approval for service charges.

## Success and handoff

Milestone 0 is complete when purpose, boundaries, operating rules, process, research
implications, and open decisions are internally consistent and reviewable in GitHub.
Repository controls and publication evidence must be reported accurately. The next
milestone needs explicit human approval; requirements engineering is a proposed next
step, not work initiated by this charter.

Open product decisions include proficiency definitions, first supported source formats,
learner age/audience, assessment standards, provider support, retention, accessibility
targets, deployment constraints, and affordable operating limits. Open architecture
decisions include retrieval strategy, storage, framework, background work, and the
execution sandbox. No framework or database is selected here.
