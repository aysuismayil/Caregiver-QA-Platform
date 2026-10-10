# Evidence

Evidence files are **not included** in this repository. Real bug evidence (screenshots, screen recordings, network captures, Postman results, SQL output) can contain confidential information: internal URLs, user data, internal tools or infrastructure details.

Each bug report lists the type of evidence that was attached to the original ticket.

## What I collect as evidence

| Evidence | When I use it | Tool |
|---|---|---|
| Screenshot | Visible bug state, with the problem marked | Device or browser screenshot |
| Screen recording | Timing-dependent or multi-step bugs | Built-in screen recorder |
| Network capture | Frontend/backend investigation, request and response details | Chrome DevTools Network tab |
| Console log | JavaScript errors and warnings | Chrome DevTools Console |
| Postman result | API reproduction without the UI | Postman |
| SQL query result | Confirming the real saved data | DBeaver (read-only) |

## Rules before attaching evidence

- Replace real user emails, names, phone numbers and addresses with test data.
- Remove auth tokens, session cookies and API keys completely.
- Remove internal URLs and infrastructure details.
- Use staging/test data only, never production customer data.
- Check that screenshots don't show unrelated private information (other tabs, bookmarks, notifications).
