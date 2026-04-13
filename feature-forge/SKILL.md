---
name: feature-forge
description: Use before writing a single line of code for any new feature — asks the right questions, thinks like both PM and dev, produces a full spec with requirements and acceptance criteria
---

# Feature Forge

## Overview

Writing code without a clear spec wastes time and produces the wrong thing. This skill transforms a vague feature idea into a precise, buildable specification before any implementation begins.

**Core principle:** No code until the spec is clear. Requirements discovered during coding cost 10x more than requirements discovered before.

**Announce at start:** "I'm using the feature-forge skill to define the spec before we code."

## When to Use

Use before implementing ANY feature, especially when:
- The requirement is vague or ambiguous
- Multiple implementation approaches exist
- The feature touches multiple systems
- You're not 100% sure what "done" looks like
- The user said "just build X" without details

## Phase 1 — Discovery Questions

Ask these before writing a single line:

### Functional Scope
1. **What is the user trying to accomplish?** (job-to-be-done, not feature description)
2. **Who is the user?** (role, technical level, context)
3. **What triggers this feature?** (event, user action, schedule?)
4. **What does success look like?** (measurable outcome)
5. **What are the edge cases?** (empty state, errors, limits, concurrency)

### Technical Boundaries
6. **What does this connect to?** (APIs, DB tables, external services)
7. **What data goes in? What comes out?** (input/output contract)
8. **What must NOT change?** (existing behavior to preserve)
9. **Are there performance requirements?** (latency, throughput, scale)
10. **What are the security requirements?** (who can access this?)

### Definition of Done
11. **What tests prove it works?** (happy path + edge cases)
12. **What's explicitly OUT of scope?** (prevents scope creep)
13. **What's the rollout strategy?** (feature flag, gradual, immediate?)

## Phase 2 — Spec Document

Produce this document before any code:

```markdown
# Feature: [Name]
**Author:** [name] | **Date:** [date] | **Status:** Draft

## Problem Statement
[1-2 sentences: what user pain does this solve?]

## User Story
As a [role], I want to [action] so that [outcome].

## Acceptance Criteria
- [ ] Given [context], when [action], then [result]
- [ ] Given [context], when [action], then [result]
- [ ] Error case: Given [invalid input], when [action], then [error message shown]

## Technical Design

### Data Model
[New fields, tables, or schema changes]

### API Contract
- Endpoint: `POST /api/[resource]`
- Request: `{ field: type, ... }`
- Response: `{ field: type, ... }`
- Errors: `400 | 401 | 404 | 500`

### Dependencies
- Reads from: [table/service]
- Writes to: [table/service]
- Calls: [external API]

## Out of Scope
- [Thing that sounds related but isn't in this feature]
- [Future enhancement for later]

## Test Plan
1. Unit: [what to unit test]
2. Integration: [what API/DB flows to test]
3. E2E: [critical user journey to automate]

## Rollout
- [ ] Feature flag: `feature_[name]`
- [ ] Migration needed: yes/no
- [ ] Backward compatible: yes/no
```

## Phase 3 — Pre-Code Verification

Before starting implementation, confirm:
- [ ] Every acceptance criterion is testable
- [ ] No ambiguous "the system should..." requirements
- [ ] Edge cases and error states are specified
- [ ] Data model changes are documented
- [ ] Stakeholder has reviewed and approved

## Common Spec Smells (Fix These)

| Smell | Fix |
|-------|-----|
| "The system should be fast" | "P95 response < 200ms under 1000 req/s" |
| "Handle errors gracefully" | "Show error toast: 'Upload failed. Try again.'" |
| "User can manage X" | "User can create/read/update/delete X with these constraints..." |
| "Integrate with Y" | "Call Y's /endpoint, map fields A→B, handle 401/500" |
| "It should work like Stripe" | Link to exact Stripe behavior you're replicating |

## Activation Phrases

Claude will activate this skill when you say:
- "define feature" / "write spec for..." / "requirements for..."
- "I want to build X" (before any code discussion)
- "what do we need to implement Y?"
