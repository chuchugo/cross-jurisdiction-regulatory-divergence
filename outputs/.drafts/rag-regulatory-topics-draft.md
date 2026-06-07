# Research Topic Suggestions: RAG × Life Sciences Regulatory Documents
# Targeting GACLM 2026 + EMNLP 2026

**Date:** 2026-06-07
**Building on:** outputs/rag-lifescience-regulatory.md

---

## Context: What Each Venue Wants

**GACLM 2026 — "GenAI in Healthcare, Education, Robotics, and Smart Cities" track**
- Deadline: June 15, 2026 (8 days)
- Wants: Applied GenAI with clear real-world healthcare impact, system demonstrations, novel use cases, deployed results. Broad applied AI venue.

**EMNLP 2026 — Strong Accept Target**
- ARR May deadline: May 25, 2026 (passed); next ARR cycle needed OR direct submission
- Wants: Substantial, original, empirically rigorous NLP research. Explicitly lists "Clinical and Biomedical Applications" as a topic area. Special theme: "New Missions for NLP Research" — rethinking evaluation, building new resources, advancing scientific understanding. Strong paper = novel benchmark OR novel method with clean ablations + reproducible results.

**The Sweet Spot:** A paper needs a new benchmark or method + clear NLP contribution + healthcare/regulatory application framing. "We built a regulatory compliance benchmark and a RAG method that beats baselines" hits both venues.

---

## Open Problems in the Field (Literature-Derived)

From the evidence gathered:

1. **Hierarchical document structure** — Level 4 nested CTD/SmPC structures: ~15% accuracy. No dedicated solution exists.
2. **Knowledge base staleness** — FDA regulations revised at 15%/year; no paper addresses incremental KG updating for regulatory corpora.
3. **Multi-jurisdiction contradiction detection** — FDA vs. EMA guidance contradictions on the same drug/indication: entirely unstudied as an NLP task.
4. **Benchmark gap** — FDARxBench (generic drugs), PED-X-Bench (pediatric extrapolation), AIReg-Bench (AI regulation), DART (Italian) exist — but NO public end-to-end benchmark for regulatory compliance QA across major CTD modules.
5. **Agentic RAG for multi-hop regulatory reasoning** — 30% of multi-hop queries fail silently in enterprise deployments; no pharma-specific evaluation framework.
6. **CMC document automation** — Module 3 (Chemistry, Manufacturing, Controls) is the most document-heavy CTD section with the highest FDA deficiency rate; almost no RAG literature addresses it.
7. **Pharmacovigilance RAG** — Post-market surveillance (PSUR/PBRER reports, FAERS data integration) mentioned as a future direction in every PV AI paper but not addressed with RAG.
8. **Graph-RAG for regulatory triplets** — RAGulating Compliance (arXiv 2508.09893) and GraphCompliance (arXiv 2510.26309) exist but are preliminary, GDPR-focused, or non-pharma.

---

## Topic Candidates

---

### Topic 1 ★★★★★ — RECOMMENDED TOP PICK
## "RegulatoryBench: A Multi-Domain Benchmark for RAG-Based Pharmaceutical Compliance QA"

**Core idea:** Build the first comprehensive public benchmark for pharmaceutical regulatory compliance QA, covering 4 CTD modules (Module 2 overviews, Module 3 CMC, Module 4 nonclinical, Module 5 clinical) × 3 jurisdictions (FDA, EMA, ICH). Pair it with a baseline RAG system and ablations.

**Why it wins at EMNLP:**
- New resource + evaluation = core EMNLP contribution type
- Clinical and Biomedical Applications track is explicitly listed
- Special theme: "new evaluation methodologies" — a benchmark for regulatory NLP is exactly this
- FDARxBench (generic drugs only) and PED-X-Bench (pediatric labels only) leave a massive gap; a multi-domain, multi-jurisdiction benchmark is a clear advance
- Ablation-friendly: compare naive RAG / HiSACC / Graph-RAG / agentic RAG on the same benchmark

**Why it wins at GACLM Healthcare track:**
- Direct GenAI impact on healthcare regulatory process
- Framed as: "enabling GenAI to assist FDA/EMA submission teams" — prototypical healthcare GenAI application
- Deployable system angle

**Execution scope:**
- 500–1,000 QA pairs across 4 CTD module types
- Sourced from public FDA guidance documents, published drug applications (hatch-waxman databases, EMA EPAR), and ICH guidelines
- Metrics: faithfulness, retrieval recall@K, answer F1, citation accuracy
- 3-way comparison: baseline RAG, RegGuard-style advanced RAG, KG-RAG

**Risk:** Dataset construction is ~60% of the work. If scope is too ambitious for 8 days (GACLM), aim for Module 5 clinical only with 200–300 QA pairs as a focused contribution.

---

### Topic 2 ★★★★★ — CO-RECOMMENDED
## "Cross-Jurisdiction Regulatory Contradiction Detection with Graph-RAG"

**Core idea:** FDA and EMA frequently have divergent guidance on the same topic (e.g., pediatric extrapolation, statistical design, labeling requirements). Build a system that: (1) retrieves relevant sections from both corpora, (2) constructs a policy graph per jurisdiction, (3) detects semantic contradictions between graphs, and (4) summarizes the discrepancy for a regulatory affairs specialist.

**Why it wins at EMNLP:**
- Contradiction detection is a well-established NLP task with EMNLP precedent
- Cross-lingual / cross-domain semantic alignment is technically novel in the regulatory context
- Graph-RAG + contradiction detection = methodologically interesting combination
- No paper has addressed FDA vs. EMA regulatory contradiction detection — clear novelty

**Why it wins at GACLM Healthcare track:**
- Highly impactful for pharma: harmonization failures cost companies millions in duplicate submissions
- "AI that automatically flags where FDA and EMA disagree" = clear GenAI-in-healthcare narrative
- Demonstrates GenAI reasoning beyond retrieval — alignment + contradiction = reasoning

**Execution scope:**
- Curate 50–100 topic pairs where FDA and EMA guidance diverges (from existing comparative literature, ICH diffs)
- Build GraphCompliance-style policy graphs for each jurisdiction
- Contradiction classifier: NLI-based or LLM-judged with graph structural features
- Evaluation: precision/recall of detected contradictions vs. expert-annotated gold set

**Risk:** Expert annotation is expensive; could use GPT-4o as judge + human review of 20% sample as a reasonable approximation.

---

### Topic 3 ★★★★ 
## "Agentic RAG for CTD Module Compliance: Evaluation of Multi-Hop Regulatory Reasoning"

**Core idea:** Current RAG is single-hop (one query → retrieve → answer). CTD compliance review is inherently multi-hop: "Does this statistical section comply with ICH E9(R1)?" requires (1) retrieve E9(R1) section on estimands, (2) check if the protocol defines a primary estimand, (3) check the intercurrent event handling strategy, (4) cross-reference the SAP. Design and evaluate an agentic RAG pipeline for this task.

**Why it wins at EMNLP:**
- Novel task formulation: multi-hop regulatory reasoning benchmark
- Connects to active EMNLP area: agentic/tool-use NLP systems
- Clinical and Biomedical Applications + Resources and Evaluation tracks
- 30% silent failure rate in enterprise multi-hop deployments is a known problem with no pharmaceutical-domain solution

**Why it wins at GACLM:**
- Agentic GenAI system for healthcare compliance automation
- "AI agent that reviews clinical protocols before FDA submission" is compelling applied GenAI narrative

**Execution scope:**
- Select 3–5 ICH guidelines (E6, E8, E9, E9(R1), E11)
- Build 100–200 multi-hop QA chains, each requiring 2–4 retrieval steps
- Baseline: single-hop RAG; proposed: ReAct + verification agent
- Metric: step-level retrieval accuracy + final answer accuracy + faithfulness

---

### Topic 4 ★★★★
## "CMC Module Compliance QA: RAG for Chemistry, Manufacturing, and Controls Regulatory Review"

**Core idea:** CMC (CTD Module 3) is the most document-dense section of an NDA/BLA and has the highest FDA deficiency rate. Yet it has zero dedicated NLP/RAG literature. Build a RAG system and evaluation framework for CMC compliance questions (stability testing, manufacturing process description, analytical method validation per ICH Q2(R2)).

**Why it wins at GACLM:**
- Unaddressed healthcare/pharma GenAI application with enormous industry demand
- Clear cost savings: CMC deficiency letters from FDA delay drug approvals by 6–12 months
- Applied GenAI system with real-world deployment potential

**Why it wins at EMNLP:**
- Domain-specific benchmark + system = resource + methods paper
- Less competitive niche than clinical NLP (fewer submissions) but clearly within "Clinical and Biomedical Applications"
- Novel: no prior NLP work on CMC-specific regulatory documents

**Execution scope:**
- Focus on ICH Q8/Q9/Q10/Q2(R2) guidelines + FDA CMC guidance documents
- 100–150 expert-annotated QA pairs covering: stability, formulation, manufacturing process, analytical validation
- RAG baseline + HiSACC chunking ablation (since CMC docs have complex table/section structure)

---

### Topic 5 ★★★
## "SmPC Hierarchical Extraction at Scale: Benchmarking LLM+RAG for IDMP Compliance"

**Core idea:** Extend the SmPC/IDMP work (PMC12380897) which found Level 4 nested structures hit ~15% accuracy. Systematically benchmark extraction across all nesting levels with multiple chunking strategies (fixed, HiSACC, rule-based, hierarchical) and embedding models (e5-small-v2, ClinicalBERT, text-embedding-ada-002).

**Why it's good but not top pick:**
- More incremental than Topics 1–2 (extending existing work rather than opening new ground)
- Niche audience (IDMP is EU-specific regulatory data standard)
- Lower EMNLP novelty unless paired with a new method for hierarchical extraction

---

### Topic 6 ★★★
## "Pharmacovigilance RAG: Grounding Adverse Event Detection in PSURs and FAERS"

**Core idea:** Periodic Safety Update Reports (PSURs) and FDA Adverse Event Reporting System (FAERS) data are the two primary post-market safety document types. Build a RAG pipeline that answers "Has a safety signal for [drug] + [event] been identified in the post-market period?" by retrieving from both structured (FAERS) and unstructured (PSUR narrative) sources.

**Why it's interesting:**
- Pharmacovigilance is identified as a high-priority future direction in every PV AI paper
- FAERS is publicly available — real data pipeline possible
- FDA's "Elsa" tool (launched June 2025) signals regulatory AI in this space is live

**Why it's not top pick:**
- Requires access to PSURs (often proprietary) unless using EMA public EPARs
- Straddles NLP and healthcare informatics — may not be strong enough for EMNLP without a rigorous NLP contribution

---

## Decision Matrix

| Topic | EMNLP Fit | GACLM Fit | Novelty | Feasibility (fast) | Recommended |
|-------|-----------|-----------|---------|-------------------|-------------|
| 1. RegulatoryBench | ★★★★★ | ★★★★ | ★★★★★ | ★★★ (dataset work) | **YES — Top Pick** |
| 2. Contradiction Detection | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | **YES — Co-Pick** |
| 3. Agentic RAG CTD | ★★★★ | ★★★★ | ★★★★ | ★★★★ | Yes |
| 4. CMC Module QA | ★★★ | ★★★★★ | ★★★★★ | ★★★★ | Yes — GACLM priority |
| 5. SmPC IDMP Bench | ★★★ | ★★★ | ★★★ | ★★★★ | Maybe |
| 6. Pharmacovigilance RAG | ★★★ | ★★★★ | ★★★★ | ★★★ | If PSUR access available |

---

## Recommended Path Forward

**For GACLM 2026 (June 15 deadline — 8 days):**
Go with **Topic 4 (CMC Module QA)** or a scoped version of **Topic 1 (RegulatoryBench — Module 5 clinical only)**. CMC is the most underexplored and most practically impactful. A focused system paper with 100–150 QA pairs and clear results is achievable in 8 days if you already have the regulatory documents.

**For EMNLP 2026 (next ARR cycle — check deadlines):**
Go with **Topic 1 (RegulatoryBench)** or **Topic 2 (Contradiction Detection)**. Both open new tasks, enable follow-on work, and have the resource + methods structure that EMNLP rewards. Topic 2 is more technically elegant; Topic 1 has broader community impact.

**If doing just one paper for both venues:**
**Topic 2** (Cross-Jurisdiction Contradiction Detection) is the strongest dual-submission candidate:
- GACLM: "GenAI system that detects FDA/EMA regulatory conflicts for pharma teams" — clear applied healthcare GenAI
- EMNLP: "Graph-RAG with contradiction detection for regulatory NLP" — novel method + clinical application
- 8-day feasibility: curate 50 topic pairs, build graph alignment, LLM-based contradiction scoring, evaluate against expert annotations (or GPT-4o judge)
