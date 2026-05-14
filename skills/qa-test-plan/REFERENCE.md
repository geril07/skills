# QA Test Plan Reference

## Priority Rubric

- **P0** — blocks release. Core happy path, auth, payment, anything that can lose data or expose information. If this fails, do not ship.
- **P1** — important. Common negatives, key error scenarios, primary edge cases real users will hit.
- **P2** — nice-to-have. Rare paths, low-likelihood edges, polish.

If every case is P0, the priorities are not doing work — re-rank.

## Step-writing — Do / Don't

Do:
- "Open /login"
- "Submit the form with a valid email and a wrong password"
- "Click the **Delete** button on the first row"
- "Wait for the success toast"

Don't:
- "Click `[data-testid=submit-btn]`" — selector leak
- "Submit with email `qa@example.com`" — literal data, not behavior; the tester or fixture picks the value
- "Call `POST /api/login` with body `{…}`" — implementation, not user behavior
- "Set `state.user = null`" — internal state, not observable
- "Confirm the redux store updates" — internal state

Describe inputs by their *kind* (valid, invalid, empty, at the limit, one over the limit, contains unicode) so a manual tester can pick a value and an automated test can use a fixture without changing the case.

Expected results work the same way: describe what the user sees or where they end up, not what the code did.

## Coverage Heuristics

Negatives to consider on any input or auth-gated action:
- Empty / missing required field
- Wrong type or malformed input
- Out-of-range or too-long value
- Unauthorized / wrong credentials
- Forbidden / role mismatch
- Duplicate / conflict (already-used email, taken slug)

Error / failure modes to consider on any write or async flow:
- Network unreachable
- Server 5xx
- Timeout / slow response
- Optimistic update that later fails (rollback)
- Partial success in a batched operation

Edge cases to consider:
- Boundary values (0, 1, max, max+1, empty list, single item, very long string)
- Concurrent action by the same user in another tab
- Stale data (entity deleted between load and submit)
- Permissions changed mid-session
- Unicode / RTL / emoji in user-supplied text

Non-functional, when relevant:
- Accessibility: keyboard nav, focus order, screen-reader labels on new controls
- Performance: visible budget (e.g., list renders under N items without jank)
- Responsive: target breakpoints render without layout break
- Internationalization: copy not truncated in long-locale languages

## Grouping

Group by user-facing area first, then by category within:

```
## <Area / feature>
### Functional
### Negative
### Error / Failure
### Edge cases
### Non-functional
```

Skip a category when it has no cases — but on a write flow, write `no error cases — read-only` (or a similar one-line note) instead of silently dropping Error / Failure, so the omission is visible.

## Worked Example — "Add a comment to a post"

Description: Logged-in users can post a comment up to 500 chars on a post. Comments appear immediately after submit. Authors can delete their own comments.

### Comments — Functional
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-01 | Post a comment | User logged in, post exists | 1. Open the post<br>2. Enter valid comment text<br>3. Submit | Comment appears at the top of the list, attributed to current user | P0 |
| TC-02 | Delete own comment | User has posted a comment | 1. Open the post<br>2. Click **Delete** on the user's own comment<br>3. Confirm | Comment removed from the list | P0 |

### Comments — Negative
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-03 | Submit empty comment | User logged in | 1. Open the post<br>2. Leave input empty<br>3. Submit | Submit disabled or no comment created; hint shown | P1 |
| TC-04 | Submit over-length comment | User logged in | 1. Open the post<br>2. Enter text one character over the limit<br>3. Submit | Inline length error, comment not posted | P1 |
| TC-05 | Anonymous user tries to comment | Logged out | 1. Open the post<br>2. Focus the input or submit | Prompted to log in; no comment created | P1 |
| TC-06 | Delete someone else's comment | Other user's comment visible | 1. Open the post<br>2. Inspect another user's comment | No **Delete** affordance shown | P0 |

### Comments — Error / Failure
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-07 | Submit while offline | User logged in, network offline | 1. Open the post<br>2. Enter valid comment text<br>3. Submit | Error message shown, input retains its value, no duplicate after retry | P1 |
| TC-08 | Server rejects submit (5xx) | User logged in, server failing | 1. Submit a comment | Error message shown, comment not appended, retry possible | P1 |

### Comments — Edge cases
| ID | Title | Preconditions | Steps | Expected | Priority |
|----|-------|---------------|-------|----------|----------|
| TC-09 | Comment at the exact length limit | User logged in | 1. Enter text at the maximum allowed length<br>2. Submit | Comment posted successfully | P2 |
| TC-10 | Post deleted between load and submit | User logged in | 1. Load the post<br>2. Author deletes the post in another tab<br>3. Submit a comment | Friendly error, no orphaned comment created | P2 |
| TC-11 | Unicode, RTL, and emoji content | User logged in | 1. Enter text containing RTL characters and emoji<br>2. Submit | Comment displays correctly, no encoding artifacts | P2 |

### Open questions / assumptions
- Assumed comments are not edited (only created/deleted) — confirm.
- Assumed the character limit is enforced on both client and server.
