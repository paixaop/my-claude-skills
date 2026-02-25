# My Claude Code Skills

A collection of custom skills for [Claude Code](https://docs.anthropic.com/en/docs/claude-code) that extend its capabilities with structured workflows for planning, security analysis, and skill authoring.

## Skills

### PMP (Plan-Management Pipeline)

Full planning lifecycle from brainstorm to execution. Covers ideation, plan generation, review, GitHub Issues publishing, and implementation with coordinated agent teams.

**Workflows:**
- **Brainstorm** — Explore ideas, define requirements, and produce design documents
- **Plan** — Generate detailed implementation plans from specs, roadmaps, or GitHub Issues
- **Review** — Structured review and approval of plans before execution
- **GitHub Planning** — Create epics and sub-issues, sync changes back to GitHub
- **Execute** — Run implementation with parallel agents and task dependencies
- **E2E Loop** — Code-test-fix cycles with multiple testing strategies

**Usage:**
```
/pmp                    # Interactive mode
/pmp:brainstorm         # Start from an idea
/pmp:plan-spec          # Generate plan from a spec or roadmap
/pmp:plan-issues        # Generate plan from GitHub Issues
/pmp:review             # Review an existing plan
/pmp:execute            # Execute a plan
/pmp:e2e               # Run E2E test loop
```

### Security Analyst

Offensive security analysis suite that runs multi-phase penetration testing with parallel agents. Finds real vulnerabilities with concrete exploits — not compliance checklists.

**9-Phase Pipeline:**
1. **Recon** — Build a security map of the codebase
2. **Attack Surface** — Analyze HTTP, API, auth, frontend, integrations, and git history
3. **SBOM** — Software Bill of Materials and supply chain review
4. **Business Logic** — Race conditions, IDOR, pipeline exploits, DoS vectors
5. **Data Flow Tracing** — Follow sensitive data through the system
6. **Exploit Development** — Build proof-of-concept exploits for findings
7. **Validation** — Adversarial critic review to eliminate false positives
8. **Reporting** — Executive summary with CVSS scores, CWEs, and MITRE ATT&CK mapping
9. **Remediation** — Actionable fix plan with code patches and regression tests

**Features:**
- Auto-detects 16 frameworks (FastAPI, Django, Express, Next.js, Spring Boot, Rails, etc.)
- LOD (Level of Detail) system saves 80-96% of prompt tokens
- Supports compliance mapping (SOC 2, ISO 27001, PCI DSS, HIPAA, GDPR)
- Privacy assessments for PII flows and data subject rights
- Diff mode to compare security posture between runs

**Usage:**
```
/security-analyst                       # Interactive mode
/security-analyst:full                  # All 9 phases
/security-analyst:focused [component]   # Targeted analysis
/security-analyst:recon                 # Reconnaissance only
/security-analyst:threat-model          # STRIDE + attack trees
/security-analyst:sbom                  # Dependencies and supply chain
/security-analyst:compliance [framework] # SOC 2, ISO 27001, PCI DSS, etc.
/security-analyst:privacy               # PII and data privacy assessment
/security-analyst:fix-plan [run-dir]    # Generate fix plan from findings
/security-analyst:diff [run-a] [run-b]  # Compare two runs
/security-analyst:variant-hunt [vuln]   # Find all variants of a known vuln
```

### Create Skill

A skill for creating new Claude Code skills. Provides the official format, best practices, and validation guidance from Anthropic's skill architecture.

**Covers:**
- SKILL.md structure and YAML frontmatter
- Progressive disclosure (three-level context loading)
- String substitution and dynamic context
- Subagent execution patterns
- Skill categories and design principles

**Usage:**
```
/create-skill [skill-name] [purpose]
```

## Installation

These skills are located in `~/.claude/skills/` and are automatically available to Claude Code. No additional setup is required.

## Reference

The repository includes `skills-complete-guide.md` — a comprehensive reference covering skill architecture, design patterns, testing, distribution, and MCP integration.
