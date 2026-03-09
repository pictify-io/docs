---
status: complete
priority: p3
issue_id: "008"
tags: [code-review, quality, documentation]
dependencies: []
---

# Fix Webhook Pause/Resume Inconsistency

## Problem Statement

`api-reference/webhooks.mdx` correctly marks pause/resume as dashboard-only with `<Note>` blocks, but `concepts/webhooks.mdx` may still show curl commands for pause/resume, creating inconsistency.

## Findings

- **Source:** architecture-strategist agent
- **Location:** `concepts/webhooks.mdx` vs `api-reference/webhooks.mdx`

## Proposed Solutions

### Option A: Update concepts page to match
- **Pros:** Consistent documentation
- **Cons:** None
- **Effort:** Small
- **Risk:** Low

Add the same dashboard-only note to concepts/webhooks.mdx if it documents pause/resume API calls.

## Technical Details

- **Affected files:** `concepts/webhooks.mdx`

## Acceptance Criteria

- [ ] Both pages consistently describe pause/resume as dashboard-only
- [ ] No curl commands shown for pause/resume operations

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |
