---
name: qa-test-plan
description: Produces a structured QA test plan from a feature description. Cases are written in a behavioral style — concrete enough to execute manually, neutral enough to map to automated E2E later. Use when the user asks for a test plan, test cases, or QA coverage for a feature.
---

# QA Test Plan

## Quick start

Read the feature description, identify the user-facing behaviors in scope, then output one markdown table per group: Functional, Negative, Error / Failure, Edge cases (and Non-functional when relevant). Each case is one row with ID, Title, Preconditions, Steps, Expected, Priority.

## Example

### Functional
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-01 | Login — valid creds | User exists | 1. Open /login<br>2. Enter email<br>3. Enter password<br>4. Submit | Redirect to /dashboard | P0 |

### Negative
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-02 | Login — wrong password | User exists | 1. Open /login<br>2. Enter email<br>3. Enter wrong password<br>4. Submit | Inline error shown, stays on /login | P1 |

## When To Use

- The user asks for a test plan, test cases, QA coverage, or "how would you test this"
- Planning verification for a new feature, before or after implementation
- Reviewing whether a feature is testable or has coverage gaps

## When Not To Use

- The user wants automated test code — go write the tests directly
- Exploratory testing charters — different artifact
- Verifying a single bug fix where one repro step is enough
- Pure unit-test design for an internal function

## Core Rules

1. Steps are user-visible actions ("Submit the form", "Open /login"), never selectors, fixtures, or framework code.
2. Expected results are observable outcomes (URL, visible text, state the user can see), never internal state.
3. One assertion focus per case — split when a case verifies two unrelated things.
4. Default coverage per behavior: happy path + key negatives + 1–2 edge cases. Stop when extra cases stop reducing risk.
5. Consider error scenarios for every flow with user-visible consequence (submit, save, payment, async). If the flow is read-only with a trivial loading state, write `no error cases — read-only` instead of silently skipping.
6. One case per failure mode, not per HTTP code. "Save fails — error toast shown, form remains editable, no data loss" covers a class.
7. Group cases by user-facing area, not by technical layer.
8. Priority: P0 blocks release / core path, P1 important non-blocking, P2 nice-to-have / rare.
9. Describe inputs by *kind*, not literal values. "Enter a valid email" — not `a@b.com`. "Enter text one character over the limit" — not "Enter 501 chars". Specific values belong in fixtures or are picked by the tester at execution time.

## Workflows

1. Read the feature description and list the user-facing behaviors in scope.
2. Note ambiguities in a short "Open questions / assumptions" block — do not silently guess.
3. For each behavior, draft cases in this order: happy path → negatives → error scenarios (or `no error cases — read-only`) → edge cases.
4. Assign priority per case.
5. Output grouped markdown tables: Functional · Negative · Error / Failure · Edge cases · Non-functional (when relevant).
6. End with "Open questions / assumptions" if any remain.

## Advanced features

See [REFERENCE.md](REFERENCE.md) for the priority rubric, step-writing do/don't, coverage heuristics, and a longer worked example.

## Red Flags

- Steps that mention CSS selectors, data-testids, or framework APIs
- Steps with literal data values baked in (specific emails, magic strings, hard-coded numbers) instead of described by kind
- Expected results phrased as internal state ("the `isLoggedIn` flag is true")
- A case that asserts two unrelated outcomes
- Missing negatives on a form or auth-gated action
- Error / Failure section omitted on a write/save/payment flow with no `no error cases` note
- Every case marked P0
