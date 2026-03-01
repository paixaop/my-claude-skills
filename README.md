# My Claude Skills

A [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces) with plugins for planning and security analysis. Install them individually.

## Installation

Add the marketplace inside a Claude Code session:

```shell
/plugin marketplace add paixaop/my-claude-skills
```

Then install the plugins you want:

```shell
/plugin install pmp@my-claude-skills
/plugin install security-analyst@my-claude-skills
```

### Local / Development

Test a plugin locally without installing:

```bash
claude --plugin-dir ./plugins/pmp
claude --plugin-dir ./plugins/security-analyst
```

## Plugins

### PMP (Plan-Management Pipeline)

Full planning lifecycle from brainstorm to execution. Covers ideation, plan generation, review, GitHub Issues publishing, and implementation with coordinated agent teams.

**Workflows:**
- **Brainstorm** — Explore ideas, define requirements, and produce design documents
- **Plan** — Generate detailed implementation plans from specs, roadmaps, or GitHub Issues
- **Review** — Structured review and approval of plans before execution
- **GitHub Planning** — Create epics and sub-issues, sync changes back to GitHub
- **Execute** — Run implementation with parallel agents and task dependencies
- **E2E Loop** — Code-test-fix cycles with multiple testing strategies

**Trigger phrases:**
> "plan", "brainstorm", "design this", "create a plan", "review plan", "execute plan", "plan from roadmap", "plan from issues", "plan from epic", "e2e loop", "code-test-fix", "run e2e tests", "extend plans", "create issues", "publish to GitHub", "make an epic", "update issues", "sync issues"

**Slash commands:**
```
/pmp:pmp                          # Interactive mode
/pmp:pmp brainstorm               # Start from an idea
/pmp:pmp plan-spec                # Generate plan from a spec or roadmap
/pmp:pmp plan-issues              # Generate plan from GitHub Issues
/pmp:pmp review                   # Review an existing plan
/pmp:pmp execute                  # Execute a plan
/pmp:pmp e2e                      # Run E2E test loop
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

**Trigger phrases:**
> "security audit", "penetration test", "pentest", "threat model", "vulnerability hunt", "find vulnerabilities", "security fix plan", "SBOM", "compliance mapping", "privacy assessment", "security posture", "security review", "attack surface"

**Slash commands:**
```
/security-analyst:security-analyst                       # Interactive mode
/security-analyst:security-analyst full                  # All 9 phases
/security-analyst:security-analyst focused [component]   # Targeted analysis
/security-analyst:security-analyst recon                 # Reconnaissance only
/security-analyst:security-analyst threat-model          # STRIDE + attack trees
/security-analyst:security-analyst sbom                  # Dependencies and supply chain
/security-analyst:security-analyst compliance [framework] # SOC 2, ISO 27001, PCI DSS, etc.
/security-analyst:security-analyst privacy               # PII and data privacy assessment
/security-analyst:security-analyst fix-plan [run-dir]    # Generate fix plan from findings
/security-analyst:security-analyst diff [run-a] [run-b]  # Compare two runs
/security-analyst:security-analyst variant-hunt [vuln]   # Find all variants of a known vuln
```

## Structure

```
.claude-plugin/
  marketplace.json              # Marketplace catalog
plugins/
  pmp/                          # Plan-Management Pipeline plugin
    .claude-plugin/plugin.json
    skills/pmp/
  security-analyst/             # Security analysis plugin
    .claude-plugin/plugin.json
    skills/security-analyst/
```

## Reference

The repository includes `skills-complete-guide.md` — a comprehensive reference covering skill architecture, design patterns, testing, distribution, and MCP integration.
