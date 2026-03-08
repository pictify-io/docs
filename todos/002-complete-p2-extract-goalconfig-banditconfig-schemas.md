---
status: complete
priority: p2
issue_id: "002"
tags: [code-review, openapi, architecture]
dependencies: []
---

# Extract GoalConfig/BanditConfig into Named Schemas

## Problem Statement

`goalConfig` and `banditConfig` are inlined in the Experiment schema rather than extracted as reusable named schemas. This reduces reusability and makes the spec harder to maintain.

## Findings

- **Source:** architecture-strategist agent
- **Location:** `openapi/v1.yaml` — Experiment schema, inline `goalConfig` and `banditConfig` objects
- These are complex objects that deserve their own schema definitions

## Proposed Solutions

### Option A: Extract to named schemas
- **Pros:** Cleaner spec, reusable, easier maintenance
- **Cons:** Minor refactor
- **Effort:** Small
- **Risk:** Low

Create `GoalConfig` and `BanditConfig` as separate schemas under `components/schemas`, then `$ref` them from the Experiment schema.

## Recommended Action

_(To be filled during triage)_

## Technical Details

- **Affected files:** `openapi/v1.yaml`

## Acceptance Criteria

- [ ] `GoalConfig` extracted as named schema
- [ ] `BanditConfig` extracted as named schema
- [ ] Experiment schema uses `$ref` to reference them
- [ ] Spec parses without errors

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |

## Resources

- Review agent: architecture-strategist
