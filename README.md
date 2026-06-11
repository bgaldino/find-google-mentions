# find-google-mentions

A Claude Code plugin that finds all unresolved Google Docs, Sheets, and Slides
comments where you are @mentioned — including docs you never opened.

## The problem

Someone @mentions you in a Google Doc comment. You get an email notification.
It scrolls below the fold. Three weeks later you realize you never responded.

This plugin catches those by searching your Gmail notifications and
cross-referencing against the actual comment resolution status per document.

## Installation

**From inside a Claude Code session (GUI/TUI):**

```
/plugin marketplace add bgaldino/find-google-mentions
/plugin install find-google-mentions@bgaldino-find-google-mentions
```

**From the terminal:**

```bash
claude plugin marketplace add bgaldino/find-google-mentions
claude plugin install find-google-mentions@bgaldino-find-google-mentions
```

After install, reload plugins in your current session:

```
/reload-plugins
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

| # | Document          | Who          | What they need             | Date   | Link                        |
|---|-------------------|--------------|----------------------------|--------|-----------------------------|
| 1 | Q4 Launch Plan    | Jane D.      | "Can you confirm the date?"| Jun 10 | [Open](https://docs.go...) |
| 2 | Pricing Model v3  | Alex M.      | "Is this the right tier?"  | Jun 08 | [Open](https://docs.go...) |
| 3 | Partner Deck      | Sam R.       | "Add your slide by Friday" | May 29 | [Open](https://docs.go...) |

You Replied But Thread Still Open

| # | Document          | Who          | Status                     | Date   | Link                        |
|---|-------------------|--------------|----------------------------|--------|-----------------------------|
| 1 | API Design Doc    | Chris T.     | You replied "yes" — still  | Jun 05 | [Open](https://docs.go...) |
|   |                   |              | open                       |        |                             |

Summary: 3 threads need your response across 3 documents.
1 thread is open but you already replied.
```

## Sharing

Send a teammate these two lines:

```
/plugin marketplace add bgaldino/find-google-mentions
/plugin install find-google-mentions@bgaldino-find-google-mentions
```

They paste them into any Claude Code session. Done.
