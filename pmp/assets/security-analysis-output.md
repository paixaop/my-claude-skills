# Security Analysis — [Plan/Feature Name]

Date: [date] | Source: [plan document path]

## Executive Summary
[3-5 sentences: highest-risk threats, most exposed crown jewels, key architectural gaps]

## Extracted Architecture
### Entry Points — [table: endpoint, auth requirement, input sources]
### Trust Boundaries — [boundary, components on each side, data crossing]
### Crown Jewels — [asset, location, protection mechanism from plan]
### Data Flows — [entry → processing → storage/action, validation points]

## STRIDE Analysis
[One subsection per trust boundary]

## Attack Trees
[One tree per crown jewel]

## Business Logic Concerns
[Race conditions, pipeline manipulation, DoS/cost findings]

## Risk Matrix
[Sorted by risk score descending]

## Spec Change Recommendations
### Critical (must address before implementation)
### Important (address during implementation)
### Suggested (defense-in-depth)

## Positive Observations
[What the plan already handles well]

## Gaps in Specification
[Missing architecture elements preventing complete analysis]
