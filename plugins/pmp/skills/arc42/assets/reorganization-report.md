# Reorganization Proposal: [Directory/Project Name]
<!-- Template for arc42 reorganization proposal — used by references/arc42.md -->

**Date:** YYYY-MM-DD
**Source directory:** [path]
**Files analyzed:** N
**Total word count:** W

---

## Current File Inventory

| # | File | Word Count | Arc42 Section | Concerns |
|---|------|------------|---------------|----------|
| 1 | [path] | [count] | [section name or multiple] | [concern names] |

---

## Section Assignment Table

| Source File | Source Section | Arc42 Section | Concern | Word Count |
|-------------|---------------|---------------|---------|------------|
| [file] | [heading] | [1-12 or 99] | [concern name] | [count] |

---

## Proposed Arc42 Structure

```
<target-dir>/
├── README.md
├── NN-section-name/
│   ├── README.md
│   ├── concern-a.md
│   └── concern-b.md
└── ...
```

| # | Section Directory | Concern Files | Source Files | Estimated Words |
|---|-------------------|---------------|-------------|-----------------|
| 1 | `01-introduction-and-goals/` | [file list] | [source-a.md, source-b.md] | [count] |

---

## Content Movement Map

### [Source File 1]

| Section | Target Directory | Target File | Action | Notes |
|---------|-----------------|-------------|--------|-------|
| [heading] | [NN-section/] | [concern.md] | move / merge / keep | [if merge: merged with section X from file Y] |

### [Source File 2]

| Section | Target Directory | Target File | Action | Notes |
|---------|-----------------|-------------|--------|-------|
| [heading] | [NN-section/] | [concern.md] | move / merge / keep | |

---

## Duplicates to Merge

| # | Content Summary | Location A | Location B | Keep From | Action |
|---|-----------------|------------|------------|-----------|--------|
| 1 | [summary] | [file:section] | [file:section] | [file] | merge / deduplicate |

---

## Contradictions Requiring Resolution

| # | Description | Statement A | Source A | Statement B | Source B | Severity |
|---|-------------|-------------|---------|-------------|---------|----------|
| 1 | [description] | [text] | [file:section] | [text] | [file:section] | Critical / Important / Minor |

**Action required:** For each contradiction, please indicate which version to keep, or provide a reconciled statement.

---

## Files to Delete (After Reorganization)

| File | Reason |
|------|--------|
| [path] | Fully consumed into [target directories] |

---

## Files Unchanged

| File | Classification | Reason |
|------|---------------|--------|
| [path] | reference / schema / config / image | Not a reorganization target |

---

## Review Annotations Inventory

| Annotation Type | Count | Source Files |
|-----------------|-------|-------------|
| `[!CAUTION]` | N | [files] |
| `[!WARNING]` | N | [files] |
| `[!NOTE]` | N | [files] |

All annotations will be preserved and moved with their associated content.
