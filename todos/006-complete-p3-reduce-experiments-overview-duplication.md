---
status: complete
priority: p3
issue_id: "006"
tags: [code-review, quality, documentation]
dependencies: []
---

# Reduce Duplication in api-reference/experiments.mdx

## Problem Statement

`api-reference/experiments.mdx` contains ~354 lines with extensive CodeGroup examples that duplicate what the individual endpoint stub pages auto-generate from OpenAPI. ~310 lines could be cut.

## Findings

- **Source:** code-simplicity-reviewer agent
- **Location:** `api-reference/experiments.mdx`
- Individual endpoint pages auto-render request/response examples from OpenAPI
- Overview page repeats the same information manually

## Proposed Solutions

### Option A: Trim to overview-only content
- **Pros:** Eliminates duplication, easier maintenance
- **Cons:** Less self-contained overview page
- **Effort:** Small
- **Risk:** Low

Keep introduction, status table, slug requirements, and links to endpoints. Remove CodeGroup examples for individual operations.

## Technical Details

- **Affected files:** `api-reference/experiments.mdx`

## Acceptance Criteria

- [ ] Overview page focuses on concepts and navigation
- [ ] Detailed examples deferred to endpoint pages
- [ ] No broken links or references

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |
