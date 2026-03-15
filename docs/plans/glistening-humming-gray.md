# Plan: Product Discovery Skill

## Context

The user wants a new skill that orchestrates 8 specialized agents in parallel to produce a comprehensive product blueprint. This mirrors the nexus-spatial-discovery.md example from the agency-agents repo — where 8 agents ran simultaneously and produced a unified 850+ line report covering market validation, technical architecture, brand strategy, growth, support, UX, project planning, and a domain-specific section.

**Key insight:** All 8 agents are already installed at `~/.claude/agents/` with full personality definitions, capabilities, and behavioral rules. The skill does NOT need to create agent prompts — it just needs to dispatch to them via `subagent_type` with the product context and section assignment.

## Skill Location

New plugin: `plugins/product-discovery/`

```
plugins/product-discovery/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── product-discovery/
        ├── SKILL.md
        └── references/
            └── report-template.md
```

## SKILL.md Design

### Frontmatter

```yaml
---
name: product-discovery
description: "Run a full multi-agent product discovery exercise. Use when the user wants to explore a product idea, validate a concept, create a product blueprint, or run product discovery. Triggers on: 'product discovery', 'explore this idea', 'validate this product', 'product blueprint', 'full discovery', 'agency discovery', even partial matches like 'discover' + product context."
---
```

### Workflow (3 phases)

**Phase 1: Gather Product Context**
- Extract product name and description from user input
- If the user provided a brief idea, ask 2-3 clarifying questions (target market, key differentiator, domain)
- If they provided a detailed spec, extract context directly
- Determine the domain-specific agent for Section 9 based on product type

**Phase 2: Launch 8 Agents in Parallel**

All agents dispatched via the `Agent` tool with `run_in_background: true`. Each agent already has its full personality and expertise installed — the skill only needs to provide:
- Product name and full description/spec
- The section number and title they're responsible for
- Brief guidance on deliverables for their section
- Instruction to output clean markdown

The agents already know HOW to do their job (market research, system design, brand strategy, etc.) — the skill just tells them WHAT product to analyze and WHERE their output fits in the report.

| # | Section | `subagent_type` | Agent Already Knows How To... |
|---|---------|-----------------|-------------------------------|
| 2 | Market Validation | `Trend Researcher` | Market sizing, competitive analysis, trend detection |
| 3 | Technical Architecture | `Backend Architect` | System design, data models, API design, tech stack selection |
| 4 | Brand Strategy | `Brand Guardian` | Positioning, naming, visual identity, brand voice |
| 5 | Go-to-Market & Growth | `Growth Hacker` | GTM strategy, pricing, launch plans, growth channels |
| 6 | Customer Support Blueprint | `Support Responder` | Support tiers, onboarding, knowledge bases, escalation |
| 7 | UX Research & Design Direction | `UX Researcher` | Personas, journey maps, design principles, usability |
| 8 | Project Execution Plan | `Project Shepherd` | Phase planning, sprints, risk registers, milestones |
| 9 | Domain-Specific | *varies* | Depends on product domain |

Domain agent selection logic (by `subagent_type` name):
- Spatial/XR/VR/AR → `XR Interface Architect`
- AI/ML → `AI Engineer`
- Mobile app → `Mobile App Builder`
- Web app/SaaS → `Frontend Developer`
- Security → `Security Engineer`
- Data/analytics → `Data Engineer`
- Blockchain/web3 → `Solidity Smart Contract Engineer`
- Default → `Software Architect`

**Phase 3: Synthesize and Write Report**

After all 8 agents complete:
1. Write Section 1 (The Opportunity) — summarizing the product concept
2. Collect all agent outputs into their respective sections (2-9)
3. Write Section 10 (Cross-Agent Synthesis) — analyzing across all outputs:
   - Agreements (where agents align)
   - Tensions (where agents conflict)
   - Key risks surfaced by multiple agents
   - Final recommendation (go/no-go/conditional)
   - Phased roadmap combining Project Shepherd's plan with other inputs
4. Save the full report to `docs/discovery/{product-name}-discovery.md`

### Agent Dispatch Pattern

Each agent dispatch is minimal — the agent's installed personality handles the expertise. The prompt just provides context:

```
Product discovery exercise for **{PRODUCT_NAME}**.

Product: {PRODUCT_DESCRIPTION}

Write Section {N}: {SECTION_TITLE} for a comprehensive product discovery report.
Cover: {2-3 bullet points of key deliverables for this section}

Output as clean markdown starting with ## {N}. {SECTION_TITLE}.
Be specific, data-driven, and actionable. Target 80-150 lines.
```

### Report Template

The `references/report-template.md` file provides the skeleton with section headers and deliverable checklists for each section, based on the nexus-spatial-discovery.md format.

## Files to Create

1. **`plugins/product-discovery/.claude-plugin/plugin.json`** — Plugin metadata
2. **`plugins/product-discovery/skills/product-discovery/SKILL.md`** — Main skill file (~200 lines)
3. **`plugins/product-discovery/skills/product-discovery/references/report-template.md`** — Section templates and deliverable checklists (~100 lines)

## Test Cases

After creating the skill, test with these prompts:

1. **Brief idea**: "Run product discovery for a browser extension that uses AI to summarize and organize browser tabs"
2. **Detailed spec**: "Product discovery for CloudKitchen — a SaaS platform that helps ghost kitchen operators manage multiple virtual restaurant brands, with AI-powered menu optimization, demand forecasting, and unified order management across delivery platforms"
3. **Domain-specific (spatial)**: "Run discovery on an AR navigation app for warehouse workers that overlays pick paths and inventory locations in their field of view"

## Verification

- Skill triggers when user says "product discovery", "explore this idea", "product blueprint"
- All 8 agents launch in parallel via `run_in_background: true`
- Each agent uses its installed `subagent_type` (no custom personality prompts needed)
- Report is saved to `docs/discovery/` with all 10 sections
- Cross-Agent Synthesis section references findings from other sections
- Domain-specific agent is correctly chosen based on product type
