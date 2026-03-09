---
status: complete
priority: p2
issue_id: "001"
tags: [code-review, openapi, quality]
dependencies: []
---

# Missing `required` Fields on Experiment Schema

## Problem Statement

The `Experiment` schema in `openapi/v1.yaml` does not specify `required` fields. Without `required`, API consumers cannot determine which fields are guaranteed in responses or mandatory in requests.

## Findings

- **Source:** architecture-strategist agent
- **Location:** `openapi/v1.yaml` — `Experiment` schema
- The schema defines many properties but has no `required` array
- This affects both request validation and response contract clarity

## Proposed Solutions

### Option A: Add required array to Experiment schema
- **Pros:** Immediate fix, follows OpenAPI best practice
- **Cons:** None
- **Effort:** Small
- **Risk:** Low

Add `required: [uid, name, type, status, variants, createdAt]` to the Experiment schema.

## Recommended Action

_(To be filled during triage)_

## Technical Details

- **Affected files:** `openapi/v1.yaml`
- **Components:** Experiment schema definition

## Acceptance Criteria

- [ ] `Experiment` schema has `required` array with core fields
- [ ] Other new schemas reviewed for missing `required` fields
- [ ] OpenAPI spec still parses without errors

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |

## Resources

- Review agent: architecture-strategist
