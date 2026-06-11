# find-google-mentions

A Claude Code plugin that finds all unresolved Google Docs, Sheets, and Slides
comments where you are @mentioned — including docs you never opened.

## The problem

Someone @mentions you in a Google Doc comment. You get an email notification.
It scrolls below the fold. Three weeks later you realize you never responded.

This plugin catches those by searching your Gmail notifications and
cross-referencing against the actual comment resolution status per document.

## Installation

```bash
claude plugin add /path/to/find-google-mentions
```

Or from a Git URL (once published to a repo):

```bash
claude plugin add https://github.com/bgaldino/find-google-mentions
```

## Usage

```
/open-comments         # default: last 30 days
/open-comments 7       # last 7 days
/open-comments 90      # last 90 days
```

## Prerequisites

- Claude Code with the **Google Workspace** MCP plugin connected
  (standard in Salesforce Claude deployments)
- Gmail access (for searching comment notification emails)
- Google Docs/Sheets/Slides comment read access

## How it works

1. Searches Gmail for `from:comments-noreply@docs.google.com` in the time window
2. Extracts unique document IDs from notification email bodies
3. Calls the comments API per document to check thread resolution status
4. Reports two buckets:
   - **Needs your response** — open thread, you haven't replied
   - **You replied, still open** — may need follow-up or the other person to close

## Output example

```
Needs Your Response (newest first)

| Document              | Who            | What they need              | Date   | Link |
|-----------------------|----------------|-----------------------------|--------|------|
| CPQ End of Sale v2    | Annie Wright   | "does this sound correct?"  | Jun 10 | ...  |
| module-2-v2.md        | Ashley Shults  | "Did you say not to mention | May 15 | ...  |
|                       |                |  Stripe?"                   |        |      |

Summary: 11 threads need your response across 8 documents.
2 threads are open but you already replied.
```

## Sharing

To share with a teammate:
1. Send them this repo (or push it to any Git host)
2. They run: `claude plugin add <path-or-url>`
3. Done. `/open-comments` is available in every session.
