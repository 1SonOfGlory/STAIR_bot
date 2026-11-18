# Requirements traceability

Date: 2026-11-17. Status: specification coverage, not implementation completion.
Every row refers to the unique definition in [SRS](../requirements/SRS.md). Tests are
specified in [TEST_STRATEGY](../testing/TEST_STRATEGY.md). M02-M08 are defined in
[ROADMAP](ROADMAP.md). All implementation/evidence status is **not run / not built**.
No checkmark in a document substitutes for test output or reviewer acceptance.

| Requirement | Charter capability | Accountable module | Primary specification | Gate milestone | Acceptance suite |
| --- | --- | --- | --- | --- | --- |
| FR-001 | P11 | Identity | SECURITY_MODEL | M02 | T-FR-001 |
| FR-002 | P11 | Identity | API_CONTRACTS | M02 | T-FR-002 |
| FR-003 | P01,P11 | Workspaces | API_CONTRACTS | M02 | T-FR-003 |
| FR-004 | P01 | Learning | LEARNING_ENGINE | M05 | T-FR-004 |
| FR-005 | P02,P03 | Sources | RAG_ARCHITECTURE | M03 | T-FR-005 |
| FR-006 | P03,P04 | Sources | RAG_ARCHITECTURE | M03 | T-FR-006 |
| FR-007 | P04 | Acquisition | SECURITY_MODEL | M03 | T-FR-007 |
| FR-008 | P04,P06 | Jobs/Processing | SYSTEM_ARCHITECTURE | M03 | T-FR-008 |
| FR-009 | P05 | Knowledge | RAG_ARCHITECTURE | M03 | T-FR-009 |
| FR-010 | P06 | Knowledge | DATA_MODEL | M03 | T-FR-010 |
| FR-011 | P06 | Retrieval | RAG_ARCHITECTURE | M04 | T-FR-011 |
| FR-012 | P08 | Grounding | RAG_ARCHITECTURE | M04,M05 | T-FR-012 |
| FR-013 | P07 | Learning | LEARNING_ENGINE | M05 | T-FR-013 |
| FR-014 | P08,P11 | Tutoring | API_CONTRACTS | M05 | T-FR-014 |
| FR-015 | P09 | Assessment | LEARNING_ENGINE | M06 | T-FR-015 |
| FR-016 | P09 | Assessment | LEARNING_ENGINE | M06 | T-FR-016 |
| FR-017 | P10 | Execution | SECURITY_MODEL | M07 | T-FR-017 |
| FR-018 | P11 | Progress | LEARNING_ENGINE | M06 | T-FR-018 |
| FR-019 | P11 | Workspaces/Learning | DATA_MODEL | M05,M06 | T-FR-019 |
| FR-020 | P12 | Gateway | SYSTEM_ARCHITECTURE | M02,M04,M08 | T-FR-020 |
| FR-021 | P12 | Gateway | SECURITY_MODEL | M02 | T-FR-021 |
| FR-022 | P12 | Gateway | OPERATIONS | M02,M08 | T-FR-022 |
| FR-023 | P02,P06 | Sources | RAG_ARCHITECTURE | M03 | T-FR-023 |
| FR-024 | P06 | Privacy | DATA_MODEL | M03,M08 | T-FR-024 |
| FR-025 | P11,P12 | Privacy | SECURITY_MODEL | M08 | T-FR-025 |
| FR-026 | P06,P11 | Jobs | OPERATIONS | M02,M03,M08 | T-FR-026 |
| FR-027 | P08,P09 | Learning | API_CONTRACTS | M05,M06 | T-FR-027 |
| FR-028 | P01-P12 | Frontend | PRD | M05,M08 | T-FR-028 |
| FR-029 | P06,P11,P12 | All owned modules | SECURITY_MODEL | M02-M08 | T-FR-029 |
| FR-030 | P02,P04 | Sources | API_CONTRACTS | M03 | T-FR-030 |
| NFR-001 | All | Security/each implementer | SECURITY_MODEL | M02-M08 | T-NFR-001 |
| NFR-002 | All | Privacy | DATA_MODEL | M03,M08 | T-NFR-002 |
| NFR-003 | P06,P11 | Jobs/each implementer | SYSTEM_ARCHITECTURE | M02-M08 | T-NFR-003 |
| NFR-004 | All | Operations | OPERATIONS | M08 | T-NFR-004 |
| NFR-005 | P04,P06,P08 | Operations/ML | TEST_STRATEGY | M04,M08 | T-NFR-005 |
| NFR-006 | All | Operations | OPERATIONS | M08 | T-NFR-006 |
| NFR-007 | P12 | Gateway/Operations | OPERATIONS | M02,M08 | T-NFR-007 |
| NFR-008 | All | Engineering | SYSTEM_ARCHITECTURE | M02-M08 | T-NFR-008 |
| NFR-009 | All | Frontend/Product | TEST_STRATEGY | M05,M08 | T-NFR-009 |
| NFR-010 | P05-P11 | ML/Data | TEST_STRATEGY | M03-M08 | T-NFR-010 |
| NFR-011 | All | Operations | OPERATIONS | M02,M08 | T-NFR-011 |
| NFR-012 | P06,P11 | Operations/Privacy | OPERATIONS | M08 | T-NFR-012 |

Document key: [PRD](../product/PRD.md), [SYSTEM_ARCHITECTURE](../architecture/SYSTEM_ARCHITECTURE.md),
[RAG_ARCHITECTURE](../architecture/RAG_ARCHITECTURE.md), [DATA_MODEL](../architecture/DATA_MODEL.md),
[API_CONTRACTS](../architecture/API_CONTRACTS.md), [SECURITY_MODEL](../architecture/SECURITY_MODEL.md),
[LEARNING_ENGINE](../learning/LEARNING_ENGINE.md), [OPERATIONS](../operations/OPERATIONS.md).
Every FR also has an API/UI flow or internal enforcement boundary in API_CONTRACTS;
NFRs are validated through those same flows rather than separate cosmetic endpoints.

For implementation, extend each milestone evidence record with requirement ID, test command,
fixture/config hashes, actual output/artifact, commit, reviewer, disposition and remaining
limitation. One failed mandatory row blocks its milestone. When a requirement changes,
review the linked schema/API, tests and dependent milestones together. Cross-cutting
requirements recur intentionally; a feature-local pass is not a full-release pass.
