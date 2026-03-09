---
status: complete
priority: p3
issue_id: "009"
tags: [code-review, quality, openapi]
dependencies: []
---

# Audit YAGNI Fields in OpenAPI Schemas

## Problem Statement

Several fields in the Experiment and related schemas may be YAGNI (You Aren't Gonna Need It): `minimumDetectableEffect`, `practicalSignificanceThreshold`, `epsilon_greedy`, `fallbackTemplateUid`. These add complexity without confirmed use in the actual API.

## Findings

- **Source:** code-simplicity-reviewer agent
- **Location:** `openapi/v1.yaml` — Experiment schema

## Proposed Solutions

### Option A: Remove unimplemented fields
- **Pros:** Simpler spec, no false promises to API consumers
- **Cons:** Would need to re-add if implemented later
- **Effort:** Small
- **Risk:** Low

Verify against actual API implementation, remove fields that don't exist.

### Option B: Keep but mark as future/beta
- **Pros:** Documents planned features
- **Cons:** Confuses current API consumers
- **Effort:** Small
- **Risk:** Low

## Technical Details

- **Affected files:** `openapi/v1.yaml`

## Acceptance Criteria

- [ ] Each questioned field verified against actual API
- [ ] Unimplemented fields removed or clearly marked

## Work Log

| Date | Action | Notes |
|------|--------|-------|
| 2026-03-09 | Created | From code review findings |
