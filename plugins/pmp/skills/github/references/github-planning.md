# GitHub Planning

Create GitHub Issues and Projects from implementation plans.

## Modern GitHub Features (2024+)

| Feature | Description | When Used |
|---------|-------------|-----------|
| Native Sub-Issues | GitHub's built-in parent/child relationship | STANDARD, COMPLEX |
| YAML Issue Forms | Structured input templates | All tiers |
| Projects v2 API | GraphQL-based project management | STANDARD, COMPLEX |
| Milestones | Time-boxed release tracking | STANDARD, COMPLEX |
| Label Taxonomy | Structured labeling (type:*, priority:*, status:*) | All tiers |

## Contextd Integration (Optional)

If contextd MCP is available:
- `memory_record` for planning decisions

If contextd is NOT available:
- GitHub artifact creation works normally
- No cross-session memory of planning decisions

## When to Use

Called after plan review is complete to create:
- **SIMPLE:** Single Issue with checklist
- **STANDARD:** Epic Issue + native sub-Issues + Milestone + Projects v2 board
- **COMPLEX:** Epic + native sub-Issues + Projects v2 board + Automation rules

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- Repository has GitHub remote configured
- Complexity tier determined (from complexity-assessment)
- For Projects v2: GitHub GraphQL API access (`gh api graphql`)

---

## Pre-Flight (MANDATORY)

**BEFORE creating ANY GitHub artifacts:**

```
1. Verify gh CLI authentication:
   gh auth status
   → If fails: STOP. User must run `gh auth login`

2. Verify GitHub remote:
   git remote get-url origin
   → Must be a GitHub URL (github.com)

3. Get tier from calling context:
   → If no tier provided: STOP. Complexity assessment required first.

4. Check for existing labels:
   gh label list --json name --jq '.[].name' | grep "type:"
   → If missing: Create label taxonomy (see Label Taxonomy section)

5. If contextd MCP is available:
   mcp__contextd__memory_search(
     project_id: "<project>",
     query: "github planning <feature-name>"
   )
   → Check if Issues already exist for this feature
   → If contextd is NOT available: skip this step (no cross-session memory)

6. Verify GraphQL API access (for STANDARD/COMPLEX tier):
   gh api graphql -f query='{ viewer { login } }'
   → If fails: Fall back to REST API patterns

7. Query existing Projects v2 boards (for STANDARD/COMPLEX tier):
   gh project list --owner <owner> --format json
   → If projects exist: Present list to user with AskQuestion:
     - "Create a new project"
     - "Add to existing project: <project-name> (#<number>)"
   → Store user's choice for use in Projects v2 Setup section
   → If no projects exist: Proceed with new project creation
```

**Do NOT proceed to Issue creation without completing all pre-flight checks.**

---

## Label Taxonomy (Required)

**Create these labels in your repository if they don't exist.** See [config.md](../../pmp/config.md) Label Taxonomy for the full list of labels, colors, and descriptions.

```bash
# Create labels using values from config.md
gh label create "<label>" --color "<color>" --description "<description>"
```

---

## Issue Body Strategy

**ALWAYS write issue bodies to temp files.** Heredocs inside `gh issue create --body` break on markdown with pipes, backticks, angle brackets, and nested quotes. Temp files avoid all shell escaping issues.

**Pattern for every issue:**

```bash
# 1. Write body to temp file
cat > /tmp/issue-body.md << 'ISSUE_BODY'
<markdown body content>
ISSUE_BODY

# 2. Create issue reading from file
gh issue create \
  --title "<title>" \
  --label "<labels>" \
  --body-file /tmp/issue-body.md

# 3. Clean up
rm /tmp/issue-body.md
```

**For GraphQL mutations**, read the temp file into a variable:
```bash
BODY=$(cat /tmp/sub-issue-body.md)
gh api graphql -f query='...' -f body="$BODY" ...
```

**Naming convention for temp files** when creating multiple issues:
- `/tmp/epic-body.md` — the epic
- `/tmp/sub-issue-1-body.md`, `/tmp/sub-issue-2-body.md` — sub-issues
- Clean up ALL temp files after issue creation is complete

---

## Tier-Based Output

### Issue Body Templates

Each tier uses specific templates for issue bodies. Read the template, fill in values from the plan. See [config.md](../../pmp/config.md) Plan ↔ GitHub Mapping for the full hierarchy.

| Level | Template | Label |
|------|----------|-------|
| SIMPLE (single feature) | [assets/issue-simple.md](../assets/issue-simple.md) | `type:feature,priority:medium` |
| Epic (plan) | [assets/issue-epic.md](../assets/issue-epic.md) | `type:epic,priority:high` |
| Feature (sub-issue of epic) | [assets/issue-sub-issue.md](../assets/issue-sub-issue.md) | `type:feature` |
| Task (sub-issue of feature) | [assets/issue-task.md](../assets/issue-task.md) | `type:task` |

### SIMPLE Tier

Create single Issue using [assets/issue-simple.md](../assets/issue-simple.md):

```bash
# Write body from template to temp file, filling in plan values
cat > /tmp/issue-body.md << 'ISSUE_BODY'
# ... fill in assets/issue-simple.md with plan data ...
ISSUE_BODY

gh issue create \
  --title "<feature-name>" \
  --label "type:task,priority:medium" \
  --body-file /tmp/issue-body.md

rm /tmp/issue-body.md
```

### STANDARD Tier

Create Epic Issue + native sub-Issues + Milestone + Projects v2 board.

**1. Create Milestone (if needed):**
```bash
gh api repos/{owner}/{repo}/milestones \
  --method POST \
  -f title="<milestone-name>" \
  -f description="<milestone-description>" \
  -f due_on="<YYYY-MM-DDTHH:MM:SSZ>"
```

**2. Create Epic using [assets/issue-epic.md](../assets/issue-epic.md):**

> **CRITICAL:** For the "Full Implementation Plan" `<details>` section, read the plan file and inline its content directly into the body. Do NOT use `$(cat /path/to/plan.md)` or any local path reference — that exposes your filesystem path in the GitHub Issue and breaks for other readers.

```bash
# Write body from template to temp file, filling in plan values
# INLINE the plan content — do not use $(cat ...) or file path references
cat > /tmp/epic-body.md << 'EPIC_BODY'
# ... fill in assets/issue-epic.md with plan data ...
EPIC_BODY

gh issue create \
  --title "[EPIC] <feature-name>" \
  --label "type:epic,priority:high" \
  --milestone "<milestone-name>" \
  --body-file /tmp/epic-body.md

rm /tmp/epic-body.md
```

**3. Create Feature Issues using [assets/issue-sub-issue.md](../assets/issue-sub-issue.md):**

```bash
# Get Epic node ID first
EPIC_ID=$(gh api graphql -f query='
  query($owner: String!, $repo: String!, $number: Int!) {
    repository(owner: $owner, name: $repo) {
      issue(number: $number) {
        id
      }
    }
  }
' -f owner="<owner>" -f repo="<repo>" -F number=<epic-number> --jq '.data.repository.issue.id')

# Write feature issue body from template to temp file
cat > /tmp/feature-N-body.md << 'FEATURE_BODY'
# ... fill in assets/issue-sub-issue.md with plan data ...
FEATURE_BODY

# Create feature issue with native parent relationship to Epic
BODY=$(cat /tmp/feature-N-body.md)
FEATURE_ID=$(gh api graphql -f query='
  mutation($repositoryId: ID!, $title: String!, $body: String!, $parentId: ID!) {
    createIssue(input: {
      repositoryId: $repositoryId
      title: $title
      body: $body
      parentIssueId: $parentId
    }) {
      issue {
        id
        number
        url
      }
    }
  }
' -f repositoryId="<repo-id>" \
  -f title="Feature N: <feature-name>" \
  -f body="$BODY" \
  -f parentId="$EPIC_ID" \
  --jq '.data.createIssue.issue.id')

rm /tmp/feature-N-body.md
```

**Alternative: REST API for feature issues (if GraphQL unavailable):**
```bash
cat > /tmp/feature-N-body.md << 'FEATURE_BODY'
# ... fill in assets/issue-sub-issue.md with plan data ...
FEATURE_BODY

gh issue create \
  --title "Feature N: <feature-name>" \
  --label "type:feature" \
  --milestone "<milestone-name>" \
  --body-file /tmp/feature-N-body.md

rm /tmp/feature-N-body.md

# Then link via API:
gh api repos/{owner}/{repo}/issues/<feature-issue>/sub_issues \
  --method POST \
  -f parent_issue_id=<epic-id>
```

**4. Create Task Sub-Issues for each Feature using [assets/issue-task.md](../assets/issue-task.md):**

Each feature gets exactly 2 task sub-issues: Task A (implementation) and Task B (E2E tests).

```bash
# For each feature issue, create Task A and Task B as sub-issues

# Task A: Implementation
cat > /tmp/task-N-A-body.md << 'TASK_BODY'
# ... fill in assets/issue-task.md with Task A data ...
TASK_BODY

BODY=$(cat /tmp/task-N-A-body.md)
gh api graphql -f query='
  mutation($repositoryId: ID!, $title: String!, $body: String!, $parentId: ID!) {
    createIssue(input: {
      repositoryId: $repositoryId
      title: $title
      body: $body
      parentIssueId: $parentId
    }) {
      issue { number url }
    }
  }
' -f repositoryId="<repo-id>" \
  -f title="Task A: Implement <feature-name>" \
  -f body="$BODY" \
  -f parentId="$FEATURE_ID"

rm /tmp/task-N-A-body.md

# Task B: E2E Tests
cat > /tmp/task-N-B-body.md << 'TASK_BODY'
# ... fill in assets/issue-task.md with Task B data ...
TASK_BODY

BODY=$(cat /tmp/task-N-B-body.md)
gh api graphql -f query='
  mutation($repositoryId: ID!, $title: String!, $body: String!, $parentId: ID!) {
    createIssue(input: {
      repositoryId: $repositoryId
      title: $title
      body: $body
      parentIssueId: $parentId
    }) {
      issue { number url }
    }
  }
' -f repositoryId="<repo-id>" \
  -f title="Task B: E2E tests for <feature-name>" \
  -f body="$BODY" \
  -f parentId="$FEATURE_ID"

rm /tmp/task-N-B-body.md
```

**5. Set up Projects v2 board** — Proceed to [Projects v2 Setup](#projects-v2-setup-standard--complex) section (STANDARD configuration: Status field + Board view only).

### COMPLEX Tier

Create Epic + feature issues + task sub-issues + Projects v2 board with automation rules.

**1. Create Epic, Feature Issues, and Task Sub-Issues** (same as STANDARD steps 1-4)

**2. Set up Projects v2 board** — Proceed to [Projects v2 Setup](#projects-v2-setup-standard--complex) section (full COMPLEX configuration: all fields, all views, automation rules).

---

## Projects v2 Setup (STANDARD + COMPLEX)

**Shared section for both STANDARD and COMPLEX tiers.** The tier determines which fields, views, and automation are configured.

### 1. Create or Select Project

**If user chose an existing project in pre-flight step 7:** Skip creation, retrieve the existing project ID:
```bash
# Get existing project ID by number
PROJECT_ID=$(gh api graphql -f query='
  query($login: String!, $number: Int!) {
    user(login: $login) {
      projectV2(number: $number) {
        id
        number
      }
    }
  }
' -f login="<owner>" -F number=<project-number> --jq '.data.user.projectV2.id')
```

**If creating a new project:**
```bash
# Get owner ID
OWNER_ID=$(gh api graphql -f query='
  query($login: String!) {
    user(login: $login) { id }
  }
' -f login="<owner>" --jq '.data.user.id')

# Create Project v2
PROJECT_ID=$(gh api graphql -f query='
  mutation($ownerId: ID!, $title: String!) {
    createProjectV2(input: {
      ownerId: $ownerId
      title: $title
    }) {
      projectV2 {
        id
        number
      }
    }
  }
' -f ownerId="$OWNER_ID" -f title="<feature-name>" --jq '.data.createProjectV2.projectV2.id')
```

### 2. Configure Fields

**STANDARD tier — Status field only:**
```bash
# Add Status field (single select)
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2Field(input: {
      projectId: $projectId
      dataType: SINGLE_SELECT
      name: $name
      singleSelectOptions: [
        {name: "Backlog", color: GRAY}
        {name: "Ready", color: BLUE}
        {name: "In Progress", color: YELLOW}
        {name: "In Review", color: PURPLE}
        {name: "Done", color: GREEN}
      ]
    }) {
      projectV2Field { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Status"
```

**COMPLEX tier — Status + Priority + Sprint (all three):**
```bash
# Add Status field (same as STANDARD)
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2Field(input: {
      projectId: $projectId
      dataType: SINGLE_SELECT
      name: $name
      singleSelectOptions: [
        {name: "Backlog", color: GRAY}
        {name: "Ready", color: BLUE}
        {name: "In Progress", color: YELLOW}
        {name: "In Review", color: PURPLE}
        {name: "Done", color: GREEN}
      ]
    }) {
      projectV2Field { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Status"

# Add Priority field
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2Field(input: {
      projectId: $projectId
      dataType: SINGLE_SELECT
      name: $name
      singleSelectOptions: [
        {name: "P0 Critical", color: RED}
        {name: "P1 High", color: ORANGE}
        {name: "P2 Medium", color: YELLOW}
        {name: "P3 Low", color: GREEN}
      ]
    }) {
      projectV2Field { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Priority"

# Add Sprint/Iteration field
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2Field(input: {
      projectId: $projectId
      dataType: ITERATION
      name: $name
    }) {
      projectV2Field { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Sprint"
```

### 3. Create Views

**STANDARD tier — Board view only:**
```bash
# Create Board View
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2View(input: {
      projectId: $projectId
      name: $name
      layout: BOARD_LAYOUT
    }) {
      projectV2View { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Board"
```

**COMPLEX tier — Board + Table + Roadmap:**
```bash
# Create Board View
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2View(input: {
      projectId: $projectId
      name: $name
      layout: BOARD_LAYOUT
    }) {
      projectV2View { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Board"

# Create Table View
gh api graphql -f query='
  mutation($projectId: ID!, $name: String!) {
    createProjectV2View(input: {
      projectId: $projectId
      name: $name
      layout: TABLE_LAYOUT
    }) {
      projectV2View { id }
    }
  }
' -f projectId="$PROJECT_ID" -f name="Table"
```
> **Continued in [github-planning-verification.md](github-planning-verification.md)** — Remaining Projects v2 setup (views, issue linking, automation), YAML Issue Form Templates, PR Template, Verification Criteria, Bidirectional Linking, contextd Integration, Error Handling, Cleanup Protocol, Mandatory Checklist, Red Flags, and Common Mistakes.
