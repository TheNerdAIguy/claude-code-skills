---
name: the-fool
description: Use before committing to any big decision, architecture choice, or plan — challenges your thinking from 5 different angles including devil's advocate, red team, pre-mortem, and more
---

# The Fool

## Overview

The biggest risk in decision-making is confirmation bias — you propose an approach and only look for reasons it will work. The Fool forces you to actively seek reasons it will fail, be attacked, or be wrong.

**Core principle:** A plan that can't survive 5 adversarial challenges isn't ready to execute.

**Named after:** The Tarot's Fool — the character who asks the questions others won't. Not foolish, but fearless in questioning the obvious.

**Announce at start:** "I'm using the-fool skill to stress-test this decision from 5 angles."

## When to Use

Use before:
- Committing to an architecture decision
- Choosing a technology stack
- Designing a major feature
- Making a product pivot
- Hiring/partnership decisions
- Publishing a plan to stakeholders
- Any "we should definitely do X" moment

**Use especially when:**
- Everyone in the room agrees (groupthink warning)
- There's time pressure to decide
- You've already invested in this direction (sunk cost bias)
- The decision is hard to reverse

## The 5 Challenges

### 1. Devil's Advocate
*"What's the strongest case AGAINST this?"*

Find the best argument that this plan is wrong, even if you don't believe it. Steelman the opposition.

Questions to ask:
- What would a smart person who disagrees say?
- What assumptions are we making that could be false?
- Where is our reasoning weakest?
- What would need to be true for this to be the wrong decision?

---

### 2. Red Team
*"How would an adversary attack this?"*

Think like someone who wants this to fail — a competitor, a hacker, a regulator, or a user who hates it.

Questions to ask:
- How would a competitor exploit this if we shipped it?
- Where are the security vulnerabilities?
- What could a malicious user do with this?
- How could this be abused in ways we haven't considered?
- What would a journalist write if this went wrong?

---

### 3. Pre-Mortem
*"It's 6 months from now and this failed. Why?"*

Assume failure happened. Work backwards to identify the most likely causes. This bypasses optimism bias.

Questions to ask:
- What's the most likely reason this failed?
- What did we not anticipate that killed this?
- Which dependency or assumption turned out to be wrong?
- What did the team fight about?
- What did we run out of? (time, money, expertise, buy-in)

---

### 4. Edge Case Interrogator
*"What breaks this at the boundaries?"*

Test the decision against extreme inputs, unusual conditions, and second-order effects.

Questions to ask:
- What happens at 0? At 1? At 1 million?
- What happens when the network fails? The DB goes down? The API is slow?
- What about the user with unusual data (empty name, emoji in email, 10GB file)?
- What's the cascade if this component fails?
- What are the second-order effects we haven't modeled?

---

### 5. Alternatives Auditor
*"Are we solving the right problem? Is there a better way?"*

Challenge whether this is the right solution — or even the right problem to solve.

Questions to ask:
- What if we did nothing? What's the cost of inaction?
- What's the simplest possible solution? Why aren't we doing that?
- What would we do if this option didn't exist?
- Are there off-the-shelf solutions we're reinventing?
- Is there a completely different framing of this problem?

---

## Output Format

After running all 5 challenges, produce:

```markdown
## Decision Under Review
[The decision or plan being stress-tested]

## Challenge Results

### 1. Devil's Advocate
**Strongest counter-argument:** [...]
**Assumption at risk:** [...]
**Risk level:** Low / Medium / High

### 2. Red Team
**Main attack vector:** [...]
**Mitigation needed:** [...]
**Risk level:** Low / Medium / High

### 3. Pre-Mortem
**Most likely failure mode:** [...]
**Early warning sign:** [...]
**Risk level:** Low / Medium / High

### 4. Edge Cases
**Critical edge case:** [...]
**Handling plan:** [...]
**Risk level:** Low / Medium / High

### 5. Alternatives
**Better alternative considered:** [...]
**Why we're not doing it:** [...]

## Verdict
- Total blockers found: [N]
- Decision status: ✅ PROCEED | ⚠️ PROCEED WITH MITIGATIONS | 🛑 RECONSIDER

## Mitigations Required Before Proceeding
1. [...]
2. [...]
```

## Decision Thresholds

| Outcome | Action |
|---------|--------|
| 0 blockers | Proceed with confidence |
| 1-2 medium risks | Proceed with documented mitigations |
| Any high risk | Address before proceeding |
| Fundamental flaw found | Reconsider the decision entirely |

## Activation Phrases

Claude activates this skill when you say:
- "challenge this" / "poke holes in my plan" / "stress test this"
- "am I missing something?" / "what could go wrong?"
- "red team this" / "devil's advocate"
- Before major architecture or product decisions
