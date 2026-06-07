# Provenance: Cross-Jurisdiction Contradiction Detection (Graph-RAG)

- **Date:** 2026-06-07
- **Rounds:** 1 (direct search, 6 queries + 3 fetches)
- **Sources consulted:** 11 accepted + 2 unreadable PDFs (recovered via abstract search)
- **Sources accepted:** 11
- **Sources rejected/blocked:** arXiv 2504.00180 and aclanthology 2026.findings-eacl.305 PDFs were binary-unreadable via WebFetch; content recovered through abstract/search snippets (CLAUSE confirmed via arXiv 2511.00340 abstract page)
- **Verification:** PASS WITH NOTES
  - Research gap (no cross-jurisdiction contradiction detection) verified by negative search across contradiction + Graph-RAG literature
  - FDA/EMA divergence confirmed concretely (estimand strategies, pediatric extrapolation) via expert review [9]
  - Data availability confirmed: FDA public domain, EMA/ICH downloadable [11]
  - GraphRAG, GraphCompliance, RAGulating Compliance confirmed at cited IDs
- **Plan:** outputs/.plans/xjurisdiction-contradiction-graphrag.md
- **Final:** outputs/xjurisdiction-contradiction-graphrag.md
- **Next step:** Step 2 — prototype experiment + autoresearch loop
