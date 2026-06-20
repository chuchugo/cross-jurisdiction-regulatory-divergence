# Response to Reviewers — R1

**RegDivergence-101** | AIFM 2026 | `xjurisdiction-divergence-r2.tex`

**R1 — Topic Alignment.** Added §3.2. Track A (59): co-topicality from peer-reviewed comparison studies. Track B (42): CTD anchoring + expert confirmation. Exclusions in `excluded_pairs.jsonl`.

**R2 — SILENT Directionality.** SILENT → SILENT\_FDA (n=13) / SILENT\_EMA (n=8); per-direction F1 in Table 2. All pairs `doc_confidence = MODERATE`; audit deferred to RegDivergence-500.

**R3 — Scale & Bias.** Evaluation-only; CI ≈ ±0.047, rankings separable. Haiku 0.804 (JCC-2025, n=40) vs. 0.844 (others). RegDivergence-500 roadmap in §7.

**R4 — Annotation.** Human-human IAA: Zhou → **κ = 0.85** (101 pairs). Prior κ = 0.542 = LLM prompt-robustness (36 pairs), labelled separately. 10 disagreements: 6 by source; 4 flagged `disputed = True`.

**Proactive.** ICH E9(R1) → tripartite doc; "findings" → "observations"; Graph-RAG prompts in Appendix B; within-model scope note added.
