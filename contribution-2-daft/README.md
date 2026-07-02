# Contribution 2: math expressions parity with pyspark — rint and bround
 
**Contribution Number:** 2  
**Student:** Christopher Miranda Vega  
**Issue:** https://github.com/Eventual-Inc/Daft/issues/3793  
**Status:** Phase I — Complete
 
---
 
## Why I Chose This Issue
 
After my first contribution to Saleor, I wanted something where I'd actually be writing new logic rather than fixing a validation path. This issue is a tracked list of math functions missing from Daft's Python API, with PySpark as the reference implementation. Most of the list was already claimed or done, but `rint` and `bround` were still open — both are floating-point rounding functions with well-defined behavior, which means there's no ambiguity about what "correct" looks like.
 
`rint` and `bround` are also a good next step for me because they require touching both Rust (the core computation layer) and Python (the API wrapper), which is more depth than my Saleor contribution. I've been wanting to get exposure to Rust in a real codebase, and Daft's numeric module is approachable because every function follows the same pattern. I can read `round.rs` and know exactly how to structure `rint.rs`.
 
## Understanding the Issue
 
### Problem Description
 
Daft is missing `rint` and `bround` from its numeric function library. `rint` rounds a float to the nearest integer but returns a float, using banker's rounding (round half to even) on ties. `bround` is the same idea but accepts an optional decimal places argument. Both exist in PySpark's math functions API and calling either in Daft currently raises an error because they're not registered. The fix means implementing both in Rust, wiring them into the function registry, and exposing them through the Python API.
 
### Expected Behavior
 
`rint(2.5)` returns `2.0`, `rint(3.5)` returns `4.0`, and `bround(2.55, 1)` returns `2.6` — all using round-half-to-even semantics instead of the default round-half-away-from-zero that most languages use for their standard `round()`.
 
### Current Behavior
 
Neither `rint` nor `bround` exist in Daft. Calling `daft.functions.rint(expr)` raises an `AttributeError` because the function isn't registered anywhere in the codebase.
 
### Affected Components
 
- `src/daft-functions/src/numeric/rint.rs` — new file implementing the Rust computation
- `src/daft-functions/src/numeric/bround.rs` — new file implementing the Rust computation
- `src/daft-functions/src/numeric/mod.rs` — registers both functions in the function registry
- `daft/functions/numeric.py` — adds Python wrapper functions
- `daft/functions/__init__.py` — exports both functions publicly
---

## Reproduction Process

### Environment Setup

[Notes on setting up your local development environment - challenges you faced, how you solved them]

### Steps to Reproduce

1. [Step 1]
2. [Step 2]
3. [Observed result]

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

[Your analysis of the root cause - what's causing the issue?]

### Proposed Solution

[High-level description of your fix approach]

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** [Restate the problem]

**Match:** [What similar patterns/solutions exist in the codebase?]

**Plan:** [Step-by-step implementation plan]
1. [Modify file X to do Y]
2. [Add function Z]
3. [Update tests]

**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [X] Progress

[What you built this week, challenges faced, decisions made]

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]
