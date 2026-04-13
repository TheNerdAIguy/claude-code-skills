---
name: spec-miner
description: Use when inheriting code with zero documentation — reads the codebase, traces data flows, and writes the spec that should have been there from day one
---

# Spec Miner

## Overview

Inherited code without documentation is a liability. Before touching it, you need to understand what it actually does — not what it was supposed to do. This skill reverse-engineers the implicit specification from the code itself.

**Core principle:** The code is the truth. Comments lie, docs are missing, but running code doesn't.

**Announce at start:** "I'm using the spec-miner skill to reverse-engineer the documentation from the codebase."

## When to Use

Use when:
- Taking over a legacy codebase with no documentation
- Onboarding to an undocumented service or module
- Before refactoring code you don't fully understand
- When the team left and took the knowledge with them
- When `git blame` shows the original author no longer exists
- Before writing tests for untested code

## Phase 1 — Codebase Reconnaissance

### 1.1 Entry Points
Find where execution starts:
```bash
# Look for main entry points
grep -r "main\|app.listen\|server.start\|__name__" --include="*.py" .
grep -r "export default\|module.exports\|createServer" --include="*.js" .
# Check package.json scripts
cat package.json | jq '.scripts'
# Check Dockerfile or Procfile
cat Dockerfile Procfile 2>/dev/null
```

### 1.2 Dependency Map
```bash
# What does this depend on?
cat package.json requirements.txt Cargo.toml go.mod 2>/dev/null
# What talks to what?
grep -r "import\|require\|from" src/ | sort | uniq -c | sort -rn | head -30
```

### 1.3 External Connections
```bash
# Find all API calls, DB connections, external services
grep -rn "fetch\|axios\|http\.\|https\.\|pg\.\|mongoose\|redis\|s3\." src/
grep -rn "process.env\|os.environ\|getenv" src/  # Find env vars = external config
```

### 1.4 Data Models
```bash
# Find all data structures
grep -rn "interface\|type\|class\|schema\|model" src/ --include="*.ts"
find . -name "*.sql" -o -name "migrations" -o -name "schema.rb"
```

## Phase 2 — Data Flow Tracing

For each major function or module, trace:

**Input → Transformation → Output**

```
What enters?     →    What happens to it?    →    What comes out?
[HTTP request]   →    [validate + transform]  →    [DB write + response]
[File upload]    →    [parse + sanitize]      →    [stored + indexed]
[Cron trigger]   →    [fetch + aggregate]     →    [report email sent]
```

Document each flow:
```markdown
### Flow: User Registration
Input: POST /api/users { email, password, name }
Steps:
  1. Validate email format + uniqueness
  2. Hash password (bcrypt, rounds=10)
  3. Create user record in DB
  4. Send welcome email via Resend
  5. Create session token
Output: { userId, token, expiresAt }
Error cases: 409 (email taken), 400 (validation), 500 (DB/email failure)
```

## Phase 3 — Behavior Documentation

### API Endpoints
For each route, document:
```markdown
## POST /api/[path]
**Purpose:** [what does this do?]
**Auth required:** yes/no (which roles?)
**Request:**
  - Body: { field: type }
  - Headers: { Authorization: Bearer ... }
**Response:**
  - 200: { ... }
  - 400: { error: "..." }
  - 401: { error: "Unauthorized" }
**Side effects:** [DB writes, emails sent, events published, etc.]
**Rate limited:** yes/no
```

### Business Rules
Extract implicit rules from code:
```typescript
// Code says:
if (user.plan === 'free' && uploads.count >= 5) throw new Error('limit')

// Rule is:
// Free plan users are limited to 5 uploads total
```

Document each discovered rule:
```markdown
## Business Rules

### Upload Limits
- Free: max 5 uploads
- Pro: unlimited
- File size: max 50MB (validated in middleware)
- Allowed types: jpg, png, pdf (from ALLOWED_TYPES constant)

### Pricing
- Trial: 14 days (from TRIAL_DAYS env var, default 14)
- Downgrade: immediate (no proration)
- Upgrade: prorated to billing cycle end
```

## Phase 4 — Generate Spec Document

Produce this artifact:

```markdown
# [Service/Module Name] — Reverse-Engineered Specification
**Mined from:** [repo/directory]
**Date:** [date]
**Confidence:** High / Medium / Low (based on code clarity)

## Purpose
[1-2 sentences: what does this service/module do?]

## Architecture Overview
[Diagram or description of components]

## Entry Points
| Entry | Description |
|-------|-------------|
| `POST /api/...` | [purpose] |
| `GET /api/...` | [purpose] |
| `cron: daily at 9am` | [purpose] |

## Data Models
### [ModelName]
| Field | Type | Notes |
|-------|------|-------|
| id | UUID | auto-generated |
| ... | ... | ... |

## Data Flows
[One section per major flow — see Phase 2 format]

## Business Rules
[Extracted implicit rules — see Phase 3 format]

## External Dependencies
| Service | Purpose | Failure behavior |
|---------|---------|-----------------|
| Supabase | User data | Returns 503 |
| Resend | Emails | Silently fails (logs error) |
| Stripe | Payments | Throws, caught by error handler |

## Environment Variables
| Variable | Purpose | Default |
|----------|---------|---------|
| DATABASE_URL | DB connection | Required |
| ... | ... | ... |

## Unknown / Unclear Areas
- [ ] [Thing that's unclear — needs investigation or original author]
- [ ] [Behavior that seems wrong or undefined]

## Suggested Tests to Write
1. [Test that would verify the most critical behavior]
2. [Test that covers the most dangerous edge case]
```

## Activation Phrases

Claude activates this skill when you say:
- "reverse engineer this" / "document this codebase" / "understand existing system"
- "what does this code do?" (at module/service level)
- "I inherited this code and there's no docs"
- "write specs for this existing code"
- "help me understand this before I change it"
