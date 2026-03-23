# Spec Alignment Analysis

Cross-reference implementation plans against their source specifications. Identifies gaps, contradictions, ambiguities, and incomplete file coverage.

## When to Run

- **Mandatory** when the plan's `Source:` field points to a local file or directory
- **Skip** when `Source:` is missing, set to `"Inline spec"`, or a non-local reference (GitHub URL, issue number). Note in output: `Spec alignment: skipped (no local spec source)`
- If `Source:` points to a directory, read all markdown files in it recursively
- If `Source:` points to a single file, read that file

---

## Step 1: Extract from the Spec

Read all spec files and extract these categories:

### Requirements
- Explicit MUST / SHOULD / MAY statements
- Numbered requirements or user stories
- Acceptance criteria defined in the spec
- Constraints (performance targets, compatibility, limits)

### Behaviors
- Described workflows and user journeys
- Expected outcomes and success conditions
- Error handling and edge cases

### Entities
- Data models, schemas, tables
- API contracts (endpoints, inputs, outputs, error codes)
- Interfaces and integration points

### Constraints
- Security requirements (auth, encryption, access control)
- Performance targets (latency, throughput, resource limits)
- Compatibility requirements (browsers, platforms, versions)

Assign each extracted item a short identifier (e.g., `REQ-1`, `BEH-3`, `ENT-2`) for traceability.

---

## Step 2: Extract from the Plan

For each feature in the plan, extract:

1. **Behavior** — the feature's described behavior and user-visible outcomes
2. **Affected Files** — files listed in the feature's "Affected Files" section
3. **Acceptance Criteria** — ACs listed for the feature
4. **API Contracts** — contracts defined in the feature (if any)
5. **Data Model** — schema changes in the feature (if any)

---

## Step 3: Build the Traceability Matrix

Map every spec item to plan feature(s):

| ID | Spec Requirement | Plan Feature(s) | Coverage | Gap |
|----|-----------------|-----------------|----------|-----|
| REQ-1 | [requirement text] | Feature N | Covered | — |
| REQ-2 | [requirement text] | Feature N, M | Partial | [what's missing] |
| BEH-1 | [behavior description] | — | Missing | No feature addresses this |
| ENT-1 | [entity definition] | Feature M | Contradicted | [plan says X, spec says Y] |

Coverage classifications:
- **Covered** — feature fully addresses the requirement
- **Partial** — feature addresses some but not all aspects
- **Missing** — no feature addresses this requirement
- **Contradicted** — feature contradicts the requirement

---

## Step 4: Detect Ambiguities

For each spec area that is vague or under-specified:

1. **Identify** — what exactly is ambiguous (quote the spec text)
2. **Note assumption** — what the plan assumes (explicitly or implicitly)
3. **Assess risk** — is the assumption reasonable, risky, or likely wrong?
4. **Recommend action:**
   - Clarify with stakeholder before implementation
   - Add explicit assumption to plan and proceed
   - Flag as risk in the plan's Security Considerations

---

## Step 5: Detect Gold-Plating

For each plan feature, verify it traces back to at least one spec item.

Features that don't trace back are either:
- **Legitimate scaffolding** — test infrastructure, CI config, build setup, migration boilerplate → acceptable, note as `Infrastructure`
- **Scope creep** — functionality not requested by the spec → flag as Important issue

---

## Step 6: Verify Affected Files

For each feature's "Affected Files" list:

1. **Missing files** — does the spec imply changes to files not listed in the feature? Cross-reference entity definitions, API contracts, and behaviors against the codebase to find files the plan should touch but doesn't mention.
2. **Orphan files** — are files listed that aren't justified by the spec or feature behavior? These may indicate gold-plating or copy-paste errors.
3. **Cross-feature conflicts** — do multiple features list the same file? If so, verify the changes are compatible and note the overlap for the "Changes by File" output section.

---

## Output

Feed all findings into the main review output ([review-output.md](../assets/review-output.md)):

- **Traceability matrix** → "Spec Alignment Analysis" section
- **Missing/contradicted requirements** → "Issues Found" with severity:
  - Missing requirement → Critical if core functionality, Important otherwise
  - Contradicted requirement → Critical
  - Partial coverage → Important
- **Ambiguities** → "Ambiguities Detected" subsection
- **Gold-plating** → "Issues Found" as Important
- **File-level findings** → "Changes by File" section (missing files, orphan files, cross-feature conflicts)
