# Cross-Jurisdiction Contradiction Detection with Graph-RAG
### Deep Research Brief

**Date:** 2026-06-07  
**Verification:** PASS WITH NOTES (2 source PDFs were binary-unreadable; recovered via abstract search)

---

## Executive Summary

Cross-jurisdiction contradiction detection — automatically finding where FDA and EMA regulatory guidance diverges on the same topic — is a **genuinely open NLP task**. The building blocks all exist and are mature: document-level NLI (ContractNLI, DocNLI), legal contradiction detection (LegalWiz, CLAUSE), Graph-RAG (Microsoft GraphRAG, GraphCompliance, RAGulating Compliance), and contradiction validation in RAG pipelines (arXiv 2504.00180). But **no prior work combines them for cross-jurisdictional regulatory divergence**. Critically, the divergence phenomenon is real and documented: FDA recommends the *treatment policy* estimand strategy while EMA recommends the *hypothetical* strategy for the same trials; pediatric extrapolation guidance differs; expert panel reviews exist that can serve as gold-standard annotations. Source data is favorable — FDA guidance is public domain, EMA guidelines and ICH documents are publicly downloadable. The research gap is clear, the data is accessible, and the task formulation is novel. This is a viable EMNLP-strength contribution with a compelling GACLM healthcare-impact story.

---

## 1. Prior Art: Contradiction Detection in NLP

### 1.1 Document-Level NLI Foundations
- **ContractNLI** (Koreeda & Manning, EMNLP Findings 2021) — first dataset for document-level NLI on contracts; classifies hypotheses as entailment / contradiction / not-mentioned and extracts evidence spans. Established that **contradiction detection lags substantially behind entailment**, especially with "negation by exception" and cross-sectional logic [1].
- **DocNLI** — extended sentence-level NLI to full documents; standard method for detecting inconsistencies across document pairs [2].

### 1.2 Legal Contradiction Detection (2025–2026)
- **LegalWiz** (arXiv 2510.03418) — multi-agent generation framework for contradiction detection in legal documents. Decomposes detection across specialized agents, each analyzing through a different lens. Reports F1 improvements over single-model baselines. Stated limitation: confined to specific document types, generalization to diverse regimes is open [3].
- **Better Call CLAUSE** (arXiv 2511.00340, EACL 2026 Findings) — discrepancy benchmark with **7,500+ perturbed contracts** from CUAD and ContractNLI. Persona-driven pipeline generates 10 anomaly categories validated against statutes via RAG. Categories include **Legal Contradictions** (conflicts with statutory requirements) and **InText Contradictions**. Key finding: LLMs miss subtle errors and **struggle to justify them legally** [4].
- **Contradiction Detection in RAG Systems** (arXiv 2504.00180) — evaluates LLMs as context validators to catch contradictions in retrieved evidence before generation [5].

### 1.3 The Critical Limitation
All existing contradiction work is **intra-document** (within one contract) or **contract-vs-statute** (single jurisdiction). LLMs "struggle to reason over conflicts arising from evolving regulations [or] overlapping jurisdictions," and RAG pipelines "leave unresolved conflicts in evidence unchecked" [3]. No work treats **two parallel regulatory corpora from different jurisdictions** as the contradiction-detection target.

---

## 2. Graph-RAG: The Retrieval Backbone

### 2.1 Microsoft GraphRAG (2024)
Introduced 2024, 20K+ GitHub stars. Pipeline: GPT-4 entity extraction → relationship mapping (co-occurrence + semantic similarity) → community detection (Leiden algorithm) → community summaries. Two query modes: **Local Query** (precise facts) and **Global Query** (themes across corpus). **LazyGraphRAG** cuts indexing cost to 0.1% of full GraphRAG [6].

### 2.2 Regulatory Graph-RAG (2025)
- **RAGulating Compliance** (arXiv 2508.09893) — multi-agent KG of regulatory subject-predicate-object triplets + RAG; ontology-free; subgraph visualization for traceability [7].
- **GraphCompliance** (arXiv 2510.26309) — aligns **policy graphs** (regulatory requirements) with **context graphs** (organizational facts) for GDPR compliance. Stated open problems: **handling conflicting regulatory interpretations**, scaling graph construction, keeping alignments accurate as regulations evolve [8].

GraphCompliance's policy-graph construction is directly reusable: build one policy graph per jurisdiction, then detect divergence at aligned nodes.

---

## 3. The Divergence Phenomenon Is Real (Gold-Standard Source)

Documented FDA/EMA divergences that can anchor a gold-standard set:

| Topic | FDA Position | EMA Position | Source |
|---|---|---|---|
| Primary estimand strategy | Treatment policy strategy | Hypothetical strategy | [9] |
| Pediatric extrapolation | ICH E11A interpretation differs | ICH E11A interpretation differs | [10] |
| Ulcerative colitis drug dev | Distinct endpoint guidance | Distinct endpoint guidance | [9] |
| Pediatric approval lag | Median 7.5 yr post-adult | Median 6.8 yr post-adult | [9] |

Expert panel reviews (e.g., the ulcerative colitis comparison in *J. Crohn's & Colitis*) provide **human-annotated divergence analyses** — these are the gold standard for evaluation [9].

---

## 4. Data Availability

| Source | Status | Notes |
|---|---|---|
| FDA Guidance Documents | Public domain | Downloadable; one study pulled 1,351 docs [11] |
| EMA Scientific Guidelines | Public | 176 docs in same study; PDF downloads [11] |
| ICH Guidelines | Public | Available via EMA/FDA portals (M3, E9(R1), E11A, etc.) [11] |
| Expert comparison reviews | Published literature | Gold-standard divergence annotations [9] |

**Verdict:** Fully buildable without proprietary data.

---

## 5. The Research Gap (Novelty Statement)

> **No existing work performs cross-jurisdictional regulatory contradiction/divergence detection as an NLP task.**

- Contradiction detection exists, but intra-document or single-jurisdiction (ContractNLI, LegalWiz, CLAUSE)
- Graph-RAG exists, but for single-corpus QA or single-jurisdiction compliance (GraphRAG, GraphCompliance, RAGulating)
- FDA/EMA divergence is documented manually by domain experts, but **never automated**

The novel contribution = **dual policy-graph construction + cross-jurisdiction alignment + divergence classification (AGREE / DIVERGE / SILENT)**, evaluated against expert-annotated regulatory comparisons.

---

## 6. Proposed Method (Refined for Experimentation)

1. **Corpus pairing**: For each topic, gather the FDA guidance section + EMA guideline section + ICH parent (if any)
2. **Policy graph construction** (per jurisdiction): extract requirement triplets (GraphRAG/GraphCompliance style)
3. **Cross-jurisdiction alignment**: entity normalization + embedding similarity to match corresponding requirement nodes
4. **Divergence classification** per aligned node-pair:
   - **AGREE** — both jurisdictions require the same thing
   - **DIVERGE** — directly contradictory requirements (NLI contradiction)
   - **SILENT** — one jurisdiction does not address it
5. **Explanation generation**: natural-language summary of the divergence for a regulatory affairs specialist

**Baselines to beat:** (a) flat RAG + LLM judge (no graph), (b) pairwise NLI on retrieved chunks, (c) GPT-4o direct comparison without retrieval.

**Metrics:** divergence classification F1 (3-class), alignment accuracy, explanation faithfulness, vs. expert-annotated gold set.

---

## 7. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Expert gold annotations are scarce | Use published comparison reviews + GPT-4o judge with 20% human spot-check |
| Triplet extraction noisy on dense regulatory text | Start with structured ICH guidelines (cleaner) before FDA prose |
| 3-class boundary fuzzy (DIVERGE vs SILENT) | Clear annotation guidelines; report inter-annotator agreement |
| Small initial dataset | Scope to 2–3 therapeutic areas / topics for prototype; expand for camera-ready |

---

## 8. Open Questions (For the Experiment to Answer)

1. Does the graph structure actually help vs. flat RAG, or is GPT-4o direct comparison sufficient?
2. What's the realistic ceiling on divergence F1 with public data + LLM judge?
3. Can the SILENT class be reliably distinguished from retrieval failure?
4. How few gold annotations are needed for a credible evaluation?

---

## Sources

[1] [ContractNLI (Koreeda & Manning, EMNLP Findings 2021)](https://aclanthology.org/2021.findings-emnlp.164/)  
[2] [DocNLI: Document-level Natural Language Inference](https://www.researchgate.net/publication/353490366_DocNLI_A_Large-scale_Dataset_for_Document-level_Natural_Language_Inference)  
[3] [LegalWiz: Multi-Agent Contradiction Detection in Legal Documents (arXiv 2510.03418)](https://arxiv.org/pdf/2510.03418)  
[4] [Better Call CLAUSE: Discrepancy Benchmark for Auditing LLMs (arXiv 2511.00340, EACL 2026)](https://arxiv.org/abs/2511.00340)  
[5] [Contradiction Detection in RAG Systems (arXiv 2504.00180)](https://arxiv.org/pdf/2504.00180)  
[6] [Project GraphRAG — Microsoft Research](https://www.microsoft.com/en-us/research/project/graphrag/)  
[7] [RAGulating Compliance: Multi-Agent Knowledge Graph for Regulatory QA (arXiv 2508.09893)](https://arxiv.org/abs/2508.09893)  
[8] [GraphCompliance: Aligning Policy and Context Graphs (arXiv 2510.26309)](https://arxiv.org/pdf/2510.26309)  
[9] [Comparison of FDA and EMA guidance in ulcerative colitis — J. Crohn's & Colitis](https://academic.oup.com/ecco-jcc/article/19/7/jjaf111/8210108)  
[10] [ICH E11A Pediatric Extrapolation — FDA](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/e11a-pediatric-extrapolation)  
[11] [EMA and FDA psychiatric drug trial guidelines assessment — PMC8157504](https://pmc.ncbi.nlm.nih.gov/articles/PMC8157504/)  
