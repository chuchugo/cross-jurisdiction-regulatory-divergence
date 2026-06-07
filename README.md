# Cross-Jurisdiction Regulatory Divergence Detection

**Detecting where FDA and EMA guidance diverges — automatically.**

This repository contains the paper, research outputs, and deep-research briefs for:

> **Detecting Regulatory Divergence Across Jurisdictions: A Cross-Corpus Contradiction Task and Baseline Hierarchy**
> Target venues: GACLM 2026 (GenAI in Healthcare track) · EMNLP 2026

---

## What this is

Pharmaceutical sponsors developing a drug for both the US and EU must reconcile two
independent regulatory corpora — FDA guidance and EMA guidelines — that frequently
diverge. This project introduces **cross-jurisdiction regulatory divergence detection**
as an NLP task: given an FDA requirement and an EMA requirement on the same topic,
classify their relationship as **AGREE**, **DIVERGE**, or **SILENT**.

---

## Repository structure

```
papers/
  xjurisdiction-divergence.md   — full paper draft (task, benchmark, baselines, findings)

outputs/
  xjurisdiction-contradiction-graphrag.md   — deep-research brief: prior art + method design
  rag-lifescience-regulatory.md             — deep-research brief: RAG in life sciences
  rag-regulatory-topics.md                  — deep-research brief: topic selection
  *.provenance.md                           — source provenance for each brief

.agents/skills/feynman/                     — research workflow skills (deepresearch, autoresearch)
```

The **benchmark code, dataset, and all experiments** live in a companion repo:
👉 [chuchugo/cross-jurisdiction-divergence-experiments](https://github.com/chuchugo/cross-jurisdiction-divergence-experiments)

---

## Key results

Four baselines evaluated on 101 expert-sourced FDA/EMA requirement pairs (bootstrap 95% CI):

| Method | Macro-F1 [95% CI] | Notes |
|--------|-------------------|-------|
| Lexical heuristic | 0.556 [0.458–0.650] | fully offline, no API |
| NLI cross-encoder (distilbert-mnli) | 0.271 [0.181–0.354] | SILENT structurally invisible |
| Graph-RAG pair-level | 0.698 [0.602–0.783] | obligation-level triplet extraction |
| **LLM judge (Claude Haiku)** | **0.819 [0.736–0.900]** | flat pairwise; no graph |

**Three findings:**
1. SILENT is semantically detectable but invisible to NLI entailment framing (F1 0.08 → 0.74)
2. Obligation-level graph structure improves over lexical (+0.14 F1) but loses to flat LLM context (−0.12 F1)
3. The graph method's correct target is corpus-level construction, not pair-level reranking

Inter-annotator agreement (second-annotator Claude Haiku, different prompt, n=36): **κ = 0.542** (moderate)

---

## Dataset

101 expert-grounded FDA/EMA requirement pairs, labels sourced from three open-access expert-panel reviews:

| Source | Pairs | Topics |
|--------|-------|--------|
| *J. Crohn's & Colitis* 2025 UC endpoint comparison | 40 | Endpoints, remission, trial design |
| PMC8157504 — EMA/FDA psychiatric guidelines | 10 | Trial duration, comparators, placebo |
| PMC6529498 — US/EU radiopharmaceutical requirements | 9 | GLP, toxicology, labeling |
| Primary guidance text (ICH E9R1, E11A, FDA/EMA) | 42 | Estimands, pediatrics, biosimilars, gene therapy, etc. |

Distribution: 38 AGREE · 40 DIVERGE · 23 SILENT

---

## Reproducing results

All experiments are in the [experiments repo](https://github.com/chuchugo/cross-jurisdiction-divergence-experiments):

```bash
git clone https://github.com/chuchugo/cross-jurisdiction-divergence-experiments
cd cross-jurisdiction-divergence-experiments
pip install anthropic transformers

# Lexical baseline (no API needed)
python3 cv_eval.py

# LLM judge (requires ANTHROPIC_API_KEY in .env)
python3 llm_baseline.py

# Graph-RAG classifier
python3 graph_classify.py

# Bootstrap CI for all methods
python3 bootstrap_ci.py

# Inter-annotator agreement study
python3 iaa.py
```

---

## Citation

If you use this benchmark or code, please cite:

```
@misc{chuchugo2026xjurisdiction,
  title  = {Detecting Regulatory Divergence Across Jurisdictions:
             A Cross-Corpus Contradiction Task and Baseline Hierarchy},
  author = {chuchugo},
  year   = {2026},
  url    = {https://github.com/chuchugo/cross-jurisdiction-regulatory-divergence}
}
```

---

## License

Data sourced from publicly available FDA/EMA guidance documents (public domain) and
open-access peer-reviewed comparisons. Code released under MIT.
