---
status: complete
priority: p2
issue_id: "004"
tags: [code-review, openapi, quality]
dependencies: []
---

# Complete ConditionTree Schema - Add Rule Operator Enum

## Problem Statement

The `ConditionTree` schema's rule `operator` field lacks an enum constraint. Without it, API consumers don't know which operators are valid.

## Findings

- **Source:** architecture-strategist agent
- **Location:** `openapi/v1.yaml` — `ConditionTree` schema, rule operator property
- The operator field is `type: string` with no enum values listed
- Operators like `equals`, `contains`, `matches`, etc. should be enumerated

## Proposed Solutions

### Option A: Add enum to operator field
- **Pros:** Clear API contract, enables client-side validation
- **Cons:** None
- **Effort:** Small
- **Risk:** Low

Add `enum: [equals, not_equals, contains, not_contains, starts_with, ends_with, matches, gt, lt, gte, lte]` (or whatever operators the actual API supports).

## Recommended Action

_(To be filled during triage)_

## Technical Details

- **Affected files:** `openapi/v1.yaml`

## Acceptance Criteria

- [ ] `ConditionTree` rule operator has enum values
- [ ] Enum values match actual API implementation
- [ ] Spec parses without errors

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |

## Resources

- Review agent: architecture-strategist
