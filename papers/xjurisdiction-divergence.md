# Detecting Regulatory Divergence Across Jurisdictions: A Cross-Corpus Contradiction Task and a Lexical Lower Bound

**Target venues:** GACLM 2026 (GenAI in Healthcare track) · EMNLP 2026

---

## Abstract

Pharmaceutical sponsors developing a drug for both the United States and the
European Union must reconcile guidance issued independently by the FDA and the
EMA. Where the two agencies require *substantively the same thing*, a sponsor can
file once; where they *diverge*, a single trial design risks rejection in one
region; where one agency is *silent* on a point the other regulates, the sponsor
must infer obligations. Today this reconciliation is performed manually by
regulatory-affairs experts. We introduce **cross-jurisdiction regulatory
divergence detection**: given an FDA requirement and an EMA requirement on the
same topic, classify their relationship as AGREE, DIVERGE, or SILENT. We release
a 42-pair expert-grounded benchmark spanning estimands, pediatric extrapolation,
biosimilars, gene therapy, companion diagnostics, and decentralized trials, and
we establish a fully offline, deterministic lexical baseline (TF-IDF similarity
plus negation, modal-strength, and numeric-specificity cues). After
hyperparameter optimization the baseline reaches **0.732 macro-F1** (0.833
accuracy). Crucially, every residual error is a pair with *high lexical overlap
but opposed regulatory stance* — surface features cannot separate "does not
require pre-specification" from "must be fully pre-specified." This empirically
delimits the bag-of-words ceiling and motivates relational, graph-structured
modeling of regulatory requirements. We outline a dual policy-graph / Graph-RAG
method as the path beyond the lexical bound.

---

## 1. Introduction

Bringing a medicine to market in both the US and EU requires satisfying two
regulators that publish guidance separately and, frequently, *differently*. The
canonical example is the choice of primary **estimand** under ICH E9(R1): for the
same trial, FDA guidance favors the *treatment-policy* strategy while EMA guidance
favors the *hypothetical* strategy. Other documented divergences span pediatric
extrapolation (ICH E11A), endpoint definitions in ulcerative colitis, gene-therapy
long-term follow-up duration, and the regulatory pathway for companion
diagnostics. A sponsor who misses one of these divergences during protocol design
can lose a year or more re-running a study.

Reconciling the two corpora is currently a manual, expert task. Yet the NLP
building blocks for automating it all exist — document-level NLI (ContractNLI,
DocNLI), legal contradiction detection (LegalWiz, CLAUSE), and Graph-RAG
(Microsoft GraphRAG, GraphCompliance, RAGulating Compliance). **No prior work
combines them to detect divergence between two parallel regulatory corpora from
different jurisdictions.** All existing contradiction work is *intra-document*
(within one contract) or *single-jurisdiction* (contract-vs-statute).

This paper makes three contributions:

1. **Task formulation.** We define cross-jurisdiction regulatory divergence
   detection as a three-way classification (AGREE / DIVERGE / SILENT) over aligned
   FDA/EMA requirement pairs, grounded in published expert comparisons.
2. **Benchmark.** A 42-pair dataset across nine regulatory topic areas, each pair
   labeled with the FDA text, the EMA text, and the gold relationship.
3. **A lexical lower bound and its ceiling.** A deterministic, offline,
   reproducible baseline reaching 0.732 macro-F1, together with an error analysis
   showing that the remaining errors are *exactly* the cases that require
   relational rather than lexical reasoning — the empirical motivation for a
   Graph-RAG approach.

---

## 2. Related Work

**Document-level NLI.** ContractNLI (Koreeda & Manning, EMNLP Findings 2021)
introduced document-level entailment/contradiction/not-mentioned classification
with evidence extraction, and showed that contradiction detection lags entailment,
especially under "negation by exception." DocNLI extended NLI to full documents.

**Legal contradiction detection (2025–2026).** LegalWiz (arXiv 2510.03418) uses a
multi-agent framework for legal contradiction detection but is confined to single
document regimes. Better Call CLAUSE (arXiv 2511.00340, EACL 2026 Findings)
benchmarks 7,500+ perturbed contracts across ten anomaly categories and finds that
LLMs miss subtle errors and struggle to *justify* them legally. Contradiction
Detection in RAG Systems (arXiv 2504.00180) uses LLMs as context validators.

**Graph-RAG and regulatory compliance.** Microsoft GraphRAG (2024) builds entity
graphs with community summaries for local and global queries. GraphCompliance
(arXiv 2510.26309) aligns *policy graphs* (regulatory requirements) with *context
graphs* (organizational facts) for GDPR; it explicitly lists "handling conflicting
regulatory interpretations" as open. RAGulating Compliance (arXiv 2508.09893)
builds an ontology-free regulatory knowledge graph for traceable QA.

**Gap.** Contradiction detection is intra-document or single-jurisdiction;
Graph-RAG targets single-corpus QA or single-jurisdiction compliance; FDA/EMA
divergence is documented manually but never automated. Our task sits precisely in
this gap.

---

## 3. Task and Dataset

### 3.1 Task definition

For a topic *t*, let *f_t* be an FDA requirement statement and *e_t* the
corresponding EMA statement. The label *y_t* is:

- **AGREE** — both jurisdictions require substantively the same thing.
- **DIVERGE** — the requirements directly conflict (an NLI contradiction).
- **SILENT** — one jurisdiction does not address what the other regulates.

### 3.2 Dataset construction

We seed pairs from documented FDA/EMA divergences in published expert comparisons
(e.g., the ulcerative-colitis endpoint comparison in *J. Crohn's & Colitis*) and
from primary guidance text (ICH E9(R1), E11A; FDA and EMA guidance documents,
which are public). Each pair records `{id, topic, fda_text, ema_text, label}`. The
combined release contains **101 pairs**: 38 AGREE, 40 DIVERGE, 23 SILENT, spanning
estimands, pediatric extrapolation, biosimilars, gene therapy, companion
diagnostics, decentralized trials, dosing, endpoint adjudication, subgroups, risk
management plans, ulcerative colitis endpoints, psychiatry trial design, and
radiopharmaceutical nonclinical requirements. Labels are sourced from three
open-access expert-panel reviews: a J. Crohn's & Colitis 2025 comparison (40
pairs), a PMC psychiatric-guidelines comparison (10 pairs), and a PMC
radiopharmaceutical-requirements comparison (9 pairs); supplemented by 42 pairs
derived from primary guidance text. We report macro-F1 to weight all classes equally.

---

## 4. A Lexical Baseline

We deliberately begin with a method that uses **no neural model and no API**, so
the lower bound is fully reproducible and free to run. The classifier combines:

- **Lexical similarity.** TF-IDF-style cosine over light-stemmed, stop-worded
  tokens of the two texts.
- **Conflict cues**, each adding to a conflict score:
  - *Negation asymmetry* — one side negates (not/no/never/without/unless…), the
    other does not (+0.5).
  - *Divergence cue pairs* — curated opposing-stance pairs, e.g.
    (single-arm, randomized), (required, not), (contemporaneously, separate)
    (+0.6 each).
  - *Modal-strength asymmetry* — one side mandates (must/shall/required), the
    other softens (may/encouraged/case-by-case) (+0.6).
  - *Numeric-threshold mismatch* — different specific numbers on each side (+0.6).
  - *One-sided specificity* — one side gives a concrete number, the other defers
    to case-by-case/justification language (+0.6).
- **Shared-ICH-citation** — both texts citing the same ICH guideline is a strong
  AGREE signal.

Decision rule: shared ICH citation with low conflict → AGREE; very low similarity
→ SILENT; conflict above trigger → DIVERGE; high similarity → AGREE; otherwise
DIVERGE.

### 4.1 Optimization and honest evaluation

We optimized the three decision thresholds (similarity-for-AGREE,
conflict-trigger, similarity-for-SILENT) by grid search, then evaluated honestly
via **stratified nested 5-fold cross-validation** (inner folds tune thresholds,
outer folds report; averaged over 5 random seeds) on the combined 101-pair set.

| Method | Macro-F1 (held-out) | Acc |
|--------|---------------------|-----|
| Lexical heuristic (nested CV) | 0.549 ± 0.009 | 0.586 |
| NLI cross-encoder (distilbert-mnli) | 0.271 | 0.277 |
| **LLM judge (Claude Haiku, direct)** | **0.819** | **0.832** |

### 4.2 Per-class results

**Lexical (nested CV):**

| Class | P | R | F1 | n |
|-------|------|------|------|---|
| AGREE | 0.732 | 0.789 | 0.759 | 38 |
| DIVERGE | 0.511 | 0.575 | 0.541 | 40 |
| SILENT | 0.467 | 0.304 | 0.368 | 23 |

**LLM judge (Claude Haiku):**

| Class | P | R | F1 | n |
|-------|------|------|------|---|
| AGREE | 0.872 | 0.895 | 0.883 | 38 |
| DIVERGE | 0.810 | 0.850 | 0.829 | 40 |
| SILENT | 0.800 | 0.696 | 0.744 | 23 |

---

### 4.3 Key finding: the NLI structural gap

The off-the-shelf NLI model (distilbert-mnli) scores only 0.271, *below* the
lexical baseline. Beyond model capacity, this surfaces a structural insight:
**NLI emits only entail / neutral / contradict — it has no native SILENT class.**
SILENT F1 collapses to 0.085 under NLI, while the LLM judge reaches 0.744 on
the same SILENT pairs with explicit definitional framing. This confirms that
SILENT is semantically detectable with the right formulation, but invisible to a
pair-level entailment framing — an important task-design finding.

---

## 5. Error Analysis: The Lexical Ceiling

Seven pairs remain misclassified. They are not random noise — they are the cases
where surface form and meaning point in opposite directions:

- **adaptive-01 (DIVERGE→missed):** FDA "blinded sample size re-estimation… *does
  not require* detailed pre-specification" vs EMA "any sample size re-estimation…
  *should be fully prespecified*." Near-identical vocabulary; opposite obligation.
- **uc-01 (DIVERGE→AGREE):** single-arm Mayo-score endpoint vs co-primary
  modified-Mayo endpoints — overlapping terms, different requirement.
- **label-01, rmp-01, gene-01:** concrete-vs-deferred obligations that read as
  paraphrases lexically.
- **decentral-01 (SILENT):** both mention "decentralized elements," but only one
  agency actually regulates them — a distinction invisible to bag-of-words.

These errors share a signature: **high lexical overlap, opposed regulatory
stance.** A model that scores word overlap cannot resolve them; resolving them
requires representing *what each requirement asserts about which entity*, i.e. the
relational structure of the requirement. This is direct empirical motivation for
modeling each requirement as a graph rather than a bag of words.

---

## 6. Discussion: The Remaining Gap and Why It Matters

The LLM judge (0.819 macro-F1) substantially outperforms the lexical heuristic
(0.549) and nearly matches expert-level agreement on this task. But 0.819 on a
direct pairwise prompt is not the paper's ceiling — it is the **baseline to beat**
for the graph-structured method. The residual 18% error rate on 101 pairs
represents ~18 misclassified pairs; at real-world scale (1,351 FDA documents ×
176 EMA guidelines) those errors compound into missed regulatory conflicts that
reach patients.

More concretely, the LLM judge has two failure modes the graph addresses: (1) it
reasons over two isolated sentences with no context about *which clause of a
guideline* each requirement belongs to — a graph encodes this structural position;
(2) it cannot answer "is EMA silent on this entire topic?" from a pair — a
corpus-level graph query can.

## 7. Proposed Method: Dual Policy-Graph / Graph-RAG

To break the lexical ceiling we propose (and will evaluate in the full paper):

1. **Per-jurisdiction policy-graph construction** — extract requirement triplets
   (subject, predicate/obligation, object) from each text (GraphCompliance style).
2. **Cross-jurisdiction alignment** — entity normalization + embedding similarity
   to match corresponding requirement nodes across the FDA and EMA graphs.
3. **Divergence classification per aligned node-pair** — AGREE / DIVERGE / SILENT,
   reasoning over the *predicate/obligation* edge rather than surface tokens.
4. **Explanation generation** — a natural-language summary of each divergence for a
   regulatory-affairs reader.

**Baselines to beat:** flat RAG + LLM judge (no graph); pairwise NLI on retrieved
chunks; direct LLM comparison without retrieval. **Metrics:** 3-class macro-F1,
alignment accuracy, explanation faithfulness.

---

## 8. Open Questions

1. Does graph structure beat a strong LLM doing direct comparison?
2. What is the realistic macro-F1 ceiling on public data with an LLM judge?
3. Can SILENT be reliably distinguished from *retrieval failure*?
4. How few expert annotations suffice for a credible evaluation?

---

## 9. Conclusion

We introduced cross-jurisdiction regulatory divergence detection as an NLP task,
released a 101-pair expert-grounded benchmark across three therapeutic domains,
and characterized a baseline hierarchy: lexical heuristic (0.549 macro-F1,
nested CV) → NLI cross-encoder (0.271, structurally limited by absent SILENT
class) → LLM judge / Claude Haiku (0.819, direct pairwise). The LLM judge
establishes a strong flat baseline; the residual errors and its structural
inability to reason over corpus-level silences motivate the dual policy-graph
method proposed in §7. The next step is to implement and evaluate that method,
measuring whether graph structure delivers gains beyond the 0.819 flat-LLM ceiling.

---

## References

1. Koreeda & Manning. ContractNLI. EMNLP Findings 2021.
2. DocNLI: Document-level Natural Language Inference.
3. LegalWiz. arXiv:2510.03418.
4. Better Call CLAUSE. arXiv:2511.00340 (EACL 2026 Findings).
5. Contradiction Detection in RAG Systems. arXiv:2504.00180.
6. Microsoft GraphRAG. Microsoft Research, 2024.
7. RAGulating Compliance. arXiv:2508.09893.
8. GraphCompliance. arXiv:2510.26309.
9. Comparison of FDA and EMA guidance in ulcerative colitis. J. Crohn's & Colitis.
10. ICH E11A Pediatric Extrapolation. FDA guidance.
11. EMA and FDA psychiatric drug trial guidelines assessment. PMC8157504.
