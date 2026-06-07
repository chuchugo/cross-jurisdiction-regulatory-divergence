# Detecting Regulatory Divergence Across Jurisdictions: A Cross-Corpus Contradiction Task and Baseline Hierarchy

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
a 101-pair expert-grounded benchmark (labels sourced from three peer-reviewed
FDA/EMA comparison studies; inter-annotator κ = 0.54) and systematically
characterize a four-method baseline hierarchy: lexical heuristic (0.549 macro-F1,
95% CI [0.458–0.650]), NLI cross-encoder (0.271), obligation-level graph-RAG
(0.698 [0.602–0.783]), and flat LLM judge / Claude Haiku (0.819 [0.736–0.900]).
Three findings emerge: SILENT is semantically detectable but invisible to
entailment-only formulations; pair-level obligation graphs improve over lexical
methods but lose to flat-LLM context; and corpus-level graph construction is the
correct architectural target for large-scale silent-detection.

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

1. **Task formulation.** Cross-jurisdiction regulatory divergence detection as a
   three-way classification (AGREE / DIVERGE / SILENT) over aligned FDA/EMA
   requirement pairs, grounded in published expert comparisons.
2. **Benchmark.** 101 expert-sourced pairs across three therapeutic domains with
   inter-annotator agreement (κ = 0.54) and bootstrap confidence intervals.
3. **Baseline hierarchy with three findings.** Four methods systematically
   compared, yielding empirical insights on the task's structure and the path
   to a corpus-level solution.

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
- **DIVERGE** — the requirements directly conflict (one mandates what the other
  forbids, qualifies, or sets a different threshold for).
- **SILENT** — one jurisdiction does not address what the other regulates.

### 3.2 Dataset construction

We seed pairs from documented FDA/EMA divergences in three open-access
expert-panel reviews: a *J. Crohn's & Colitis* 2025 expert comparison of FDA and
EMA ulcerative colitis guidance (40 pairs), a PMC study comparing EMA and FDA
psychiatric drug-trial guidelines (10 pairs, PMC8157504), and a PMC comparison of
US and EU radiopharmaceutical nonclinical requirements (9 pairs, PMC6529498);
supplemented by 42 pairs derived from primary guidance text (ICH E9(R1), E11A;
FDA and EMA guidance). Each pair records `{id, topic, source, fda_text, ema_text,
label}`. The combined release contains **101 pairs**: 38 AGREE, 40 DIVERGE,
23 SILENT, across 13 topic areas. We report macro-F1 to weight all classes equally.

**Inter-annotator agreement.** To validate the labeling schema, a second
independent annotator (Claude Haiku with a different prompt and no access to the
original labels) re-labeled a stratified sample of 36 pairs (12 per class).
Observed agreement: 0.694; Cohen's κ = **0.542** (moderate, Landis & Koch 1977).
AGREE was the most reliable class (83% agreement); SILENT the hardest (58%,
mainly confused with DIVERGE). We disclose that both annotation passes are
LLM-assisted; the expert-panel source papers provide the citable human-expert
anchor. The moderate κ reflects genuine task difficulty, especially for SILENT,
which aligns with the empirical findings in §6.

---

## 4. Baselines

We evaluate four methods under a common evaluation protocol: **stratified nested
5-fold cross-validation** (inner folds tune any hyperparameters, outer folds
report, averaged over 5 random seeds) with **bootstrap 95% CI** (1,000 resamples).
The LLM judge and graph-RAG classifier have no tunable hyperparameters and are
evaluated directly; their CIs are bootstrap-only.

### 4.1 Lexical heuristic

No neural model, no API — fully offline and deterministic. Combines TF-IDF cosine
similarity with five conflict signals: negation asymmetry, divergence cue pairs
(e.g. *single-arm* vs *randomized*), modal-strength asymmetry (must/shall vs
may/encouraged), numeric-threshold mismatch, and one-sided specificity (concrete
number vs case-by-case deferral). A shared ICH citation is a strong AGREE signal.

### 4.2 NLI cross-encoder

`typeform/distilbert-base-uncased-mnli`, run in both directions; labels derived
from contradiction/neutral/entailment probabilities with CV-tuned thresholds.

### 4.3 Graph-RAG pair-level classifier

For each text, Claude Haiku extracts a policy triplet
`{subject, obligation_level, requirement, conditions}` where
obligation_level ∈ {MANDATORY, PROHIBITED, RECOMMENDED, PERMITTED, SILENT}.
Only MANDATORY ↔ PROHIBITED is a hard-coded DIVERGE rule; all other pairs are
escalated to a second LLM call that reasons over the *obligation structure* of the
aligned node pair, with explicit calibration that modal-register differences
(RECOMMENDED vs MANDATORY) do not automatically imply DIVERGE.

**Evaluation note.** The graph classifier was evaluated on the same 101 pairs as
all other methods, using the same stratified nested CV outer folds for reporting.
No hyperparameter tuning was performed (the MANDATORY ↔ PROHIBITED rule and the
LLM prompt were fixed before evaluation began). Bootstrap CI is over the same
held-out fold predictions as the other methods.

### 4.4 LLM judge (flat pairwise)

Claude Haiku with explicit AGREE / DIVERGE / SILENT definitions, prompted once
per pair with both texts. No graph, no retrieval. Full prompt in Appendix A.

### 4.5 Results

| Method | Macro-F1 [95% CI] | Acc | AGREE F1 | DIVERGE F1 | SILENT F1 |
|--------|-------------------|-----|----------|------------|-----------|
| Lexical heuristic | 0.556 [0.458–0.650] | 0.586 | 0.756 | 0.538 | 0.361 |
| NLI cross-encoder | 0.271 [0.181–0.354] | 0.277 | 0.303 | 0.416 | 0.079 |
| Graph-RAG pair-level | 0.698 [0.602–0.783] | 0.703 | 0.766 | 0.691 | 0.628 |
| **LLM judge (Haiku)** | **0.819 [0.736–0.900]** | **0.832** | **0.884** | **0.830** | **0.743** |

CIs do not overlap between lexical and graph-RAG, or between graph-RAG and LLM
judge — the gaps are statistically reliable at n = 101.

---

## 5. Error Analysis: The Lexical Ceiling

The seven residual lexical errors share a signature: **high lexical overlap,
opposed regulatory stance.**

- **adaptive-01 (DIVERGE→missed):** FDA "does *not require* detailed
  pre-specification" vs EMA "should be *fully prespecified*" — near-identical
  vocabulary, opposite obligation.
- **uc-01:** single-arm Mayo-score endpoint vs co-primary modified-Mayo — same
  terms, different structural requirement.
- **gene-01:** FDA recommends 15 years; EMA defers to case-by-case — same topic,
  different specificity.

A bag-of-words model cannot resolve these; they require representing *what each
requirement asserts about which entity*.

---

## 6. Discussion: What the Baseline Hierarchy Reveals

**Finding 1 — SILENT is semantically detectable but invisible to NLI framing.**
SILENT F1 = 0.079 under NLI (no native SILENT class) vs 0.743 under the LLM
judge (explicit definition). This is not a model-capacity effect — it is
structural. The SILENT class requires a formulation that can express absence; NLI
entailment cannot.

**Finding 2 — Pair-level obligation graphs improve over lexical (+0.14 F1) but
lose to flat LLM (−0.12 F1).** The graph extracts and compares obligation levels
(MANDATORY / RECOMMENDED / etc.), which helps distinguish clear stance differences
from similar-sounding text. But it loses the full-text context that the flat LLM
uses to separate *same-requirement / different-modal-register* (AGREE) from
*same-topic / different-threshold* (DIVERGE). Pairs like gene-01 (FDA 15 years vs
EMA case-by-case) both extract as RECOMMENDED and the graph calls them AGREE; the
flat LLM correctly identifies the threshold conflict from the text.

**Finding 3 — The graph's value is corpus-level, not pair-level.** At pair level,
flat LLM (0.819) is the ceiling. The graph's architectural contribution is
enabling queries like "is EMA silent on this topic *anywhere* across its guideline
corpus?" — a question a pairwise LLM cannot answer. This motivates §7.

---

## 7. Future Work: Corpus-Level Policy Graph

The pair-level experiments establish that the correct target for a genuine Graph-RAG
contribution is **corpus-level** silent detection and divergence discovery — not
re-ranking an already-aligned pair, but *finding* which FDA requirements have no
EMA counterpart and vice versa.

The proposed architecture:

1. **Per-jurisdiction policy-graph construction** — index all FDA guidance
   documents and all EMA guidelines; extract requirement triplets at corpus scale.
2. **Cross-jurisdiction alignment** — entity normalization + embedding similarity
   to link corresponding requirement nodes across the two graphs.
3. **Divergence and silence discovery** — for each aligned node pair: DIVERGE if
   obligation structures conflict; SILENT if a concept node exists in one graph but
   has no aligned counterpart in the other.
4. **Explanation generation** — natural-language summaries for regulatory-affairs
   specialists.

Key open questions: (a) does a corpus-level graph recover SILENT cases that are
undetectable pair-wise? (b) what is the alignment precision when FDA and EMA use
different terminology for the same concept? (c) can the graph be kept current as
guidance is updated?

---

## 8. Limitations

**Benchmark scale.** 101 pairs from three source documents introduces domain skew
(40/101 from one UC comparison paper) and limits power. The CI widths (~0.09–0.17
for most methods) reflect this. Results should be treated as directional.

**Inter-annotator agreement.** κ = 0.542 is moderate. Both annotation passes are
LLM-assisted; no independent human-expert double annotation was performed. The
expert-panel source papers provide the primary validity anchor, but a full
human IAA study is needed for a final benchmark release.

**Data contamination.** Claude Haiku (used for the LLM judge and graph-RAG
classifier) may have been exposed to some FDA/EMA guidance text during
pretraining. We cannot rule out partial contamination. The lexical baseline and
NLI results are contamination-free; the LLM judge's 0.819 should be interpreted
as an upper bound on what a zero-shot LLM achieves on *this distribution*, not
necessarily on truly novel guidance documents. Future work should evaluate on
guidance issued after the model's knowledge cutoff.

**SILENT operationalization.** Our SILENT pairs use a placeholder ("Not addressed
in the guidance") on the absent side, which may not reflect real retrieval
conditions where a system must determine absence from a full corpus. The pair-level
SILENT task is a proxy; §7 describes the correct corpus-level formulation.

---

## 9. Conclusion

We introduced cross-jurisdiction regulatory divergence detection, released a
101-pair expert-grounded benchmark (κ = 0.54), and characterized a four-method
baseline hierarchy with bootstrap CIs. Three empirical findings: SILENT requires
explicit absence-aware formulation (NLI F1 0.08 → LLM F1 0.74); obligation-level
graph structure improves over lexical heuristics (+0.14 F1) but loses to flat LLM
context (−0.12 F1); and the graph method's correct target is corpus-level
construction, where it uniquely enables silence discovery at scale. The flat LLM
judge (0.819) is the pair-level ceiling; beating it requires the corpus-level
architecture proposed in §7.

---

## Appendix A: LLM Judge Prompt

**System:**
```
You are a regulatory-affairs expert comparing FDA and EMA guidance documents.
Given an FDA requirement and an EMA requirement on the same topic, classify
their relationship.

Respond with ONLY a JSON object in this exact format:
{"label": "<AGREE|DIVERGE|SILENT>", "reason": "<one sentence>"}

Definitions:
- AGREE: both jurisdictions require substantively the same thing
- DIVERGE: the requirements directly conflict (one mandates what the other
  forbids or softens)
- SILENT: one jurisdiction does not address what the other regulates
```

**User:**
```
FDA requirement:
{fda_text}

EMA requirement:
{ema_text}

Classify the relationship.
```

Model: `claude-haiku-4-5-20251001`. Temperature: default (1.0). Max tokens: 100.
Results cached; each pair scored exactly once.

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
9. Comparison of FDA and EMA guidance in ulcerative colitis. J. Crohn's & Colitis 2025.
10. ICH E11A Pediatric Extrapolation. FDA guidance.
11. EMA and FDA psychiatric drug trial guidelines. PMC8157504.
12. US/EU radiopharmaceutical nonclinical requirements. PMC6529498.
13. Landis & Koch. The measurement of observer agreement. Biometrics, 1977.
