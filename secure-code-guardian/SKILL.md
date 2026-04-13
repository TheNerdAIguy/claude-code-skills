---
name: secure-code-guardian
description: Use when building anything with authentication, passwords, user input, or sensitive data — writes secure code from the start, preventing vulnerabilities before they happen
---

# Secure Code Guardian

## Overview

Security vulnerabilities introduced at coding time are exponentially harder to fix later. This skill enforces secure-by-default patterns for auth, input handling, and data exposure.

**Core principle:** Write secure code from the jump — not as an afterthought.

**Announce at start:** "I'm using the secure-code-guardian skill to write vulnerability-free code."

## When to Use

Activate for ANY code involving:
- Authentication / authorization (login, sessions, JWT, OAuth)
- Password handling (storage, reset flows, validation)
- User input (forms, API params, file uploads)
- Database queries with dynamic values
- File system access with user-controlled paths
- External API calls with sensitive data
- Payment or financial data

## The Security-First Checklist

Before writing a single line, verify:

### Authentication
- [ ] Passwords hashed with bcrypt/argon2 (never MD5/SHA1)
- [ ] Session tokens are cryptographically random (min 128 bits)
- [ ] JWT secrets are strong and rotated
- [ ] Login attempts are rate-limited
- [ ] Account lockout after N failed attempts
- [ ] Secure cookie flags: `HttpOnly`, `Secure`, `SameSite=Strict`

### Input Validation
- [ ] All user input validated at the boundary (never trust client)
- [ ] Schema validation before processing (Zod, Joi, Pydantic, etc.)
- [ ] File uploads restricted by type, size, and path
- [ ] No eval() or dynamic code execution with user data

### SQL / Database
- [ ] Parameterized queries ONLY — never string concatenation
- [ ] ORM used correctly (avoid raw queries unless parameterized)
- [ ] Principle of least privilege on DB users
- [ ] Sensitive fields encrypted at rest

### Output / API
- [ ] All output HTML-escaped to prevent XSS
- [ ] API responses never expose internal IDs, stack traces, or passwords
- [ ] Error messages user-friendly but not info-leaking
- [ ] CORS configured strictly (not `*` in production)

### Secrets
- [ ] No secrets in source code — use env vars
- [ ] No secrets in logs or error messages
- [ ] No secrets in URLs (use POST body or headers)

## Implementation Patterns

### Secure Password Hashing
```typescript
// CORRECT
import bcrypt from 'bcrypt'
const hash = await bcrypt.hash(password, 12)
const valid = await bcrypt.compare(input, hash)

// NEVER
const hash = md5(password) // ❌
const hash = sha1(password) // ❌
```

### Safe SQL (Parameterized)
```typescript
// CORRECT
const user = await db.query('SELECT * FROM users WHERE email = $1', [email])

// NEVER
const user = await db.query(`SELECT * FROM users WHERE email = '${email}'`) // ❌
```

### Input Validation
```typescript
// CORRECT
import { z } from 'zod'
const schema = z.object({ email: z.string().email(), age: z.number().min(0).max(150) })
const data = schema.parse(req.body) // throws if invalid

// NEVER
const { email, age } = req.body // ❌ raw, unvalidated
```

### Safe File Path
```typescript
// CORRECT
import path from 'path'
const safePath = path.join('/uploads', path.basename(userInput)) // strips traversal

// NEVER
const filePath = '/uploads/' + userInput // ❌ path traversal
```

## OWASP Top 10 Reference

| Risk | Guard |
|------|-------|
| A01 Broken Access Control | Check permissions on every route, never trust frontend |
| A02 Cryptographic Failures | bcrypt/argon2, TLS everywhere, encrypt at rest |
| A03 Injection | Parameterized queries, schema validation |
| A04 Insecure Design | Threat model before building auth flows |
| A05 Security Misconfiguration | Disable debug in prod, lock CORS, set security headers |
| A06 Vulnerable Components | Pin dependency versions, audit regularly |
| A07 Auth Failures | Rate limit, MFA, secure session management |
| A08 Data Integrity | Verify signatures, use SRI for CDN assets |
| A09 Logging Failures | Log security events, never log secrets |
| A10 SSRF | Whitelist URLs, block internal network access |

## Security Headers (Express/Node example)
```typescript
app.use(helmet()) // sets X-Frame-Options, CSP, HSTS, etc.
app.use(rateLimit({ windowMs: 15 * 60 * 1000, max: 100 }))
```

## After Writing — Verification Steps

1. Run `npm audit` / `pip-audit` / `cargo audit`
2. Check for hardcoded secrets: `grep -r "password\|secret\|key" --include="*.ts" src/`
3. Test with invalid/malicious inputs
4. Verify error messages don't leak internals
