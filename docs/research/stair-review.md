# STAIR: evidence and architectural implications

Review date: 2026-11-17. Owner: coordinator. Scope: supplied paper in full, with
bounded primary-source checks; no experiments or dataset downloads. This is an
architectural assessment, not a systematic literature review or reproduction.

## Source identity and coverage

Vineet Kumar, Meghanadh Pulivarthi, Vishwajeet Kumar, Jaydeep Sen, Riyaz Ahmad Bhat,
and Sachindra Joshi. *STAIR (STructure Aware Information Retriever): A novel dataset
and LLM based retriever for document structure augmentation.* IBM affiliation;
first-author footnote says work done at IBM, subsequently at Amazon Books Science.
The supplied PDF identifies arXiv:2609.03874v1, 3 September 2026.

- Supplied filename: `2609.03874v1.pdf`; 11 pages; 4,339,869 bytes.
- SHA-256: `3bd184d873154b105b60d16c07f6f6ec3b800b582d4b50ada034ab85245dda8b`.
- [Version-specific source](https://arxiv.org/abs/2609.03874v1).
- Read coverage: pages 1-11, sections 1-8, Tables 1-4, Figures 1-4, and references.
  Text extraction used bundled pypdf; page 8 was rendered to check Table 4 and plots.
- The [current arXiv record](https://arxiv.org/abs/2609.03874), checked on the review
  date, lists an October 13 v2 revision. This assessment remains pinned to the supplied
  bytes; equivalence to the hosted v1 PDF has not been verified. The supplied Table 4
  includes a Haiku 4.5 row not described among section 5.1's baselines.

Search scope: direct opens of the STAIR record and the RAPTOR primary paper abstract;
no search queries or broad novelty search. Inclusion criterion: the supplied primary
source and its directly relevant hierarchical retrieval alternative. The paper's linked
anonymous code/data repository was not inspected, and reproducibility is not certified.
Instructions in examples (such as the section-selection prompt) were treated as data.

## Method and experimental setup reported by the paper

Sections 3-4 (pages 3-5): SearchTome uses 18 open textbooks, three each in education,
finance, law, medicine, natural sciences, and social sciences. PDFs are parsed and
cleaned into ToC hierarchies with content mapped to leaves. Mixtral 8x7B generates
questions from paragraphs; questions are allocated to training, development, and test
sets. The target is the leaf section capable of answering a question, given a book.
This is within-book section retrieval, not end-to-end cross-library tutoring.

STAIR ingests corpus knowledge into model parameters through query/section training.
The prompt includes the query and complete book ToC; the model generates the leaf
section identifier/title. Figure 2 explicitly includes constrained generation at
inference, restricting output to valid ToC leaves. STAIR is not simply embedding
headings or performing an untrained traversal of a document tree.

Section 5 (pages 5-6): Mistral Instruct v0.2 is fine-tuned using LoRA rank 16 and alpha
32, for up to 200 epochs with dev Recall@1 early stopping and patience 20. STAIR's
maximum input is 14,000 tokens, DSI's 512; maximum output is 64 tokens. Baselines are
out-of-box Mistral, BM25 via Elasticsearch 8.11.2, out-of-box NV-Embed-v2 (called DPR
in this paper, with passage/query lengths 512/256), and corpus-fine-tuned DSI.
Metrics are Recall@1, Recall@3, and nDCG@3, computed using BEIR. The paper explicitly
describes DSI training per book; the evaluation assumes book-specific corpus knowledge.

## Results and their meaning

Table 4, page 8, reports the following averages (percent scale):

| Method | Recall@1 | Recall@3 | nDCG@3 |
| --- | ---: | ---: | ---: |
| Mistral | 13.8 | 16.4 | 15.4 |
| BM25 | 59.5 | 77.6 | 70.1 |
| Haiku 4.5 | 45.6 | 64.1 | 56.4 |
| DPR / NV-Embed-v2 | 68.7 | 85.4 | 78.6 |
| DSI | 76.9 | 85.3 | 81.9 |
| STAIR | 82.6 | 90.8 | 87.5 |

These are author-reported results, not measured platform performance. The R@1 gain
over DSI is 5.7 percentage points, approximately 7.4% relative. Section 6 reports
randomization tests at p < 0.05 for each of six domains. It does not provide exact
p-values, confidence intervals, or a multiple-comparison treatment there. Significance
on these data does not establish generalization to all corpora or isolate every cause.

Section 6.1 defines hallucination as generating a non-leaf/invalid document identifier.
The abstract says less than 0.05%; section 6.2 reports 0.05% non-leaf predictions.
Neither measures factual hallucination in generated teaching answers. Figure 3 orders
books by average training examples (highest first); Figure 4 orders leaves by training
examples and plots R@1 with a sliding window of 100. They support the authors' claim
of better behavior for sparsely represented sections, not zero-shot corpus adaptation.
Table 3 provides illustrative cases, not an independent estimate of error frequency.

## Limitations and unresolved evidence

Author-stated (sections 7-8, pages 8-9): evaluation requires existing global structure.
Inducing structure for unseen corpora, enterprise scale potentially reaching millions
of URLs, zero-shot retrieval, and iterative multi-hop reasoning are future work.

Reviewer inferences and reporting gaps:

- Synthetic questions and shared book content across splits do not demonstrate
  generalization to human learner questions or unseen books. Paragraph-level semantic
  overlap and duplicate leakage cannot be assessed without auditing split construction.
- A known book and one gold leaf simplify retrieval compared with choosing across many
  changing resources, handling missing ToCs, or collecting evidence from multiple sections.
- Dense retrieval is out of the box and truncated while STAIR/DSI receive corpus-specific
  supervision. There is no tested lexical+dense hybrid with reranking in these results.
  This is not evidence that STAIR beats every practical retrieval configuration.
- Figure 2's constrained decoding and the nonzero invalid-ID figures leave uncertainty
  about which decoding settings apply to each result. Section 6.2's 18.67% error rate
  is also not the simple complement of Table 4's 82.6% average R@1; aggregation or
  denominators need clarification. Do not silently reconcile these into a new metric.
- Training cost, update/delete cost, online latency, hardware, and production throughput
  are not sufficiently established for this product. No learning-outcome evaluation,
  generated-answer faithfulness study, privacy audit, or execution-security evidence is given.

## Architecture options for dynamic learner libraries

| Candidate | Rationale | Key uncertainty/cost | Current disposition |
| --- | --- | --- | --- |
| Original trained STAIR | Strong supplied benchmark results; uses global ToC | Corpus-specific supervision, model updates/deletion, full-ToC context, multi-book routing | Retain as reference; do not select for V1 yet |
| Structure-aware hybrid adaptation | Explicit source/section storage plus lexical and dense retrieval, optional reranking and parent/neighbor context | Extraction quality, fusion/reranking choices, no measured advantage here | Leading candidate for later evaluation, not a validated winner |
| Simpler lexical or dense retrieval | Lower conceptual complexity; necessary baselines | May miss global structure or semantic matches | Required comparison candidates |
| RAPTOR-style summary hierarchy | Primary work explores retrieval at multiple abstraction levels | Summary errors, construction/update cost, source anchoring | Consider if later multi-hop needs justify it |

[RAPTOR](https://arxiv.org/abs/2401.18059) builds a tree using embedding, clustering,
and recursive summarization and reports gains on its evaluated QA tasks. Its hierarchy
is generated rather than the original document ToC. Its results are not a head-to-head
comparison on this platform; they do not establish superiority here. Only its primary
abstract was inspected in this bounded comparison.

Engineering inference: mutable explicit indexes appear easier to update and delete
than corpus knowledge stored in model weights. Preserve original document hierarchy,
stable internal section IDs, parent relationships, source versions, original headings,
page/URL anchors, and permission metadata regardless of eventual retrieval method.
Allow missing/uncertain structure and label inferred headings as inferred. Large
sections may need smaller passages retaining ancestor context. Generated summaries
must never replace authoritative source text. These are design recommendations only.

## Evidence needed before selection

After human approval, write and independently review a bounded evaluation protocol.
Compare simple lexical, dense, hybrid without structure, and hybrid with structure
on the same corpus, candidates, context budgets, and held-out questions. Include
cross-resource routing, new/updated/deleted documents, weak structure, ambiguous
headings, unanswerable questions, and multi-section answers. Use independently checked
human-relevant labels; group related questions/paragraphs to control leakage and keep
an unseen-document test. Separate retrieval R@k/nDCG, invalid identifiers, citation
support, answer correctness, and abstention. Measure ingestion/update/delete time,
p50/p95 latency, memory, and cost. Report uncertainty and per-format results.

Choose the simplest method meeting predeclared product targets, not the newest method.
Targets, sample sizes, hardware capacity, and budget feasibility remain unresolved.
Original STAIR reproduction would require a separately approved compute plan; no such
run is authorized or attempted under the present $0 budget. No protocol is approved here.
