# Plan: Update Product Discovery Agent Roster

## Context

The product-discovery plugin dispatches 8 specialized agents in parallel. The current roster includes Backend Architect, Support Responder, and UX Researcher which are being replaced with Executive Summary Generator, Product Manager Agent, and Legal Compliance Checker. The removed agents' sections are dropped entirely (not reassigned).

## Changes

### Section Order (new)

1. Executive Summary (Executive Summary Generator) — **NEW**
2. The Opportunity (orchestrator) — was Section 1
3. Market Validation (Trend Researcher) — was Section 2
4. Product Strategy & Requirements (Product Manager Agent) — **NEW**
5. Brand Strategy (Brand Guardian) — was Section 4
6. Go-to-Market & Growth (Growth Hacker) — was Section 5
7. Legal & Compliance (Legal Compliance Checker) — **NEW**
8. Project Execution Plan (Project Shepherd) — was Section 8
9. Domain-specific — was Section 9
10. Cross-Agent Synthesis (orchestrator) — was Section 10

### Removed (dropped entirely)

- Backend Architect → Section 3: Technical Architecture
- Support Responder → Section 6: Customer Support Blueprint
- UX Researcher → Section 7: UX Research & Design Direction

### New Agent Deliverables

**Executive Summary Generator (Section 1):**
- Concise 1-page executive summary of all findings
- Key metrics snapshot (market size, timeline, budget range)
- GO / NO-GO / CONDITIONAL verdict with top 3 reasons
- Top 3 risks and top 3 opportunities
- Critical next steps

**Product Manager Agent (Section 4):**
- Problem statement and value proposition
- User stories for core flows (prioritized MoSCoW)
- MVP scope definition (in/out table)
- Success metrics and KPIs
- Product roadmap with relative milestones (Phase 1, 2, 3 — no fixed dates)

**Legal Compliance Checker (Section 7):**
- Regulatory landscape (GDPR, CCPA, industry-specific)
- Data protection and privacy requirements
- IP considerations (patents, trademarks, open-source licensing)
- Terms of service and user agreement needs
- Compliance risks ranked by severity

## Files to Modify

### 1. `plugins/product-discovery/skills/product-discovery/SKILL.md`

- Update intro paragraph: still 8 agents, but new roster description (drop "system architecture, support, UX", add "executive summary, product strategy, legal compliance")
- **Dispatch Table** (lines 65-74): Replace 3 removed agents with 3 new agents, renumber all sections
- **Section Deliverables** (lines 78-100): Remove Backend Architect, Support Responder, UX Researcher deliverables; add Executive Summary Generator, Product Manager Agent, Legal Compliance Checker deliverables
- **Existing deliverables**: Change Growth Hacker's "KPIs and growth targets by quarter" → relative phases. Change Project Shepherd's "Timeline and dependencies" → relative milestones, no fixed dates.
- **Phase 3** (lines 102-126): Update — Section 1 is now Executive Summary (agent-written), Section 2 is The Opportunity (orchestrator-written). Update assembly instructions and section references. Update Section 10 synthesis to reference new agent names.

### 2. `plugins/product-discovery/skills/product-discovery/references/report-template.md`

- Update Table of Contents with new section names and numbering
- Replace removed sections (Technical Architecture, Customer Support, UX Research) with new sections (Executive Summary, Product Strategy, Legal & Compliance)
- Renumber all sections to match new order
- Update Section 10 synthesis examples to reference new agent names

## Verification

- Count agents in dispatch table = 8
- All section numbers 1-10 are sequential with no gaps
- No references to removed agents (Backend Architect, Support Responder, UX Researcher) remain
- No references to removed sections (Technical Architecture, Customer Support Blueprint, UX Research & Design Direction) remain
- New agent deliverables are complete and specific
