# Additional IBM evidence for V1 specifications

Reviewed 2026-11-18. These sources inform proposed engineering choices; no experiments
were reproduced, datasets downloaded, or vendor services purchased. Embedded prompts
are research material, not instructions to this project.

## MTRAG-UN: conversation failures

The supplied `2602.23184v1.pdf` is **MTRAG-UN: A Benchmark for Open Challenges in
Multi-Turn RAG Conversations**, not the hyperparameter paper. Artifact SHA256:
`687b2fc0e9a958e1e3c0f3cdc2771e86e4389cb2ddf9fe3318377f742e476b9a`.
The seven-page paper, including appendices, was inspected. Canonical source:
[Rosenthal et al., v1, 26 February 2026](https://arxiv.org/abs/2602.23184v1).

Sections 2-3 describe 666 tasks across six domains, deliberately emphasizing difficult
turns. Distinguish missing evidence, ambiguous intent, dependence on earlier turns,
and requests to clarify or correct an earlier response. Table 2 reports improvements
from query rewriting for all four tested retrieval methods; Table 3 reports Elser
Recall@5 of 0.39 versus 0.52 for nonstandalone questions. This is benchmark retrieval
performance, not learner proficiency or answer factuality. Generation remains difficult.
The limitations section identifies English-only coverage and potential annotation-tool
bias. The small clarification-judge validation does not certify general judge accuracy.

Engineering inference: add conversation-stratified tests, explicit clarification, and
fresh retrieval for corrections. Previous assistant text cannot become source evidence.
Evaluate rewriting against simpler history handling before adding a paid model call.

## Hyperparameter optimization

[IBM publication page](https://research.ibm.com/publications/an-analysis-of-hyper-parameter-optimization-methods-for-retrieval-augmented-generation)
lists the AAAI 2026 workshop publication. For methodology use
[Orbach et al., arXiv v3, 31 December 2025, sections 3-4](https://arxiv.org/html/2505.03452v3).
Version matters: v3 uses three optimization metrics; the IBM abstract describes two.
The study compares five algorithms across five datasets and 162 configurations.
Model-first greedy and random search perform well in that bounded search space.
Metric choice changes preferred models; indexing reuse affects cost. Some test sets
contain only 30 examples. Larger spaces and agentic pipelines remain outside this
experiment. Its results do not establish a universal ten-trial tuning budget.

Engineering inference: begin with a small predeclared development search, comparing
model-first greedy with budget-matched random search when experimentation is approved.
Account for indexing, generation, judging, latency and monetary cost; choose the
objective before searching. Keep confirmatory test data inaccessible to selection.
Use separate grounding, adequacy and abstention criteria rather than optimizing a
single lexical score. Freeze search limits and review the protocol before execution.

## IBM storage-scale article

[Peter Hess, IBM, 13 April 2026](https://research.ibm.com/blog/cas-100-billion-vector-storage-ai)
reports a prototype over 100 billion synthetic 384-dimensional vectors. Its single
query server uses separate enterprise storage; indexing uses six H200 GPUs. Reported
mean query latency is 694 ms with a vendor-described 90% retrieval measure against
brute-force ground truth. This is not p95 latency, factuality, or an end-to-end tutoring
measurement. The article's lower latency and shorter build-time ambitions are future
targets. It reports 153 TiB for vectors and index, emphasizing the specialized setup.

Engineering inference: retain PostgreSQL/pgvector as the V1 candidate within E1.
Separate stable passage IDs and provenance from replaceable index generations so a
future measured bottleneck can justify a different backend. The article supplies no
evidence for purchasing this infrastructure or predicting our latency or costs.

## Adoption boundary

These sources refine [retrieval](../architecture/RAG_ARCHITECTURE.md),
[learning](../learning/LEARNING_ENGINE.md), and
[validation](../testing/TEST_STRATEGY.md). They neither approve a final model nor
authorize benchmarking. Dataset/code licenses need separate inspection before reuse;
publication access alone does not authorize incorporating material into a consumer app.
