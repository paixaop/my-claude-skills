# GitHub Planning — Verification & Reference

> Part 2 of GitHub Planning. See [github-planning.md](github-planning.md) for Pre-Flight, Label Taxonomy, Issue Body Strategy, Tier-Based Output, and Projects v2 Setup.

---

## Projects v2 Setup (continued)

### Roadmap View (COMPLEX only)

```bash
# Create Roadmap View
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2View(input: {
      projectId: $projectId
      name: $name
      layout: ROADMAP_LAYOUT
    }) {
      projectV2View { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Roadmap"
```

### 4. Add Issues to Project

**Same for both tiers:**
```bash
# Batch add multiple issues
for ISSUE_URL in "<epic-url>" "<sub-issue-1-url>" "<sub-issue-2-url>"; do
  gh project item-add <project-number> --owner <owner> --url "$ISSUE_URL"
done

# Or via GraphQL for more control:
gh api graphql -f query='
  mutation($projectId: ID!, $contentId: ID!) {
    addProjectV2ItemById(input: {
      projectId: $projectId
      contentId: $contentId
    }) {
      item { id }
    }
  }
' -f projectId="$PROJECT_ID" -f contentId="<issue-node-id>"
```

### 5. Configure Automation (COMPLEX only)

**Skip this step for STANDARD tier.**

```bash
# Auto-move to "In Progress" when assigned
gh api graphql -f query='
  mutation($projectId: ID!) {
    createProjectV2Workflow(input: {
      projectId: $projectId
      name: "Auto-move on assign"
      enabled: true
      triggers: [ISSUE_ASSIGNED]
    }) {
      projectV2Workflow { id }
    }
  }
' -f projectId="$PROJECT_ID"

# Auto-move to "Done" when closed
gh api graphql -f query='
  mutation($projectId: ID!) {
    createProjectV2Workflow(input: {
      projectId: $projectId
      name: "Auto-close on done"
      enabled: true
      triggers: [ISSUE_CLOSED]
    }) {
      projectV2Workflow { id }
    }
  }
' -f projectId="$PROJECT_ID"
```

---

## YAML Issue Form Templates

Store these in `.github/ISSUE_TEMPLATE/` for structured issue creation. Templates are in the `assets/` directory:

| Form | Template File | Destination |
|------|---------------|-------------|
| Feature Request | [assets/yaml-feature-form.yml](../assets/yaml-feature-form.yml) | `.github/ISSUE_TEMPLATE/feature.yml` |
| Bug Report | [assets/yaml-bug-form.yml](../assets/yaml-bug-form.yml) | `.github/ISSUE_TEMPLATE/bug.yml` |
| Epic | [assets/yaml-epic-form.yml](../assets/yaml-epic-form.yml) | `.github/ISSUE_TEMPLATE/epic.yml` |

---

## PR Template

Create `.github/pull_request_template.md` using [assets/pr-body.md](../../execute/assets/pr-body.md).

---

## Automated Verification Criteria Generation

**For EVERY task, generate verification criteria automatically based on task type:**

### Detection Rules

| Task Pattern | Verification Type | Auto-Generated Command |
|--------------|------------------|------------------------|
| `*test*`, `*spec*` | command | `npm test -- --grep "<pattern>"` |
| `*API*`, `*endpoint*` | api | `curl -X <method> <endpoint> \| jq '.'` |
| `*UI*`, `*component*`, `*page*` | browser | "Navigate to <path>, verify <element>" |
| `*config*`, `*setting*` | manual | "Verify in settings dashboard" |
| `*CLI*`, `*command*` | command | `<command> --help && <command> <args>` |
| `*integration*` | e2e | `npm run test:e2e -- --grep "<feature>"` |
| `*migration*`, `*database*` | command | `npm run migrate && npm run db:verify` |

### CI Check Integration

Link verification to CI checks:
```markdown
## Verification
| Type | Details | Expected | CI Check |
|------|---------|----------|----------|
| command | `npm test` | All pass | [![Tests](badge-url)](check-url) |
| lint | `npm run lint` | No errors | [![Lint](badge-url)](check-url) |
| build | `npm run build` | Success | [![Build](badge-url)](check-url) |
```

---

## Bidirectional Linking (Required for STANDARD/COMPLEX)

### Native Sub-Issues (Preferred - GitHub 2024+)

When using GraphQL `parentIssueId`:
- Sub-issues automatically appear in Epic's "Sub-issues" section
- No manual linking required
- GitHub UI shows full hierarchy

### Manual Linking (Fallback)

**Epic body MUST contain:**
```markdown
## Sub-Issues
- #43: Implement auth middleware
- #44: Add login component
- #45: Create user model
```

**Each sub-Issue body MUST contain:**
```markdown
## Parent
Contributes to #42
```

**Verification:** After creation, both Epic and sub-Issues should show cross-references in GitHub UI.

---

## contextd Integration

Record created artifacts:

```
mcp__contextd__memory_record(
  project_id: "<project>",
  title: "GitHub planning: <feature-name>",
  content: "Created: <issue-type>. URLs: <list>.
            Tier: <tier>. Tasks: <count>.
            Project: <project-url> (if STANDARD+).
            Milestone: <milestone-name> (if STANDARD+).",
  outcome: "success",
  tags: ["github-planning", "<tier>"]
)
```

---

## Error Handling

### gh CLI not authenticated
```
Error: gh auth status failed
Action: Run `gh auth login` to authenticate
```

### No GitHub remote
```
Error: No GitHub remote found
Action: Ensure repo has origin pointing to GitHub
```

### Rate limiting
```
Error: API rate limit exceeded
Action: Wait and retry, or use authenticated requests
```

### GraphQL API errors
```
Error: GraphQL query failed
Action: Check API permissions, verify node IDs are correct
Fallback: Use REST API alternatives where available
```

### Native sub-issues not available
```
Error: parentIssueId not supported
Action: Repository may not have sub-issues enabled
Fallback: Use manual linking with "Contributes to #X" pattern
```

---

## Cleanup Protocol

**If planning fails or is abandoned:**

```bash
# Close all created Issues
gh issue close <epic-number> --comment "Planning abandoned"
gh issue close <sub-issue-1> --comment "Parent epic closed"
gh issue close <sub-issue-2> --comment "Parent epic closed"

# Remove from Project (if STANDARD or COMPLEX)
gh project item-delete <project-number> --owner <owner> --id <item-id>

# Or delete entire project if nothing was added
gh api graphql -f query='
  mutation($projectId: ID!) {
    deleteProjectV2(input: { projectId: $projectId }) {
      projectV2 { id }
    }
  }
' -f projectId="$PROJECT_ID"
```

**Record in contextd:**
```
mcp__contextd__memory_record(
  project_id: "<project>",
  title: "GitHub planning abandoned: <feature-name>",
  content: "Reason: <reason>. Closed: #<numbers>.
            Project deleted: <yes/no>.",
  outcome: "failure",
  tags: ["github-planning", "abandoned"]
)
```

## Attribution

Adapted from Auto-Claude planning concepts. See CREDITS.md.

---

## Mandatory Checklist

**EVERY github-planning invocation MUST complete ALL steps:**

### For ALL Tiers:
- [ ] Run pre-flight checks (gh auth, remote, tier, labels)
- [ ] Use the correct body template for the tier (see Issue Body Templates table)
- [ ] Include Plan Reference section (plan file path, goal, feature number) in EVERY Issue
- [ ] Include Behavior description from the plan in EVERY Issue
- [ ] Include Affected Files from the plan in EVERY Issue
- [ ] Include Acceptance Criteria from the plan as checkboxes in EVERY Issue
- [ ] Generate verification criteria for EACH task (see Verification Criteria section)
- [ ] Include verification table in EVERY Issue body
- [ ] Include Definition of Done checklist in EVERY Issue
- [ ] Include Source Context section with verbatim spec/roadmap excerpt in EVERY Issue
- [ ] Apply appropriate label taxonomy (type:*, priority:*)
- [ ] Record created artifacts in contextd

### For SIMPLE Tier:
- [ ] Create single Issue with checklist using [assets/issue-simple.md](../assets/issue-simple.md)
- [ ] Include all tasks as checkboxes
- [ ] Add verification criteria table
- [ ] Add Definition of Done checklist
- [ ] Add complexity and source metadata

### For STANDARD Tier:
- [ ] Create Milestone (if feature is time-boxed)
- [ ] Create Epic Issue FIRST using [assets/issue-epic.md](../assets/issue-epic.md) with type:epic label
- [ ] Embed the FULL implementation plan in the Epic body (collapsible `<details>` section)
- [ ] Create each Feature Issue using [assets/issue-sub-issue.md](../assets/issue-sub-issue.md) with type:feature label and native parent relationship to Epic (GraphQL)
- [ ] Create Task A and Task B sub-issues for each Feature using [assets/issue-task.md](../assets/issue-task.md) with type:task label and native parent relationship to Feature
- [ ] Each Feature Issue Plan Reference points to the Epic (not the plan file) for the full plan
- [ ] If native sub-issues unavailable: use REST fallback with manual linking
- [ ] Verify bidirectional linking (epic <-> features <-> tasks) visible in GitHub UI
- [ ] Add all issues to milestone

### For STANDARD + COMPLEX (Projects v2):
- [ ] If existing projects found, asked user: new or existing?
- [ ] If existing project selected, verified project accessible
- [ ] Skipped project creation steps if using existing project
- [ ] Create or select Projects v2 board
- [ ] Add Epic and ALL feature/task Issues to Project
- [ ] Verify all Issues appear in Project

### For STANDARD Tier (Projects v2 — simplified config):
- [ ] Configure Status field
- [ ] Create Board view

### For COMPLEX Tier:
- [ ] Complete ALL STANDARD tier steps
- [ ] Configure all Project fields (Status, Priority, Sprint)
- [ ] Create all Project views (Board, Table, Roadmap)
- [ ] Configure automation rules (auto-move on assign/close)
- [ ] Verify all Issues appear in Project with correct field values

### Plan Annotation (after all issues created):
- [ ] **Update frontmatter:** Set `status: issues_created`, `issues_created_at` to current UTC timestamp, and `epic: "#<number>"` (see [config.md](../../pmp/config.md) Plan Frontmatter)
- [ ] Add `**Epic:** #<number>` to the plan header
- [ ] Add/update the `## GitHub Issues` table in the plan file (3-level: Epic > Feature > Task)
- [ ] Annotate Feature Dependency Graph: append `(#<number>)` to each feature
- [ ] Annotate every feature heading: `## Feature N: [Name]` -> `## Feature N: [Name] · #<number>`
- [ ] Annotate every feature's Tasks table with task issue numbers
- [ ] Verify every feature and task in the plan has its issue number inline

### Post-Creation:
- [ ] Output Issue URLs to user
- [ ] Output Project URL (if STANDARD or COMPLEX)
- [ ] Record in contextd with all URLs and project details
- [ ] If ANY step fails, record failure and cleanup

**GitHub planning is NOT complete until plan annotation AND contextd recording are done.**

---

## Red Flags - STOP and Reconsider

| Thought | Reality |
|---------|---------|
| "Skip pre-flight, I know gh is working" | Pre-flight catches stale tokens. Always check. |
| "Verification is optional for simple tasks" | Verification is NEVER optional. Always include. |
| "I'll update the Epic links later" | Bidirectional linking is immediate. Do it now. |
| "COMPLEX doesn't need a Project board" | COMPLEX REQUIRES Projects v2 board. No exceptions. |
| "contextd recording can wait" | Recording is the LAST mandatory step. Don't skip. |
| "One Issue is enough for STANDARD" | STANDARD = Epic + native sub-Issues. Always. |
| "I'll just create Issues without verification" | Every Issue needs verification criteria AND DoD. |
| "I'll use $(cat /path/to/plan.md) in the body" | NEVER embed local paths. Read the file and inline its content. |
| "Source context isn't needed" | Every Issue MUST quote the spec/roadmap excerpt it came from. |
| "A brief description is enough" | Issues must carry the plan's full feature structure: behavior, files, ACs, API contract. |
| "Plan reference isn't important" | Every Issue MUST link back to the plan file, goal, and feature number. |
| "The full plan is too long for the Epic" | Use a collapsible `<details>` section. The Epic IS the plan's home in GitHub. |
| "STANDARD doesn't need a Project board" | STANDARD REQUIRES Projects v2 board. Only SIMPLE skips it. |
| "I'll create a new project even though one exists" | Check for existing projects first. User may want to consolidate. |
| "The user didn't ask for a Project board" | Tier determines artifacts. STANDARD + COMPLEX = Project. |
| "Labels aren't important" | Label taxonomy enables filtering and automation. |
| "Skip GraphQL, REST is enough" | GraphQL enables native sub-issues and Projects v2. |
| "Automation rules are optional" | COMPLEX tier requires automation configuration. |

---

## Common Mistakes

| Mistake | Correct Approach |
|---------|------------------|
| Creating sub-Issues without native parent link | Use GraphQL parentIssueId or REST sub_issues API |
| Creating Epic without updating links | Native sub-issues auto-link; verify in GitHub UI |
| Skipping verification criteria | Use Verification Criteria section to generate for each task type |
| COMPLEX without Project board | Create Projects v2 via GraphQL with fields and views |
| Not recording in contextd | memory_record with ALL Issue/Project URLs is mandatory |
| Creating duplicate projects | Check for existing projects in pre-flight, ask user preference |
| STANDARD without Project board | STANDARD requires Projects v2 board (simplified config: Status field + Board view) |
| Using wrong tier artifacts | SIMPLE = single Issue, STANDARD = Epic+subs+Milestone+Project, COMPLEX = Epic+subs+Project+automation |
| Orphaned Issues on failure | If error mid-creation, cleanup with gh issue close |
| Missing label taxonomy | Create type:*, priority:*, status:* labels before issue creation |
| Not including Definition of Done | Every issue needs DoD checklist for completion criteria |
| Missing source context | Every issue must include the verbatim spec/roadmap excerpt that generated it |
| Thin issue bodies with just a title and description | Issues must mirror the plan's feature structure: Plan Reference, Behavior, Affected Files, ACs, Verification, DoD |
| No reference to the plan file | Every issue needs a Plan Reference section with file path, goal, and feature number |
| Epic without the full plan embedded | The Epic MUST contain the full plan in a collapsible section |
| Sub-issues duplicating the plan file path | Sub-issues reference the Epic number for the full plan, not the file path |
| Skipping Project automation | COMPLEX tier requires workflow automation rules |
