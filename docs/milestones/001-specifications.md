# Milestone 1: Version 1 specifications

Date: 2026-11-18. Owner: coordinator. Authority: user's requirements-engineering
assignment and subsequent IBM sources. Scope: documentation only; spending $0.
Base: `f8c967f66e72927683c77f4cd45869f165251156`, branch
`codex/v1-engineering-specs`, repository `1SonOfGlory/STAIR_bot`.

## Deliverables and decisions

The twelve requested specifications define 30 functional and 12 nonfunctional
requirements, module boundaries, logical data ownership, API contracts, learning rules,
security controls, validation strategy, operations, traceability and milestones M02-M08.
Proposed numeric targets are not measurements. Provider, hosting, model, evaluation
protocol, sandbox feasibility, release budget and product-scope decisions remain gated
in the roadmap. No application or experimental implementation was added.

Use a modular monolith with durable PostgreSQL jobs and a separate isolated execution
host as the candidate design. Preserve hierarchy and provenance, compare flat/hybrid/
structural retrieval before selecting it, and defer trained STAIR adoption. Additional
IBM evidence informs conversation tests and bounded tuning; it does not justify a
100-billion-vector storage deployment. See the linked research notes in README.

## Internal review record

All participants are AI-assisted specialist roles, not independent external peer review.
The human retains approval authority for requirements, experiments and implementation.

| Review | Owner / separate reviewer | Finding and consequence | Response / disposition |
| --- | --- | --- | --- |
| M1-SEC | Coordinator API/data; safety reviewer | High: replay could expose deleted text; revocation race ambiguous. Medium: preapproval parsing conflicts with source consent | ML author corrected API/data; security author corrected system/security; coordinator aligned SRS/operations. Metadata-only preview, live replay authorization, serialized admission commit. Final verification recorded below |
| M1-API | Coordinator; ML reviewer | High: fresh-browser resume lacked discovery and model-binding updates lacked CAS. Medium: DTO revision/precondition ambiguity | Added resume/list endpoints, ETags, aggregate preconditions, conditional binding creation/replacement and regression cases |
| M1-SCI | ML/data authors; research reviewer | Medium: structural comparison insufficiently specified and availability denominator inconsistent; low: plain-text scope mismatch | Defined combined structural comparison plus mechanism ablations; canonical probe denominator; HTML/text PDF only. Final pass |
| M1-ENG | Coordinator/security authors; data reviewer | No blocking inconsistency across system/security/roadmap/42-row traceability; minor fresh-auth wording narrower than SRS | Pass; coordinator broadened fresh-auth wording to all credential mutations as requested |

M1-SCI final reviewed SHA256 revisions:

- Supplemental evidence: `D38E3723E5CF752B3C4CBDFCCE13E92FDC8A57234402F21A2AC5847DFA769334`.
- RAG: `918E49CAB87E3308FBCDC0D7A049CE8A0A92F5DF09C304CDCCFA797A690F9EC5`.
- Testing: `7A2BC149436D9C058D2036A1A2607169A6749616C86FCB2C296C3952F3A787BE`.
- Learning: `D6D8B76156677E2EEFBED7E6E8F87F108DE491D07B310C8565D7AE1E32E1A900`.

M1-ENG final reviewed SHA256 revisions:

- System: `B3423605A6995EE17ABE6A1192BBFAD6BB27E6FC0C1740B49C83E21C3A5529B4`.
- Security before the requested wording clarification: `4AA66283917A199FAE2F349BE10D189BA82781D0B6263A7A4BDD404CBF77450A`.
- Roadmap: `3C464C8E7E2422051B630417281FA5E4BC5B2C46FE165829640C12DF69910331`.
- Traceability: `3361E747E04677C2E9250CC4CF78D06095C832BD9F2BBBE8F0D4459AB717ED14`.
- SRS: `264F816F7554FF7C1F04EC5E5E473C9617B6BAD87ABC5831A5BB183BBDCC0D8F`.

Hashes describe reviewed working bytes; Git line-ending normalization can change byte
hashes without changing text. The milestone commit fixes the published text revision.

## Validation scope and handoff

Validation uses requirement-ID coverage, relative-link existence, source inspection,
cross-document review and Git whitespace/diff checks. No application tests, load tests,
security attacks, model benchmarks, learning-effectiveness studies or recovery drills
were executed. Their acceptance suites are specified, not reported as passing.

The charter PR is a prerequisite. This milestone is intended as a stacked PR against
`codex/project-charter`; merging either requires the protected repository's review
process. Documentation completion does not authorize M02 or spending.

## Final verification

M1-SEC final disposition: pass after resolving the medium lock-order conflict found
between DATA_MODEL and SYSTEM/SECURITY. Canonical ordering is budget ledgers
(deployment/owner/credential), bindings, then credentials, with stable IDs per class.
The safety reviewer also verified the expanded fresh-auth wording. Final reviewed hashes:

- API: `87D091BE94D46DCAE376E7C642E9FE43BE5C954FE92789D7960C4C5EA4C9E67A`.
- Data: `C83EBCFF991E98A404ED2B28B4CFDE4B494FA459E3DA859F135F2930CC1CF405`.
- Operations before EOF whitespace normalization: `FA0F1CEAA8FB636B23245673E87AE018E38A9569E5080C44B9E3FEC7C30F9F23`.
- Security: `751DC0CE3E79CAE4C74159B20B55C1E87DE5FF28AEE9DF8B4BB1E3D7EE9AEA3C`.

Coordinator executed a temporary standard-library document checker: 42 unique SRS
IDs, matching traceability/test references and existing local path links across 21
Markdown files passed. All twelve requested specification paths exist. This checker
checks paths rather than anchor semantics; cross-document meaning was separately
reviewed. `git diff --cached --check` initially detected extra EOF blank lines, which
were removed; the rerun passed. Staged scope and requirements diff inspected; only
Markdown documentation is included. Primary-source claims were checked, including
visual inspection of MTRAG-UN's retrieval table. No raw papers or credentials published.
