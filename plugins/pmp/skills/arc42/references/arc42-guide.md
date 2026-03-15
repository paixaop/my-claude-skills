# Arc42 Section Guide

Reference for arc42's 12 sections — what belongs in each, recommended form, key tips from the [arc42 documentation](https://docs.arc42.org). Used during Phase 2 (content classification) and Phase 4 (structuring output files).

Source: [arc42.org](https://arc42.org/overview) | [docs.arc42.org](https://docs.arc42.org)

---

## Section 1: Introduction & Goals

**Content:** Relevant requirements and driving forces — business goals, essential features, functional requirements, quality goals, stakeholder expectations.

**Subsections:**
- **1.1 Requirements Overview** — short textual descriptions, possibly tabular use-case format
- **1.2 Quality Goals** — prioritized table of top 3-5 quality goals with concrete scenarios
- **1.3 Stakeholders** — table with role names, contacts, expectations

**Tips:**
- Keep compact — less than one page if possible (Tip 1-1)
- Focus on essential tasks and business goals (Tip 1-2)
- Describe only top 3-5 quality goals in 1.2 (Tip 1-16)
- Use scenarios to explain quality requirements (Tip 1-12)
- Use checklists (ISO 25010) for quality topics (Tip 1-14)
- Make assumptions explicit (Tip 1-9)
- Maintain stakeholder table with role, contact, expectations (Tip 1-20)

---

## Section 2: Constraints

**Content:** Anything that constrains design and implementation decisions or development process.

**Subsections:**
- **Technical constraints** — technology mandates, platform requirements, compatibility
- **Organizational constraints** — team structure, timeline, budget, process
- **Conventions** — coding guidelines, naming standards, versioning, documentation standards

**Tips:**
- Subdivide into technical, organizational, and conventions (Tip 2-5)
- Consider constraints from neighboring/interfacing systems (Tip 2-1)
- Clarify consequences of each constraint (Tip 2-2)
- Document organizational constraints explicitly (Tip 2-3)
- Note which constraints are negotiable vs. fixed

---

## Section 3: Context & Scope

**Content:** System scope and boundaries — neighboring systems, users, external interfaces.

**Subsections:**
- **3.1 Business Context** — domain-specific inputs/outputs, communication partners
- **3.2 Technical Context** — channels, protocols, hardware, deployment topology

**Tips:**
- Explicitly demarcate system from environment (Tip 3-1)
- Show context as diagram + explanatory table (Tip 3-2, 3-3)
- Restrict to overview — avoid excessive detail (Tip 3-5)
- Show data flows, not just dependencies (Tip 3-11)
- Show all external interfaces (Tip 3-9)
- Differentiate business and technical context (Tip 3-10)
- Pay attention to quality requirements at interfaces (Tip 3-14)

---

## Section 4: Solution Strategy

**Content:** Fundamental design decisions and solution strategies — technology choices, top-level decomposition, approaches to achieve quality goals.

**Form:** Table mapping quality goals → solution approaches → section references.

**Tips:**
- Keep compact — use keywords as shorthand (Tip 4-1)
- Present approaches in table format (Tip 4-2)
- Connect approaches explicitly to quality requirements (Tip 4-3)
- Reference concepts, views, or code for support (Tip 4-4)
- Allow strategy to develop iteratively (Tip 4-5)
- Always justify chosen solution strategy (Tip 4-6)

---

## Section 5: Building Block View

**Content:** Static decomposition of the system into building blocks and their dependencies. Hierarchical levels from context to implementation.

**Structure:**
- **Level 1** — whitebox of overall system, showing top-level building blocks as black boxes
- **Level 2** — zoom into selected Level 1 blocks with whitebox detail
- **Level 3+** — further refinement as needed

**Black box** = responsibility + interfaces (what it does, not how).
**White box** = internal structure (how it works inside).

**Tips:**
- Always describe Level 1 — "Level-1 is your friend" (Tip 5-3)
- Use common structures for all building block sections (Tip 5-1)
- Organize hierarchically (Tip 5-2)
- Prefer relevance over completeness — document important, surprising, risky, or complex blocks (Tip 5-4)
- Describe block responsibilities clearly (Tip 5-5)
- Hide inner workings in black box descriptions (Tip 5-6)
- Use tables for concise documentation (Tip 5-7)
- Show multiple levels only when there is real demand (Tip 5-11)
- Map building blocks to source code (Tip 5-13)

---

## Section 6: Runtime View

**Content:** How building blocks interact during execution — important use cases, critical external interface interactions, error/exception handling, startup/shutdown procedures.

**Form:** Named scenarios using:
- Natural language step lists
- Activity diagrams or flowcharts
- Sequence diagrams
- BPMN or event process chains
- State machines

**Tips:**
- Map activities to existing building blocks (Tip 6-1)
- Document only representative scenarios — prioritize architectural relevance (Tip 6-2)
- Use schematic, not detailed scenarios (Tip 6-3)
- Document detailed scenarios cautiously — high maintenance cost (Tip 6-4)
- Describe partial scenario excerpts when full scenarios are too large (Tip 6-6)
- Use activity diagrams with swimlanes for multi-component interactions (Tip 6-7)
- Include both small and large building blocks (Tip 6-10)

---

## Section 7: Deployment View

**Content:** Technical infrastructure and how software maps to it.

**Subsections:**
- **Infrastructure topology** — nodes, channels, geographic locations
- **Software-to-hardware mapping** — which building blocks run where
- **Environment descriptions** — dev, staging, production differences

**Tips:**
- Document technical infrastructure/hardware (Tip 7-1)
- Explain hardware and infrastructure decisions (Tip 7-2)
- Document multiple environments (dev, staging, production) (Tip 7-3)
- Document hierarchically if infrastructure is complex (Tip 7-4)
- Document mapping of building blocks to hardware (Tip 7-5)
- Use UML deployment diagrams or tables for mapping (Tip 7-6, 7-7)
- Explain your nodes — what runs on each (Tip 7-8)

---

## Section 8: Crosscutting Concepts

**Content:** Practices, patterns, and solutions spanning multiple building blocks — the "how we do things" that cuts across components.

**Topics:** Security, authentication, authorization, error handling, logging, persistence, UI patterns, i18n, configuration management, testing strategy, coding conventions.

**Form:** Concept papers per topic, with optional code examples.

**Tips:**
- Make concepts clear and understandable (Tip 8-1)
- Concepts include approaches, rules, principles, tactics, strategies (Tip 8-2)
- Document only critical topics — don't try to cover everything (Tip 8-3)
- Support concepts with source code examples for technical topics (Tip 8-8)
- Prioritize documenting decisions over general concepts (Tip 8-9)
- Create hyperlinks between building blocks and concepts (Tip 8-11)
- Include business/domain models (Tip 8-5)
- Include data models as essential content (Tip 8-7)

---

## Section 9: Architecture Decisions

**Content:** Important, expensive, or risky architecture decisions with rationale.

**Form:** Architecture Decision Records (ADRs):
```
### ADR-NNN: [Title]
**Status:** proposed | accepted | deprecated | superseded
**Context:** [Forces and tensions that led to this decision]
**Decision:** [What was decided]
**Consequences:** [Positive and negative outcomes]
```

Or: decision tables organized by importance/consequences.

**Tips:**
- Focus on architecturally relevant decisions only (Tip 9-1)
- Include explicit decision criteria (Tip 9-2)
- Explain reasoning behind significant decisions (Tip 9-3)
- Use tables or mind-maps for overview (Tip 9-4)
- Use ADR format for formal documentation (Tip 9-5)
- Document rejected alternatives (Tip 9-6)
- Add timestamps on decisions (Tip 9-8)
- Avoid redundancy with Section 4 (Solution Strategy)

---

## Section 10: Quality Requirements

**Content:** All quality requirements, expanding on the top 3-5 from Section 1.2.

**Subsections:**
- **10.1 Quality Overview** — table categorized by quality topics (use ISO 25010:2023 or Q42 model)
- **10.2 Quality Scenarios** — concrete, measurable scenarios

**Quality Scenario Format:**
| Field | Content |
|-------|---------|
| Scenario ID | QS-NNN |
| Name | [descriptive name] |
| Source | [who/what triggers] |
| Stimulus | [event or condition] |
| Environment | [system state when triggered] |
| Artifact | [component affected] |
| Response | [expected system behavior] |
| Response Measure | [how to verify] |

**Tips:**
- Use overview table for category-level summary (Tip 10-1)
- Include usage/application scenarios (Tip 10-5)
- Include change scenarios — how the system handles modifications (Tip 10-6)
- Include failure scenarios (Tip 10-7)
- Use scenarios as verification checklist during architecture evaluation (Tip 10-8)

---

## Section 11: Risks & Technical Debt

**Content:** Prioritized inventory of identified technical risks and technical debts.

**Form:** Structured list with:
- Risk/debt description
- Severity/priority
- Proposed mitigation strategies or reduction approaches

**Tips:**
- Engage different stakeholder groups when searching for risks (Tip 11-1)
- Examine external interfaces as potential risk sources (Tip 11-2)
- Use qualitative evaluation methods systematically (Tip 11-3)
- Assess operational and business processes for embedded problems (Tip 11-4)
- Review data structures and handling for potential issues (Tip 11-5)

---

## Section 12: Glossary

**Content:** Domain and technical terms that stakeholders use when discussing the system.

**Form:** Table with columns:
| Term | Definition |
|------|-----------|
| [term] | [definition] |

Optional additional columns: Synonyms, Translation (for multilingual teams).

**Tips:**
- Treat glossary as essential documentation (Tip 12-1)
- Use tabular format (Tip 12-2)
- Add graphical models for complex concepts (Tip 12-3)
- Add translation columns for international teams (Tip 12-4)
- Keep concise — exclude trivial or irrelevant terms (Tip 12-5)
- Assign maintenance ownership to product owners/project managers (Tip 12-6)

---

## Content Classification Guide

Use this table during Phase 2 to map source content to arc42 sections:

| Source Content Pattern | Arc42 Section |
|----------------------|---------------|
| Business goals, feature lists, user stories, requirements, project vision | **1 — Introduction & Goals** |
| Technology mandates, regulatory requirements, coding standards, team constraints | **2 — Constraints** |
| System boundaries, external APIs, actors, neighboring systems, integration points | **3 — Context & Scope** |
| "Why we chose X", technology rationale, approach justification, design philosophy | **4 — Solution Strategy** |
| Component descriptions, module APIs, data models, class hierarchies, package structure | **5 — Building Block View** |
| Request flows, sequence diagrams, state machines, use case walkthroughs, event processing | **6 — Runtime View** |
| Infrastructure, Docker/K8s configs, CI/CD, environment descriptions, deployment topology | **7 — Deployment View** |
| Security policies, auth patterns, error handling, logging, i18n, persistence strategy | **8 — Crosscutting Concepts** |
| ADRs, "we decided to...", design trade-offs, rejected alternatives, decision rationale | **9 — Architecture Decisions** |
| SLAs, performance targets, quality attributes, test criteria, quality scenarios | **10 — Quality Requirements** |
| Known issues, tech debt backlog, risk registers, deferred work, migration concerns | **11 — Risks & Technical Debt** |
| Term definitions, abbreviations, domain vocabulary, acronym lists | **12 — Glossary** |
| Content that doesn't fit any section above | **99 — Appendix** |
