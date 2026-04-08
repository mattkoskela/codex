---
name: start
description: Start active work on a Linear issue from Codex. Use when the user invokes `$Start TICKET-123` or says to start a Linear ticket. Resolve the issue from Linear, move it to the `In Progress` state, begin implementation work in the current workspace, and after each git commit during that work add a concise commit summary comment to the same Linear issue.
---

# Start

## Overview

Start work on a Linear ticket with one command. Treat this as an end-to-end workflow: identify the issue, move the issue into active work, gather implementation context from the repo, begin the engineering work, and keep the Linear ticket updated when commits are made.

## Workflow

1. Parse the ticket identifier from the user's message.
   Accept forms like `$Start ABC-123`, `Start ABC-123`, or `start ABC-123`.
   Normalize whitespace, but do not change the ticket key itself.

2. Load the issue from Linear before doing anything else.
   Use `mcp__linear__get_issue` with the provided ticket identifier.
   If the issue is not found, stop and tell the user that the ticket could not be resolved.

3. Move the Linear issue to `In Progress`.
   First inspect the issue's team from the fetched issue.
   Then use `mcp__linear__list_issue_statuses` for that team and confirm an exact `In Progress` status exists.
   If `In Progress` exists, use `mcp__linear__save_issue` to set the issue state to `In Progress`.
   If the issue is already `In Progress`, treat the update as a no-op and say so briefly.
   If the team does not have an exact `In Progress` state, stop and tell the user instead of guessing a substitute state.

4. Build implementation context and begin the work in the current workspace.
   Read the issue description, attachments, linked items, and recent comments if they matter to the requested work.
   Inspect the repository for the relevant code paths, tests, and local constraints before changing code.
   Infer a concrete implementation plan from the issue and the codebase.
   If the ticket is clear enough to act on, start making code changes in the same turn instead of stopping at status updates.
   If a truly blocking ambiguity remains after reading the issue and repo, ask one concise question; otherwise proceed.

5. Validate the implementation as soon as there is something concrete to verify.
   Run the narrowest useful tests, linters, or build checks for the code you changed.
   Prefer targeted validation that matches the scope of the ticket.
   If validation cannot run, state the reason clearly in the final summary.

6. When a git commit is created for work on this ticket, add a Linear comment to the same issue.
   Use `mcp__linear__save_comment` with the issue identifier from step 2.
   Post one comment per commit after the commit succeeds.
   Include the commit SHA if available, the commit subject, and a short summary of the user-facing or code-facing changes.
   Keep the comment concise and factual.
   Do not post a comment for uncommitted changes, abandoned changes, or failed commit attempts.

7. Report the result succinctly.
   Confirm the issue that was started.
   Confirm whether the Linear state changed or was already correct.
   Summarize the implementation work completed in the repo.
   Call out validation results or blockers.
   When a commit occurs later in the thread, mention that the commit summary was posted to Linear.

## Guardrails

- Read the issue before attempting any update.
- Use Linear as the source of truth for the ticket details and available workflow states.
- Reuse the same resolved issue for later Linear comments after commits.
- Treat `$Start TICKET-123` as authorization to begin the actual implementation work unless the user explicitly asked for setup only.
- Do not guess ticket IDs, issue titles, or team-specific workflow names.
- Do not use a partial match for the status; require an exact `In Progress` status name.
- Do not stop after moving the ticket to `In Progress` if actionable implementation work is possible.
- Do not post commit-summary comments before the commit exists.
- Do not combine multiple commits into one comment unless the user explicitly asks for that.

## Example

User request:

```text
$Start SER-142
```

Expected behavior:

- Fetch `SER-142` from Linear.
- Move the issue to `In Progress`.
- Inspect the relevant code and begin implementing the ticket in the current workspace.
- Run the narrowest useful validation for the changes made.
- Reply with the outcome and implementation summary.
- After each later git commit for `SER-142`, add a concise summary comment to the same Linear issue.
