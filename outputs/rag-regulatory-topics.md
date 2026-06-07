# Research Topic Suggestions: RAG × Life Sciences Regulatory Documents
### Targeting GACLM 2026 + EMNLP 2026 Strong Accept

**Date:** 2026-06-07  
**Building on:** [outputs/rag-lifescience-regulatory.md](rag-lifescience-regulatory.md)

---

## What Each Venue Rewards

| | GACLM 2026 (Jun 15) | EMNLP 2026 (next ARR) |
|---|---|---|
| **Track** | GenAI in Healthcare, Education, Robotics, Smart Cities | Clinical and Biomedical Applications; Resources & Evaluation |
| **Wants** | Applied GenAI impact, deployed systems, real-world healthcare use cases | Novel benchmark OR new method with rigorous ablations + reproducible results |
| **Special angle** | "What does GenAI enable that wasn't possible before?" | "New Missions for NLP" — rethink evaluation, build resources, advance scientific understanding |
| **Paper length** | Short applied paper OK | Full 8-page paper preferred for strong accept |

**The sweet spot for both:** *New benchmark + RAG method + healthcare/regulatory application.* GACLM loves the impact story; EMNLP rewards the rigorous resource + evaluation structure.

---

## Confirmed Open Problems (Literature-Derived)

From surveying the field:

| Gap | Evidence |
|---|---|
| Deep hierarchical nesting in CTD/SmPC | Level 4 nested structures: ~15% accuracy [PMC12380897] |
| No end-to-end regulatory compliance benchmark | FDARxBench = generic drugs only; PED-X-Bench = pediatric labels only; no multi-domain/multi-jurisdiction benchmark exists |
| Multi-jurisdiction contradiction detection | FDA vs. EMA divergence on same topics: entirely unstudied as NLP task |
| CMC Module 3 NLP | Highest FDA deficiency rate section; zero dedicated RAG/NLP literature |
| Agentic multi-hop regulatory reasoning | 30% silent failure rate in enterprise multi-hop deployments; no pharma evaluation framework |
| Pharmacovigilance RAG | Listed as future work in every PV AI paper; not addressed with RAG |
| KG staleness under regulation updates | FDA revises at 15%/year; no incremental KG updating work for regulatory corpora |

---

## Topic Candidates

---

### 🥇 Topic 1 — TOP PICK (Both Venues)
## Cross-Jurisdiction Regulatory Contradiction Detection with Graph-RAG

**One-line:** Build a system that automatically detects where FDA and EMA guidance diverges on the same drug/clinical topic, using policy graphs aligned across jurisdictions.

**The problem:** Pharma companies submitting to both FDA and EMA must manually reconcile guidance documents that frequently contradict each other (e.g., pediatric extrapolation thresholds, statistical estimand requirements, labeling sections). This costs millions in regulatory affairs time and causes submission delays.

**The method:**
1. For a given topic (e.g., "sample size re-estimation in adaptive trials"), retrieve relevant sections from FDA guidance + EMA guidance
2. Construct a *policy graph* per jurisdiction: subject-predicate-object triplets of regulatory requirements (extending GraphCompliance arXiv 2510.26309 + RAGulating Compliance arXiv 2508.09893)
3. Align graphs cross-jurisdiction using entity normalization + semantic similarity
4. Contradiction detection: NLI classifier + LLM judge on aligned node pairs flagging AGREE / DIVERGE / SILENT (one jurisdiction doesn't address it)
5. Summarize divergence in plain language for regulatory affairs specialist

**Why EMNLP strong accept:**
- Contradiction detection is a core NLP task with established precedent; regulatory is a new domain
- Graph-RAG + cross-domain alignment = methodologically novel combination
- "Clinical and Biomedical Applications" + "Resources and Evaluation" tracks
- New task formulation = opens a research line (community will build on it)
- Ablation-friendly: graph vs. no-graph, NLI vs. LLM judge, monolingual vs. cross-lingual

**Why GACLM strong accept:**
- "AI that automatically finds where FDA and EMA disagree" = prototypical GenAI-in-healthcare impact
- Goes beyond retrieval → demonstrates GenAI reasoning + alignment
- Directly applicable to any pharma company with dual US/EU submissions

**Feasibility (8 days for GACLM):**
- 50 curated topic-pairs with known FDA/EMA divergence (sources: ICH harmonization gap analyses, academic comparative regulatory studies)
- GPT-4o policy graph construction + LLM judge for contradiction scoring
- Expert spot-check of 20% sample as gold standard
- Achievable as a system/demo paper for GACLM; expand with full annotation + ablations for EMNLP

---

### 🥈 Topic 2 — CO-RECOMMENDED (EMNLP-Primary)
## RegulatoryBench: A Multi-Domain Benchmark for Pharmaceutical Compliance QA

**One-line:** The first public, multi-jurisdiction QA benchmark for pharmaceutical regulatory compliance, spanning CTD Modules 2–5 across FDA, EMA, and ICH guidelines.

**The problem:** No comprehensive regulatory compliance QA benchmark exists. Existing datasets are narrow: FDARxBench (generic bioequivalence only), PED-X-Bench (pediatric label extrapolation), AIReg-Bench (AI regulation governance). A multi-domain, multi-jurisdiction benchmark would enable systematic evaluation and comparison of all RAG approaches.

**Benchmark design:**
- 4 document domains: Module 2 (clinical overview), Module 3 (CMC), Module 4 (nonclinical), Module 5 (clinical studies)
- 3 jurisdiction corpora: FDA guidance, EMA guidelines, ICH harmonized guidelines
- 500–800 QA pairs; answer types: span extraction, yes/no compliance, open-ended reasoning
- All sourced from publicly available guidance documents (no proprietary NDA content)

**Paired contributions:**
- Baseline RAG system (naive, HiSACC, KG-RAG) evaluated on the benchmark
- Error analysis: which CTD modules and question types fail most

**Why EMNLP strong accept:**
- New resource paper — one of the most reliably accepted paper types at EMNLP
- Fills an obvious community need; will drive citations
- Special theme: "new evaluation methodologies for NLP" is literally this
- "Clinical and Biomedical Applications" track is ideal

**Why GACLM fits:**
- GenAI evaluation framework for healthcare = directly fits the track
- Applied impact story: benchmark enables pharma companies to evaluate their own RAG tools

**Feasibility (8 days for GACLM):** Scope to Module 5 clinical only, 150–200 QA pairs. Still a meaningful contribution. Full 4-module version for EMNLP.

---

### 🥉 Topic 3 — STRONG (GACLM-Primary, EMNLP-Compatible)
## CMC Module 3 Compliance QA: First RAG System for Chemistry, Manufacturing, and Controls

**One-line:** CMC (CTD Module 3) has the highest FDA deficiency rate and zero NLP literature. Build and evaluate the first RAG system for CMC regulatory compliance questions.

**The problem:** FDA sends Chemistry, Manufacturing, and Controls deficiency letters for 60%+ of NDA/BLA submissions. These deficiencies (stability testing gaps, analytical method validation issues, manufacturing process descriptions) delay drug approvals by 6–12 months. Every gap maps to a specific ICH guideline (Q2(R2), Q3, Q8, Q9, Q10, Q12) or FDA CMC guidance.

**The method:**
- Corpus: ICH Q-series guidelines + FDA CMC guidance documents (all public)
- 100–150 expert-annotated QA pairs covering stability, formulation, analytical validation, manufacturing process
- Advanced RAG with HiSACC chunking (CMC docs have complex table-heavy structure)
- Evaluation: retrieval recall, answer faithfulness, citation accuracy

**Why GACLM strong accept:**
- Zero competition in this space — completely novel application of GenAI to healthcare
- Massive industry impact story: CMC delays cost $1M+/day in lost sales for approved products
- Clear "GenAI enabling healthcare" narrative

**Why EMNLP compatible:**
- New domain + resource = novel contribution
- Weaker than Topics 1–2 for EMNLP pure NLP novelty unless paired with a new method (e.g., table-aware chunking for regulatory documents)

---

### Topic 4 — GOOD (Methodological Focus)
## Agentic RAG for Multi-Hop CTD Compliance Review

**One-line:** CTD compliance evaluation is inherently multi-hop. Design and benchmark an agentic RAG pipeline for 2–4 step regulatory reasoning chains.

**The gap:** Current RAG is single-hop. ICH E9(R1) compliance requires: retrieve estimand definition → check protocol estimand statement → verify intercurrent event handling → cross-reference SAP. 30% of multi-hop enterprise queries fail silently; no pharma-specific evaluation exists.

**Best for:** EMNLP (methodological + systems paper). Less urgent for GACLM Jun 15 given time required.

---

### Topic 5 — INTERESTING (Long-Shot for Both)
## Pharmacovigilance RAG: Grounding Safety Signal Detection in PSURs and FAERS

**One-line:** Build a hybrid RAG pipeline over public FAERS structured data + EMA EPAR narrative safety reports to answer post-market safety surveillance questions.

**The gap:** Every pharmacovigilance AI paper lists RAG as future work. FAERS is publicly available. EMA EPARs are public. This is buildable without proprietary data.

**Caveat:** PSURs themselves are proprietary; must use EMA EPARs as substitute. Weaker signal than Topics 1–3 for EMNLP unless paired with a novel safety detection method.

---

## Decision Matrix

| # | Topic | EMNLP | GACLM | Novelty | 8-day feasibility | Pick |
|---|-------|-------|-------|---------|-------------------|------|
| 1 | Contradiction Detection (Graph-RAG) | ★★★★★ | ★★★★★ | ★★★★★ | ★★★★ | **Both venues** |
| 2 | RegulatoryBench | ★★★★★ | ★★★★ | ★★★★★ | ★★★ (data work) | **EMNLP primary** |
| 3 | CMC Module 3 QA | ★★★ | ★★★★★ | ★★★★★ | ★★★★ | **GACLM primary** |
| 4 | Agentic RAG CTD | ★★★★ | ★★★★ | ★★★★ | ★★★ | EMNLP secondary |
| 5 | Pharmacovigilance RAG | ★★★ | ★★★★ | ★★★★ | ★★★ | If PSUR access |

---

## Recommended Path

**Single paper for both venues → Topic 1: Cross-Jurisdiction Contradiction Detection**

This is the clearest dual-fit. The GACLM version (Jun 15): 50-pair system paper demonstrating the pipeline with GPT-4o judge. The EMNLP version (next ARR): full annotation, ablations, graph construction variants, multi-language extension. Same core contribution, two framings.

**If you want a cleaner EMNLP bet → Topic 2: RegulatoryBench**  
New benchmarks are reliably accepted at EMNLP when the community gap is real and the annotation methodology is rigorous. The gap here is undeniable.

**If you need something deployable fast for GACLM → Topic 3: CMC Module 3**  
Completely unaddressed domain, high industry stakes, 8-day buildable scope.

---

## Sources

- [RegGuard arXiv 2601.17826](https://arxiv.org/html/2601.17826v1) — architecture, benchmarks
- [Waikar et al. PMC12917324](https://pmc.ncbi.nlm.nih.gov/articles/PMC12917324/) — RAG for clinical protocol compliance
- [SmPC/IDMP PMC12380897](https://pmc.ncbi.nlm.nih.gov/articles/PMC12380897/) — hierarchical extraction limits
- [RAGulating Compliance arXiv 2508.09893](https://arxiv.org/abs/2508.09893) — KG + RAG for regulatory QA
- [GraphCompliance arXiv 2510.26309](https://arxiv.org/pdf/2510.26309) — policy graph alignment (GDPR)
- [FDARxBench arXiv 2603.19539](https://arxiv.org/pdf/2603.19539) — FDA generic drug benchmark
- [EMNLP 2026 CFP](https://2026.emnlp.org/calls/main_conference_papers/) — topics, special theme, ARR deadlines
- [NLLP 2026](https://www.aclweb.org/portal/content/nllp-2026) — regulatory NLP workshop (Aug 11)
- [IntuitionLabs RAG pharma](https://intuitionlabs.ai/articles/rag-performance-pharmaceutical-documents) — benchmarks, adoption data
- [21 CFR Part 11 + AI](https://intuitionlabs.ai/articles/21-cfr-part-11-compliance-ai-systems) — compliance framework
