---
description: Find all unresolved Google Doc/Sheet/Slides comments where you are @mentioned
argument-hint: "[days] (default: 30)"
---

# Open Comments

Find all **unresolved comments that @mention me** across Google Docs, Sheets, and
Slides. Surfaces threads that arrived via email notification but may have scrolled
below the fold — including docs you never opened.

**Time scope:** $ARGUMENTS days (default to 30 if blank or not a number).

---

## Workflow

### Phase 1 — Discover via Gmail

Google Drive has no "all comments mentioning me" endpoint. The workaround is
Gmail: Google sends a notification email for every @mention from
`comments-noreply@docs.google.com`.

1. **Search Gmail** — paginate through ALL results:
   ```
   from:comments-noreply@docs.google.com newer_than:{DAYS}d
   ```
   Use `search_gmail_messages` with `page_size: 25`. Keep calling with the
   returned `page_token` until no more results.

2. **Batch-fetch bodies** — call `get_gmail_messages_content_batch` (max 25 IDs
   per call). From each email body, extract:
   - **Document ID** — from the URL pattern:
     - `docs.google.com/document/d/{ID}/...`
     - `docs.google.com/spreadsheets/d/{ID}/...`
     - `docs.google.com/presentation/d/{ID}/...`
   - **Document title** — from the email body text
   - **Commenter name** — who @mentioned you
   - **Date** — email send date
   - **Comment snippet** — the text around the @mention
   - **Doc type** — `document`, `spreadsheet`, or `presentation`

3. **Deduplicate** by document ID. You now have the unique set of docs that
   @mentioned you in the time window.

### Phase 2 — Check resolution status

For each unique document, call the matching comments API:

| Doc type | Tool |
|----------|------|
| document | `list_document_comments` |
| spreadsheet | `list_spreadsheet_comments` |
| presentation | `list_presentation_comments` |

From the returned comments, keep ONLY threads that:
- Are **NOT** marked `[RESOLVED]`
- Contain your email address in the comment content OR are replies to a thread
  that @mentions you

Determine whether YOU have replied in each thread by checking if any reply
author matches your name/email.

### Phase 3 — Report

Present results in two sections:

---

**Needs Your Response** — unresolved threads where you have NOT replied.
Sort newest first.

| Document | Who | What they need | Date | Link |
|----------|-----|----------------|------|------|

---

**You Replied But Thread Still Open** — unresolved threads where you DID reply
but the thread remains open. May need follow-up or the other person to close it.

| Document | Who | Status | Date | Link |
|----------|-----|--------|------|------|

---

End with: "**Summary:** X threads need your response across Y documents.
Z threads are open but you already replied."

---

## Required MCP tools

All from the **Google Workspace** plugin (standard in Salesforce Claude deployments):

- `search_gmail_messages`
- `get_gmail_messages_content_batch`
- `list_document_comments`
- `list_spreadsheet_comments`
- `list_presentation_comments`

No additional servers or API keys needed.

## Error handling

- If a document returns an access error on the comments API, report it as
  "(unable to verify — you may lack access)" and continue. Do not fail the run.
- If Gmail returns zero results, report "No comment notifications found in the
  last {DAYS} days."
- Do NOT include threads where you are the original commenter mentioning someone
  else, unless they @mention you back in the same thread.
