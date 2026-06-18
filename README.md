# github-contribution-log

# Contribution 1: Bug: externalUrl field validation rejects valid relative URLs
 
**Contribution Number:** 1  
**Student:** Christopher Miranda Vega  
**Issue:** https://github.com/saleor/saleor/issues/18012  
**Status:** Phase I — Completed, Phase II - IP
 
---
 
## Why I Chose This Issue
 
This issue caught my attention because it is a well-diagnosed, clearly scoped bug in a real-world Python/Django e-commerce backend. The root cause has already been identified by other contributors in the comments — Django's `URLValidator` rejects relative URLs by default, even though relative URLs starting with `/` are valid according to RFC 3986 — which means I can focus on understanding the fix rather than spending time on discovery. As someone making my first open source contribution, I wanted an issue where the problem and the affected code were clearly defined, so I could practice the contribution workflow without getting overwhelmed.
 
Beyond the scope, I chose this issue because it involves backend validation logic in Python, which aligns with my current skill set. I hope to learn how Django validators work under the hood, how a production-grade project like Saleor structures its payment utilities, and how to write a PR that maintainers will take seriously.
 
---
 
## Understanding the Issue
 
### Problem Description
 
Saleor's `externalUrl` field on the `TransactionCreate` mutation rejects relative URLs — such as `/dashboard/apps/QXBwOjY=/app/app/transactions/...` — even though these are valid paths per RFC 3986. The `=` character in Base64-encoded path segments triggers the rejection.
 
### Expected Behavior
 
The validation logic should accept any relative URL that begins with `/` and contains only characters that are valid in URI path segments per RFC 3986, including `=`.
 
### Current Behavior
 
Saleor logs a warning and rejects the input with: `"Input should be a valid URL, relative URL without a base."` This causes the payment transaction flow to fail for apps that use relative URLs.
 
### Affected Components
 
- `saleor/payment/utils.py` — specifically the `validate_external_url` function (around line 891), which uses Django's `URLValidator`. Django's validator does not accept relative URLs out of the box.
---
 
## Reproduction Process
 
### Environment Setup
 
[Notes on setting up your local development environment - challenges you faced, how you solved them]
 
### Steps to Reproduce
 
1. Set up a Saleor instance locally using Docker (`docker compose up`)
2. Create a checkout using the dummy payment method
3. Select a delivery method for the checkout
4. Initialize a transaction via the `TransactionCreate` mutation, passing a relative URL as the `externalUrl` value — for example: `"/dashboard/apps/QXBwOjI=/app/app/transactions/75ad3453-fddb-4e32-abf9-2804702e4fe0"`
5. Observe that Saleor logs a warning and rejects the input: `"Input should be a valid URL, relative URL without a base."`
### Reproduction Evidence
 
- **Commit showing reproduction:** (https://github.com/cmirandavega/saleor/tree/fix-issue-18012)
- **Screenshots/logs:** [<img width="1913" height="907" alt="Screenshot 2026-06-17 205838" src="https://github.com/user-attachments/assets/f6bbb41a-c63c-4e14-978d-74c04ce8a0e8" />] 
- **My findings:** Running the TransactionCreate mutation with a relative externalUrl value containing = characters returns "Invalid format of externalUrl." — confirming that Saleor's validation rejects valid relative URLs. The bug is reproducible locally and matches the behavior described in the issue. 

---
 
## Solution Approach
 
### Analysis
 
[Your analysis of the root cause - what's causing the issue?]
 
### Proposed Solution
 
[High-level description of your fix approach]
 
### Implementation Plan
 
sing UMPIRE framework (adapted):

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
 
- [RFC 3986 — URI Generic Syntax](https://datatracker.ietf.org/doc/html/rfc3986)
- [Django URLValidator documentation](https://docs.djangoproject.com/en/stable/ref/validators/#django.core.validators.URLValidator)
- [GitHub issue #18012 comments](https://github.com/saleor/saleor/issues/18012)
