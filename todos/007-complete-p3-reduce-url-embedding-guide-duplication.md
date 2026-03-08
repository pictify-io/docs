---
status: complete
priority: p3
issue_id: "007"
tags: [code-review, quality, documentation]
dependencies: []
---

# Reduce Duplication in guides/url-embedding.mdx

## Problem Statement

`guides/url-embedding.mdx` overlaps significantly (~70 lines) with `concepts/url-rendering.mdx`. Both explain the same three URL patterns.

## Findings

- **Source:** code-simplicity-reviewer agent
- **Location:** `guides/url-embedding.mdx` vs `concepts/url-rendering.mdx`

## Proposed Solutions

### Option A: Make guide more practical, less conceptual
- **Pros:** Clear separation of concerns
- **Cons:** Minor rewrite
- **Effort:** Small
- **Risk:** Low

Keep the guide focused on practical embedding examples and use cases. Link to the concept page for how URL rendering works.

## Technical Details

- **Affected files:** `guides/url-embedding.mdx`

## Acceptance Criteria

- [ ] Guide focuses on practical use cases and copy-paste examples
- [ ] Conceptual explanations link to concepts/url-rendering.mdx
- [ ] No broken references

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |
