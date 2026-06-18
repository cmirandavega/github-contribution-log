# github-contribution-log

# Contribution 1: Bug: externalUrl field validation rejects valid relative URLs
 
**Contribution Number:** 1  
**Student:** Christopher Miranda Vega  
**Issue:** https://github.com/saleor/saleor/issues/18012  
**Status:** Phase I — Completed, Phase II - Completed, Phase III - Completed, Phase IV - Awaiting Response
 
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
 
Saleor rejects the input with: "Invalid format of externalUrl." This causes the payment transaction flow to fail for apps that use relative URLs.

### Affected Components
 
saleor/graphql/payment/mutations/transaction/transaction_create.py — specifically the validate_external_url method, which uses Django's URLValidator. Django's validator does not accept relative URLs out of the box. Since TransactionUpdate inherits from TransactionCreate, this fix covers both mutations.
---
 
## Reproduction Process
 
### Environment Setup
 
Setting up the local development environment on Windows required running Saleor through WSL2 and Docker Desktop. The main challenges were getting Docker connected to WSL2 (resolved by enabling WSL2 integration in Docker Desktop settings), and a filesystem permission issue caused by cloning the repo to /mnt/c/... (the Windows filesystem) which caused uv package installations to fail. This was resolved by copying the project into the WSL2 native filesystem at ~/saleor and running all commands from there. The virtual environment also needed a full rebuild (rm -rf .venv && uv sync) after the move due to corrupted package installs. Once the environment was stable, Docker containers for the database, cache, dashboard, and mail server were started via docker compose up from the .devcontainer folder.
 
### Steps to Reproduce
 
1. Set up a Saleor instance locally using Docker (`docker compose up`)
2. Create a checkout using the dummy payment method
3. Select a delivery method for the checkout
4. Initialize a transaction via the `TransactionCreate` mutation, passing a relative URL as the `externalUrl` value — for example: `"/dashboard/apps/QXBwOjI=/app/app/transactions/75ad3453-fddb-4e32-abf9-2804702e4fe0"`
5. Observe that Saleor logs a warning and rejects the input: `"Invalid format of `externalUrl`."`
### Reproduction Evidence
 
- **Commit showing reproduction:** (https://github.com/cmirandavega/saleor/tree/fix-issue-18012)
- **Screenshots/logs:** [<img width="1913" height="907" alt="Screenshot 2026-06-17 205838" src="https://github.com/user-attachments/assets/f6bbb41a-c63c-4e14-978d-74c04ce8a0e8" />] 
- **My findings:** Running the TransactionCreate mutation with a relative externalUrl value containing = characters returns "Invalid format of externalUrl." — confirming that Saleor's validation rejects valid relative URLs. The bug is reproducible locally and matches the behavior described in the issue. 

---
 
## Solution Approach
 
### Analysis
 
The root cause is in the `TransactionCreate` mutation's input validation for the `externalUrl` field. The validation relies on Django's built-in `URLValidator`, which is designed to only accept absolute URLs containing a scheme (e.g. `https://`) and a host. When a relative URL like `/dashboard/apps/QXBwOjI=/app/app/transactions/...` is passed in, Django's validator rejects it immediately because it has no scheme or host — even though the URL is perfectly valid per RFC 3986, which explicitly allows relative paths starting with `/`. The `=` character in Base64-encoded path segments makes this worse, as it further confuses the validator.
 
### Proposed Solution
 
Update the `externalUrl` validation logic in `saleor/graphql/payment/mutations/transaction/transaction_create.py` to handle relative URLs as a separate case. Before passing the value to Django's `URLValidator`, add a check: if the value starts with `/`, validate it against a regex that accepts valid URI path characters per RFC 3986 instead. If it does not start with `/`, continue using the existing `URLValidator` as before. This approach is minimal, targeted, and avoids changing behavior for absolute URLs.
 
### Implementation Plan
 
Using UMPIRE framework (adapted):

**Understand:** The `TransactionCreate` mutation rejects valid relative URLs passed as `externalUrl` because the underlying validator uses Django's `URLValidator`, which only accepts absolute URLs with a scheme and host. Relative URLs starting with `/` are valid per RFC 3986 but get rejected, causing the transaction flow to fail.

**Match:** The validation lives in `saleor/graphql/payment/mutations/transaction/transaction_create.py`. The pattern to follow is how other validators in the codebase handle conditional logic — check the input type first, then apply the appropriate validation. A similar URL validation fix was done in `saleor/core/utils/url.py` for the `validate_storefront_url` function (issue #8556).

**Plan:** 
1. Locate the `externalUrl` validation logic in `saleor/graphql/payment/mutations/transaction/transaction_create.py`
2. Add a pre-check: if the value starts with `/`, validate it as a relative path using a regex for valid URI path characters per RFC 3986 instead of passing to Django's `URLValidator`
3. If the value does not start with `/`, continue using the existing `URLValidator` as before
4. Add unit tests in `saleor/graphql/payment/tests/mutations/transaction/test_transaction_create.py` covering valid relative URLs, valid absolute URLs, and invalid inputs


**Implement:** (https://github.com/cmirandavega/saleor/tree/fix-issue-18012)

**Review:** Before submitting the PR I will verify:
- Commit message is under 50 characters and uses imperative form (e.g. `fix: allow relative URLs in externalUrl validation`)
- A changelog entry is added under `Unreleased → Other changes` in `CHANGELOG.md`
- The change does not introduce new validation restrictions (not a breaking change per Saleor's guidelines)
- Pre-commit hooks pass (`uv run pre-commit run --all-files`)

**Evaluate:** 
- Run `uv run poe test saleor/graphql/payment/tests/mutations/transaction/test_transaction_create.py` to confirm new tests pass. Manually verify by running the `transactionCreate` mutation with the failing relative URL from the issue and confirming it no longer returns `"Invalid format of externalUrl."`
---
 
## Testing Strategy
 
### Unit Tests
 
- [x] Test case 1: Valid relative URL containing `=` characters is accepted — `test_transaction_create_external_url_accepts_relative_url_by_app`
- [x] Test case 2: Valid absolute URL still passes validation unchanged — `test_transaction_create_external_url_accepts_absolute_url_by_app`
- [x] Test case 3: Invalid relative URL containing a space is rejected with `INVALID` error code — `test_transaction_create_external_url_rejects_invalid_relative_url_by_app`

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2
No integration tests were written for this fix. The change is contained to a single validation method and is fully covered by unit tests.
### Manual Testing

Reproduced the bug locally by running the `TransactionCreate` mutation in the GraphQL playground (`http://localhost:8000/graphql/`) with a relative `externalUrl` value of `/dashboard/apps/QXBwOjI=/app/app/transactions/75ad3453-fddb-4e32-abf9-2804702e4fe0`. Confirmed the error `"Invalid format of externalUrl."` before the fix. After applying the fix, the same mutation accepted the relative URL without errors.

---

## Implementation Notes

### Week [1] Progress

Located the `validate_external_url` method in `saleor/graphql/payment/mutations/transaction/transaction_create.py`. Confirmed the root cause: Django's `URLValidator` rejects relative URLs by design. Added a module-level `RELATIVE_URL_REGEX` compiled from RFC 3986 valid path characters, then updated the method to check if the input starts with `/` and validate it against the regex instead of passing it to `URLValidator`. The absolute URL branch is unchanged. Discovered that `TransactionUpdate` inherits from `TransactionCreate`, so the fix covers both mutations automatically. Added three unit tests and ran them locally — all 4 tests passed (3 new + 1 existing). Opened a PR to the upstream Saleor repository.

 
### Code Changes
 
- **Files modified:** `saleor/graphql/payment/mutations/transaction/transaction_create.py`, `saleor/graphql/payment/tests/mutations/test_transaction_create.py`, `CHANGELOG.md`
- **Key commits:** https://github.com/cmirandavega/saleor/commit/3d0709acb298b6ce06391f72222c24ec5be820a3
- **Approach decisions:** Used `re.fullmatch` with a compiled regex rather than modifying Django's `URLValidator` directly — keeps the change minimal, targeted, and easy for maintainers to review. The relative URL branch returns early so the existing absolute URL logic is completely untouched.
 
## Pull Request

**PR Link:** https://github.com/saleor/saleor/pull/19325

**PR Description:** 
Relative URLs starting with "/" are valid per RFC 3986 but were rejected 
by Django's URLValidator, which only accepts absolute URLs. This caused 
the TransactionCreate mutation to fail for apps that use relative externalUrls.

- Add RELATIVE_URL_REGEX to validate path-absolute references
- Update validate_external_url to accept relative URLs as a separate case
- Add three unit tests covering relative URL, absolute URL, and invalid input

Fixes #18012

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** Awaiting review

---
 
## Learnings & Reflections
 
### Technical Skills Gained
 
Django's URL validation internals — learned that URLValidator only accepts absolute URLs and why, which led you to understand how to extend validation logic conditionally
RFC 3986 — I now understand what makes a URL valid at the spec level, not just in practice
GraphQL mutations in Django — I navigated a real production GraphQL mutation, understood how input validation is structured, and traced an error from the API layer down to a specific function
Regex for input validation — I wrote and tested a fullmatch regex for URI path characters, understanding why fullmatch is safer than match or search
pytest in a large codebase — I learned how to write tests that follow an existing project's fixtures and conventions rather than starting from scratch
Docker + WSL2 local dev setup — I set up a full multi-container development environment on Windows, working through real permission and filesystem issues
Open source contribution workflow — forking, branching, committing with proper messages, writing a PR description that links to an issue, and adding a changelog entry
 
### Challenges Overcome
 
WSL2 and Docker permissions — Docker wasn't connected to WSL2 and the socket wasn't accessible. Solved by enabling WSL2 integration in Docker Desktop settings and using sudo for Docker commands.
Working across two filesystems — the repo was cloned to /mnt/c/... (Windows filesystem) which caused uv package installation to fail with permission errors. Solved by copying the project to ~/saleor (WSL2 native filesystem) and running all commands from there.
Broken virtual environment — moving the project broke the .venv paths and several packages had corrupted installs. Solved by deleting .venv entirely and running uv sync from scratch.
HANDLE_PAYMENTS permission — the GraphQL playground kept returning a permission denied error even after granting superuser access. Solved by creating an App token with the permission directly assigned, which bypasses JWT-based permission checks.
Reproducing the bug — needed a valid checkout ID and app-level auth to trigger the mutation. Worked through creating a checkout, generating an app token, and setting the Authorization header correctly.
Locating the fix — the issue pointed to payment/utils.py but the actual validation lives in transaction_create.py. Solved by using Claude in VS Code to trace the error through the codebase.
 
### What I'd Do Differently Next Time
 
Clone directly into the WSL2 filesystem from the start — working from /mnt/c/... caused filesystem permission issues that took significant time to debug. Starting in ~/ would have avoided all of it.
Set up the app token authentication before trying to reproduce the bug — a lot of time was spent fighting JWT permission errors. Knowing upfront that Saleor requires app-level tokens for payment mutations would have saved several debugging cycles.
Read the full CONTRIBUTING.md before starting — some steps like adding the CHANGELOG entry and pre-commit hooks were discovered late. Reading it end to end first would have made the workflow smoother.
Verify the exact file location of the bug before writing the plan — the issue pointed to payment/utils.py but the actual fix was in transaction_create.py. Tracing the error in the codebase first would have made the implementation plan more accurate from the start.
Fork and sync with upstream before branching — the PR initially included unrelated upstream commits because the fork was out of sync. Syncing first would have kept the PR clean from the start.
 
---
 
## Resources Used
 
- [RFC 3986 — URI Generic Syntax](https://datatracker.ietf.org/doc/html/rfc3986)
- [Django URLValidator documentation](https://docs.djangoproject.com/en/stable/ref/validators/#django.core.validators.URLValidator)
- [GitHub issue #18012 comments](https://github.com/saleor/saleor/issues/18012)
