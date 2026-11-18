# STAIR_bot

A planned consumer AI learning platform informed by IBM's STAIR retrieval research.
Learners describe a goal, approve learning resources, and follow a source-grounded
pathway with interactive teaching, practice, assessment, and persistent progress.

Status: charter and Version 1 engineering specifications prepared for review. No application, retriever,
model training, benchmark reproduction, or production deployment has been implemented.
The retrieval architecture remains undecided. A structure-aware hybrid retriever would
be an adaptation of STAIR's ideas, not a reproduction of its trained generative index.

## Project documents

- [Permanent agent instructions](AGENTS.md)
- [Project charter and scope](docs/charter.md)
- [Engineering process and release gates](docs/engineering.md)
- [STAIR evidence and architectural implications](docs/research/stair-review.md)
- [Decision log](docs/decisions/README.md)
- [Milestone 0 evidence and handoff](docs/milestones/000-charter.md)

GitHub repository: [1SonOfGlory/STAIR_bot](https://github.com/1SonOfGlory/STAIR_bot).
Changes use feature branches and pull requests. Implementation requires approval of
the next milestone. Current spending authorization is $0.

Documentation was prepared with AI assistance and is subject to human review.
The repository's existing [MIT license](LICENSE) applies to repository work; it does
not grant rights to third-party learning resources or the STAIR paper.


## Version 1 specifications

- [Product requirements](docs/product/PRD.md)
- [Software requirements and proposed targets](docs/requirements/SRS.md)
- [System architecture](docs/architecture/SYSTEM_ARCHITECTURE.md)
- [Retrieval architecture](docs/architecture/RAG_ARCHITECTURE.md)
- [Data model](docs/architecture/DATA_MODEL.md)
- [API contracts](docs/architecture/API_CONTRACTS.md)
- [Security model](docs/architecture/SECURITY_MODEL.md)
- [Learning engine](docs/learning/LEARNING_ENGINE.md)
- [Test strategy](docs/testing/TEST_STRATEGY.md)
- [Operations](docs/operations/OPERATIONS.md)
- [Implementation roadmap](docs/planning/ROADMAP.md)
- [Requirements traceability](docs/planning/REQUIREMENTS_TRACEABILITY.md)
- [Additional IBM evidence](docs/research/ibm-rag-additional-evidence.md)
- [Milestone 1 validation and review](docs/milestones/001-specifications.md)
