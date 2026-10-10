# Bug Report Template

The structure I use for every bug report (the same fields I use in Jira). All bugs in [bug-reports/](../bug-reports/) follow it.

---

**Title:**
One specific line with the feature area and the problem. Format: `[Area] What is wrong`

**Short description:**
1–3 sentences: what the bug is and why it matters.

**Preconditions:**
System, account and data state needed before the steps.

**Steps to reproduce:**
Numbered, exact, minimal steps that anyone can follow.

**Actual result:**
What actually happened, exactly.

**Expected result:**
What should happen, linked to the acceptance criteria or spec when possible.

**Test data:**
Exact test data used (accounts, values, IDs). Test data only.

**Environment:**
Build, environment (staging/production), browser/OS/device, feature flags if relevant.

**Evidence:**
Screenshots, screen recordings, network captures, console logs, Postman results or SQL output. Sensitive data removed.

**Severity:**
Impact of the bug (Critical / High / Medium / Low). See the scale below.

**Priority:**
How urgent the fix is (P0 / P1 / P2 / P3). See the scale below.

**Repro rate:**
How often it happens (for example "5/5" or "intermittent, about 30%").

**Workaround:**
Is there a way to avoid the problem now? If not: "None".

**Investigation notes:**
What I found myself (DevTools, API, SQL, hypotheses) to help the developer start faster. Hypotheses are marked as not verified.

---

## Severity scale

| Severity | Meaning |
|---|---|
| **Critical** | Crash, data loss or corruption, security issue, or a core flow (booking, payment, login) completely blocked with no workaround |
| **High** | Major function broken or badly degraded; a workaround may exist but is painful |
| **Medium** | Function partly broken or wrong in non-critical ways; a workaround exists |
| **Low** | Cosmetic, small UX issue, or an edge case with very small impact |

## Priority scale

| Priority | Meaning |
|---|---|
| **P0** | Fix immediately. Blocks the release. |
| **P1** | Urgent. Fix in the current or next sprint. Usually fixed before release, unless the PM and engineering team formally accept the risk for that release. |
| **P2** | Fix in the next 1–2 sprints. |
| **P3** | Backlog, fix when there is time. |

Severity and priority are separate. A **Low severity, P0** bug can exist (for example, a typo on a legal page that must be fixed now), and so can a **Critical severity, P2** bug (for example, a crash in a rarely used admin tool with a workaround).
