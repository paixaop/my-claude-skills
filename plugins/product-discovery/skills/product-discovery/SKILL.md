---
name: product-discovery
description: "Run a full multi-agent product discovery exercise that produces a comprehensive product blueprint. Use when the user wants to explore a product idea, validate a concept, create a product blueprint, run product discovery, or get a full-agency analysis of a product opportunity. Triggers on phrases like 'product discovery', 'explore this idea', 'validate this product', 'product blueprint', 'full discovery', 'agency discovery', 'run discovery on', or any request to analyze a product idea from multiple perspectives (market, brand, growth, legal, product strategy, project planning)."
---

# Product Discovery

Orchestrates 8 specialized agents in parallel to produce a comprehensive product blueprint. Each agent brings deep domain expertise — executive summary, market research, product strategy, brand strategy, growth, legal compliance, project management, and a domain-specific specialist chosen based on the product type.

The agents are already installed with full personalities and capabilities. This skill dispatches them with product context and assembles their outputs into a unified report.

## Phase 1: Gather Product Context

Extract the product name and description from the user's message.

**If the user provided a brief idea** (1-3 sentences), ask these clarifying questions before proceeding:
1. Who is the target user/customer?
2. What's the key differentiator or unique angle?
3. What domain does this fall into? (web app, mobile, AI/ML, spatial/XR, security, data, blockchain, or other)

**If the user provided a detailed spec** (paragraph+), extract the context directly and proceed.

Determine the **domain-specific agent** for Section 9 based on the product type:

| Product Domain | `subagent_type` | Section 9 Title |
|----------------|-----------------|-----------------|
| Spatial / XR / VR / AR | `XR Interface Architect` | Spatial Interface Architecture |
| AI / ML / LLM | `AI Engineer` | AI Architecture |
| Mobile app | `Mobile App Builder` | Mobile Stack |
| Web app / SaaS | `Frontend Developer` | Frontend Architecture |
| Security | `Security Engineer` | Security Architecture |
| Data / Analytics | `Data Engineer` | Data Architecture |
| Blockchain / Web3 | `Solidity Smart Contract Engineer` | Smart Contract Architecture |
| General / Other | `Software Architect` | System Architecture |

## Phase 2: Launch 8 Agents in Parallel

Launch all 8 agents in a **single message** using the Agent tool with `run_in_background: true` for each. The agents already have their full expertise installed — give each one the product context and their section assignment.

### Agent Roster

All 8 agents must be dispatched in one turn. For each agent, use this prompt structure:

```
Product discovery exercise for **{PRODUCT_NAME}**.

## Product
{FULL_PRODUCT_DESCRIPTION — include all context gathered in Phase 1}

## Your Assignment
Write **Section {N}: {SECTION_TITLE}** for a comprehensive product discovery report.

Focus on: {SECTION_DELIVERABLES — 2-4 bullet points from the report template}

## Output Format
- Clean markdown starting with `## {N}. {SECTION_TITLE}`
- Use tables, bullet points, and sub-sections
- Be specific, data-driven, and actionable — include concrete numbers, names, tool recommendations
- Target 80-150 lines of markdown
- Cross-reference other sections where relevant (e.g., "aligned with the growth strategy in Section 5")
```

### Dispatch Table

| Agent Call | `subagent_type` | Section | Description (for Agent tool) |
|------------|-----------------|---------|------------------------------|
| 1 | `Executive Summary Generator` | 1. Executive Summary | Executive summary generation |
| 2 | `Trend Researcher` | 3. Market Validation | Market validation research |
| 3 | `Product Manager Agent` | 4. Product Strategy & Requirements | Product strategy and requirements |
| 4 | `Brand Guardian` | 5. Brand Strategy | Brand strategy development |
| 5 | `Growth Hacker` | 6. Go-to-Market & Growth | GTM and growth strategy |
| 6 | `Legal Compliance Checker` | 7. Legal & Compliance | Legal and compliance analysis |
| 7 | `Project Shepherd` | 8. Project Execution Plan | Project execution planning |
| 8 | *domain-specific* | 9. {Domain Title} | Domain-specific analysis |

### Section Deliverables per Agent

**Executive Summary Generator (Section 1):**
- Concise 1-page executive summary, key metrics snapshot (market size, budget range), GO/NO-GO/CONDITIONAL verdict with top 3 reasons, top 3 risks and opportunities, critical next steps

**Trend Researcher (Section 3):**
- Market size (TAM/SAM/SOM with sources), competitive landscape table, market gaps, GO/NO-GO/CONDITIONAL verdict

**Product Manager Agent (Section 4):**
- Problem statement and value proposition, user stories (MoSCoW prioritized), MVP scope (in/out table), success metrics and KPIs, product roadmap with relative milestones (Phase 1, 2, 3 — no fixed dates)

**Brand Guardian (Section 5):**
- Positioning statement, name candidates, visual identity direction, brand voice, competitive differentiation

**Growth Hacker (Section 6):**
- GTM phases, pricing model, acquisition channels ranked by ROI, viral mechanics, KPIs and growth targets by phase

**Legal Compliance Checker (Section 7):**
- Regulatory landscape (GDPR, CCPA, industry-specific), data protection and privacy requirements, IP considerations (patents, trademarks, open-source licensing), terms of service needs, compliance risks ranked by severity

**Project Shepherd (Section 8):**
- Phase breakdown with relative milestones, first 2-3 sprints detailed, team composition, risk register, dependencies

**Domain Agent (Section 9):**
- Domain-specific architecture, technical considerations, integration points, best practices

## Phase 3: Synthesize and Write Report

As agents complete in the background, you'll be notified. Once all 8 are done:

1. **Read the report template** at [references/report-template.md](references/report-template.md)

2. **Assemble Section 1: Executive Summary** from the Executive Summary Generator's output.

3. **Write Section 2: The Opportunity** yourself — summarize:
   - The product concept and problem it solves
   - Why now (market timing)
   - Target users
   - What makes this opportunity compelling

4. **Assemble Sections 3-9** from the agent outputs. Light editing for consistency is fine, but preserve each agent's analysis and recommendations.

5. **Write Section 10: Cross-Agent Synthesis** yourself by analyzing across all 8 outputs:
   - **Agreements:** Where 3+ agents converge (e.g., both Growth Hacker and Trend Researcher agree on target market)
   - **Tensions:** Where agents conflict (e.g., Product Manager Agent scopes a large MVP but Project Shepherd flags complexity risk)
   - **Top Risks:** Risks surfaced by multiple agents, ranked by severity
   - **Final Recommendation:** GO / NO-GO / CONDITIONAL — with clear conditions if conditional
   - **Phased Roadmap:** Integrated roadmap combining Project Shepherd's plan with inputs from all agents (relative milestones, no fixed dates)

5. **Save the report** using the Write tool to `docs/discovery/{product-name-slug}-discovery.md` (create the `docs/discovery/` directory if it doesn't exist). Use a URL-safe slug for the product name (lowercase, hyphens).

6. **Tell the user** the report is ready and where it's saved. Highlight 2-3 key findings from the synthesis.
