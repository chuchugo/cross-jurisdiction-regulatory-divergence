# Deep Research Plan: RAG in Life Sciences for Regulatory Documents

**Slug:** `rag-lifescience-regulatory`  
**Date:** 2026-06-07  
**Status:** DRAFT — awaiting user confirmation

---

## Key Questions

1. What is the current state of RAG adoption in life sciences regulatory workflows (FDA, EMA, ICH submissions)?
2. What are the dominant architectures used (naive RAG vs. advanced RAG vs. agentic RAG) for regulatory document corpora?
3. What are the key challenges specific to regulatory documents (versioning, traceability, hallucination risk, compliance)?
4. What validation and audit requirements apply to AI/RAG systems used in regulatory contexts (21 CFR Part 11, EU AI Act, GxP)?
5. Who are the key vendors, platforms, and open-source projects targeting this space?
6. What does the evidence say about performance, accuracy, and production deployments?

---

## Evidence Needed

- Academic papers on RAG applied to pharmaceutical/regulatory text
- Industry reports and white papers (FDA, EMA, ICH guidance on AI use)
- Vendor/product landscape (Veeva, Medidata, OpenAI, Anthropic, etc.)
- Case studies or benchmarks on regulatory document QA
- Regulatory frameworks governing AI in drug development (FDA AI/ML action plan, EU AI Act)

---

## Scale Decision

**Multi-agent (3 researcher subagents)** — topic is broad with distinct threads:
- T1: RAG architectures and technical approaches for regulatory docs
- T2: Regulatory/compliance frameworks governing AI use (FDA, EMA, ICH, GxP)
- T3: Vendor landscape, case studies, and production deployments

---

## Task Ledger

| # | Task | Owner | Status |
|---|------|-------|--------|
| 1 | Write per-researcher briefs (T1, T2, T3) | Lead | PENDING |
| 2 | Spawn researcher subagents | Lead | PENDING |
| 3 | Synthesize draft | Lead | PENDING |
| 4 | Citation pass (verifier) | verifier | PENDING |
| 5 | Review pass (reviewer) | reviewer | PENDING |
| 6 | Deliver final + provenance | Lead | PENDING |

---

## Verification Log

*(empty — pre-research)*

---

## Decision Log

- 2026-06-07: Scale set to multi-agent (3 researchers) given breadth of topic across technical, regulatory, and market dimensions.
