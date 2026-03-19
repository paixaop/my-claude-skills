# Single Source of Truth — Spec File Rules

Architecture specifications are organized as **one file per concept**. Every distinct architectural element has exactly one canonical Markdown file. No monolithic documents. No duplicated definitions.

These rules apply to any PMP skill that creates, reorganizes, or reviews spec files.

### Core Principle

Each named concept — component, subsystem, pattern, decision, runtime flow — lives in **exactly one file** that is its single source of truth. All other references point to that file. If information exists in two places, one must be deleted and replaced with a link.

### What Counts as One File

| Category | Examples |
|----------|----------|
| Component / subsystem / module | UserService, PaymentGateway, AuthMiddleware, MessageBroker |
| External interface or adapter | REST API adapter, gRPC adapter, third-party SDK wrapper |
| Runtime flow or scenario | Request lifecycle, checkout flow, event processing pipeline |
| Crosscutting concern or pattern | Error handling, logging, caching, retry strategy, authentication |
| Architecture decision (ADR) | Database selection, message broker choice, API versioning strategy |
| Quality requirement or test strategy | Performance targets, test strategy, SLA definitions |
| Configuration or settings | Config schema, environment variables, feature flags |
| Glossary or domain terms | Ubiquitous language, domain-specific terminology |
| Risk or technical debt item | Known risks, accepted trade-offs, deferred items |
| Data model or schema | Database schema, event schema, API contract |

If two concepts are tightly coupled (e.g. authentication and authorization), they still get separate files. Merge only when the split creates more confusion than value — and ask first.

### Folder Structure

The project's architecture docs live in a root directory (e.g. `docs/architecture/`, `specs/`, or whatever the project uses) with a `README.md` at the root as the entry point. The directory is organized into **section folders** that group related concepts.

**Discover the project's existing folder structure first. Adapt to it.** Do not impose a new structure on a project that already has one. The principles in this document apply regardless of how sections are named or organized.

When no structure exists, organize into logical section folders that fit the project's domain. Name folders descriptively — by topic, not by number.

**Example (flat, domain-driven):**
```
docs/architecture/
├── README.md
├── components/
├── flows/
├── crosscutting/
├── decisions/
└── glossary/
```

**Example (arc42-style — use when the project already follows arc42 or when `/pmp:arc42` is invoked):**
```
docs/architecture/
├── README.md
├── 01-introduction-and-goals/
├── 05-building-block-view/
├── 06-runtime-view/
├── 08-crosscutting-concepts/
├── 09-architecture-decisions/
└── 12-glossary/
```

Each section folder contains:
- A `README.md` index listing all files in that section with a brief description
- One `.md` file per concept (kebab-case, descriptive)

### Progressive Disclosure

Spec files are structured so readers can stop at the level of detail they need:

| Level | What | Where | Answers |
|-------|------|-------|---------|
| 0 | System overview + maintenance rules | Root `README.md` | What is this system? What are the major areas? How do I maintain these specs? |
| 1 | Section index | Each section's `README.md` | What concepts exist in this area? |
| 2 | File summary block | Top of each spec file (before `---`) | What does this file define? What are its key sections? What does it depend on? |
| 3 | Full section content | Below the `---` in each spec file | Complete definitions, interfaces, invariants, examples |

An agent or human exploring the architecture reads Level 0 — which includes both the navigation table and the maintenance rules — before touching any file. It picks a section, reads Level 1 (which links back to the rules), picks a file, reads Level 2, and only then drills into the specific Level 3 sections it needs via fragment anchors. The rules are in context at every level.

### File Format

Every spec file follows this structure:

```markdown
<!-- Source: [original source files or "new"] on [date] -->

# [Concept Name]

[2-4 sentence summary: what this file defines and why it matters.]

> **Depends on:**
> - [Concept § Section](relative-path.md#section) — [why]
> - [Concept § Section](relative-path.md#section) — [why]
>
> **Key sections:**
> - [Section Name](#section-anchor) — [one-line description]
> - [Section Name](#section-anchor) — [one-line description]
> - [Section Name](#section-anchor) — [one-line description]

---

## [First major section]
...

### [Subsection]
...

## [Second major section]
...
```

Header rules:
- HTML comment on line 1 tracks provenance (which source files were merged, or "new")
- Level-1 heading = concept name, exactly one per file
- Summary paragraph immediately after the H1 — enough to decide whether to read further
- `Depends on` block lists the specific sections in other files this file relies on, with a short reason
- `Key sections` block lists this file's own sections with anchors and one-line descriptions — a scannable table of contents
- Horizontal rule separates the summary block from the full content

Heading rules:
- **Never number headings.** These are living documents — sections get added, removed, and reordered. Numbered headings (`## 1. Overview`, `### 2.1 Components`) create renumbering churn and break anchors. Use descriptive names only.
- Headings are link targets. Other files point to them via fragment anchors (`#error-classification`, `#retry-strategy`).
- **Never rename a heading** without searching for all inbound links and updating them.
- Choose heading names that are descriptive and stable — names that describe the concept, not its position.

### File Size Limit

Target **≤ 500 lines** per file. If a file exceeds this:
1. Identify natural sub-concerns within it
2. Split into multiple files named `{concept}-{subconcern}.md`
3. Update the section README.md index

Example: `security.md` (1200 lines) → `security-auth.md`, `security-detection.md`, `security-crypto.md`, `security-validation.md`

### Cross-References

References are **markdown links to the exact section**, not prose. A reference is a link — not a sentence that happens to contain a link.

Fragment anchors follow GitHub-flavored Markdown: lowercase, spaces → hyphens, strip punctuation. `## Error Classification` → `#error-classification`. `### Retry Strategy` → `#retry-strategy`.

**Link to the whole file** only when referencing the concept in general. **Link to the specific section** whenever referencing a definition, rule, interface, invariant, or behavior within that file.

---

#### Good: bare link, points to exact section

```markdown
Errors are classified into three tiers — [Error Handling § Error Classification](error-handling.md#error-classification).
```

#### Good: link list at top of file for primary dependencies

```markdown
> **Depends on:**
> - [Auth Service § Token Validation](auth-service.md#token-validation)
> - [Error Handling § Retry Strategy](../crosscutting/error-handling.md#retry-strategy)
> - [Config § Feature Flags](../crosscutting/config.md#feature-flags)
```

#### Good: inline link replacing what would otherwise be a duplicated definition

```markdown
Each request passes through the [authentication middleware](auth-service.md#middleware)
before reaching the handler.
```

#### Good: general concept reference links to the whole file

```markdown
The caching strategy is defined in [caching.md](caching.md).
```

---

#### Bad: prose wrapper around the link

```markdown
See [Error Classification](error-handling.md#error-classification) for how errors are categorized.
```
The word "See" adds nothing. Write the statement that needs the reference, then attach the link.

#### Bad: link to file when a specific section is the real target

```markdown
Errors are classified into three tiers — [error-handling.md](error-handling.md).
```
This forces the reader to scan the entire file. Link to `#error-classification` directly.

#### Bad: link text is a file path

```markdown
Defined in [../crosscutting/error-handling.md](../crosscutting/error-handling.md#error-classification).
```
Link text should name the concept, not the path.

---

**Link text rules:**
- Link text names the concept, not the file path
- For section links, use `[Concept § Section](file.md#anchor)` or `[Section Name](file.md#anchor)`
- For inline references where the concept is already named in the sentence, the link wraps the concept name directly

### Naming Conventions

| Pattern | Example |
|---------|---------|
| Component / subsystem | `auth-service.md`, `payment-gateway.md`, `message-broker.md` |
| Crosscutting concern | `error-handling.md`, `logging.md`, `caching.md` |
| Concern shard (split file) | `security-auth.md`, `security-crypto.md`, `observability-metrics.md` |
| Architecture decision | `adr-NNN-short-description.md` |
| Runtime scenario | `request-lifecycle.md`, `checkout-flow.md`, `event-processing.md` |
| Data model / schema | `user-schema.md`, `event-catalog.md` |

All filenames: kebab-case, lowercase, descriptive, `.md` extension.

### Eliminating Duplication

When you find the same concept defined in multiple places:

1. **Choose the canonical file and section** — the one in the correct section folder with the most complete/current content
2. **Merge** the best information from all occurrences into that file's appropriate section
3. **Delete** the duplicate content from other files entirely
4. **Insert a section-specific pointer** where the duplicate lived. Point to the exact section, not just the file:

Single canonical section:
```markdown
> **Canonical definition** → [Auth Service § Token Validation](../building-blocks/auth-service.md#token-validation)
```

Multiple canonical sections (when the duplicate spanned several topics):
```markdown
> **Canonical definitions:**
> - [Error Handling § Error Classification](../crosscutting/error-handling.md#error-classification)
> - [Auth Service § Token Validation](../building-blocks/auth-service.md#token-validation)
> - [Config § Retry Settings](../crosscutting/config.md#retry-settings)
```

5. **Resolve contradictions** — keep the chosen version; add a note only if the conflict is architecturally significant:

```markdown
> Previously conflicting definitions existed in [source]. Canonical version: [Error Handling § Retry Strategy](../crosscutting/error-handling.md#retry-strategy).
```

### Updating Section Indexes

After creating, renaming, splitting, or moving files, update the relevant `README.md`:
- The section's own `README.md` (add/remove table rows)
- The root architecture `README.md` if section descriptions changed

Index table format:
```markdown
| File | Concern |
|------|---------|
| [auth-service.md](auth-service.md) | Authentication, authorization, token management |
```

### ADR Format

Architecture Decision Records follow:

```markdown
<!-- Source: [context] on [date] -->

# ADR-NNN: [Decision Title]

## Status
Accepted | Proposed | Superseded by ADR-NNN

## Context
[Problem and constraints]

## Decision
[What was decided and why]

## Consequences
[Trade-offs accepted]
```

### Self-Policing: Embed Rules in the Architecture README

The architecture must carry its own maintenance rules at the entry point — the root `README.md` of the architecture directory. After the navigation table, add or maintain a `## Maintenance Rules` section containing the rules summary, file format template, and the creating/reviewing checklists below. Link back to this file for the full reference with examples.

Each section-level `README.md` should include a one-line pointer at the top:

```markdown
> **Before editing any file in this section** — read [Maintenance Rules](../README.md#maintenance-rules).
```

### Rules Summary

1. **One concept = one file.** No exceptions without explicit justification.
2. **No duplication.** If it exists in two places, one dies and becomes a section-specific link.
3. **Summary block on every file.** Summary paragraph, `Depends on` links, `Key sections` list — before the `---`.
4. **Section-level links.** Cross-references point to the exact heading (`file.md#section`), not just the file.
5. **Links, not prose.** References are markdown links — not "see X for Y" sentences.
6. **No numbered headings.** Descriptive names only (`## Verdict`, not `## 3. Verdict`).
7. **Stable anchors.** Never rename a heading without updating all inbound links.
8. **≤ 500 lines.** Split by sub-concern when exceeded.
9. **Provenance comment.** Every file tracks where its content came from.
10. **Indexes stay current.** Section README.md reflects actual files.
11. **Relative links only.** No absolute paths, no duplicated content.
12. **kebab-case filenames.** Descriptive, unique, lowercase.

### When Creating New Specs

1. Determine the correct section folder
2. Check if a file for this concept already exists (search by name and content)
3. If yes → add to the existing file (or split it if it would exceed 500 lines)
4. If no → create `{section}/{concept-name}.md` using the file format above
5. Write the summary block first: summary paragraph, `Depends on` links, `Key sections` list
6. Use descriptive, unnumbered headings so anchors stay stable across reorders
7. Update the section's `README.md` index
8. Add cross-reference links from related specs — markdown links to the specific section, not prose references

### When Reviewing Existing Specs

Look for and fix:
- Missing or incomplete summary blocks → add summary paragraph, `Depends on` links, `Key sections` list
- Concepts defined in multiple files → consolidate to the canonical section, replace duplicates with section-level pointers
- Files exceeding 500 lines → split
- Missing index entries → add them
- Broken cross-reference links → repair (check both file path and fragment anchor)
- File-level links that should be section-level → add the `#anchor` to point to the exact section
- `Key sections` list out of sync with actual headings → update anchors and descriptions
- `Depends on` links pointing to renamed/moved sections → update
- Contradictions between files → resolve, document which version wins with a section-specific pointer to the canonical definition
- Orphan files not listed in any index → add to index or delete if obsolete
- Renamed headings with stale inbound anchors → update all referencing files
