---
name: start
description: Start active work on a Linear issue from Codex. Use when the user invokes `$Start TICKET-123` or says to start a Linear ticket. Resolve the issue from Linear, move it to the `In Progress` state, begin implementation work in the current workspace, and keep the same Linear issue updated throughout the thread when plans are approved, meaningful file edits are made, subagents finish material work, and git commits are created.
---

# Start

## Overview

Start work on a Linear ticket with one command. Treat this as an end-to-end workflow: identify the issue, move the issue into active work, gather implementation context from the repo, begin the engineering work, and keep the Linear ticket updated as the work evolves.

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

4. Establish the started ticket as the active Linear context for the thread.
   Reuse the same resolved issue identifier for later Linear updates during this work unless the user explicitly switches tickets.
   Keep the issue title, identifier, current description, and team handy so later updates can preserve existing content instead of overwriting it blindly.

5. Build implementation context and begin the work in the current workspace.
   Read the issue description, attachments, linked items, and recent comments if they matter to the requested work.
   Inspect the repository for the relevant code paths, tests, and local constraints before changing code.
   Infer a concrete implementation plan from the issue and the codebase.
   If the ticket is clear enough to act on, start making code changes in the same turn instead of stopping at status updates.
   If a truly blocking ambiguity remains after reading the issue and repo, ask one concise question; otherwise proceed.

6. Mirror approved plans into the Linear issue description.
   When a concrete plan is proposed and the user approves it, update the issue description with `mcp__linear__save_issue`.
   Preserve the existing description. Append or refresh a concise `Implementation plan` section rather than replacing the whole issue body.
   Only write approved plans. Do not post speculative, rejected, or outdated plans as the current plan.

7. Log meaningful file-edit progress back to Linear.
   After each meaningful batch of file edits that leaves real working changes on disk, post a concise progress comment with `mcp__linear__save_comment`.
   Summarize what changed, why it changed, and list the key files when that helps future readers.
   Group tightly related edits into one comment instead of spamming per file.
   Skip comments for no-op inspections, abandoned edits, purely mechanical formatting-only churn unless it materially affects the task, or changes that are reverted before they represent real progress.

8. Log completed subagent or delegated work back to Linear.
   When a subagent or delegated task completes with material output, post a concise Linear comment summarizing what it finished, the outcome, and any follow-up still needed.
   Mention changed files or validated results when known.
   Skip comments for failed, canceled, or no-op delegations.

9. Validate the implementation as soon as there is something concrete to verify.
   Run the narrowest useful tests, linters, or build checks for the code you changed.
   Prefer targeted validation that matches the scope of the ticket.
   If validation cannot run, state the reason clearly in the final summary.

10. When a git commit is created for work on this ticket, add a Linear comment to the same issue.
   Use `mcp__linear__save_comment` with the issue identifier from step 2.
   Post one comment per commit after the commit succeeds.
   Include the commit SHA if available, the commit subject, and a short summary of the user-facing or code-facing changes.
   Keep the comment concise and factual.
   Do not post a comment for uncommitted changes, abandoned changes, or failed commit attempts.

11. Report the result succinctly.
   Confirm the issue that was started.
   Confirm whether the Linear state changed or was already correct.
   Summarize the implementation work completed in the repo.
   Call out which Linear updates were made during the work, plus validation results or blockers.
   When a commit occurs later in the thread, mention that the commit summary was posted to Linear.

## Guardrails

- Read the issue before attempting any update.
- Use Linear as the source of truth for the ticket details and available workflow states.
- Reuse the same resolved issue for later Linear description updates and comments throughout the thread.
- Treat `$Start TICKET-123` as authorization to begin the actual implementation work unless the user explicitly asked for setup only.
- Do not guess ticket IDs, issue titles, or team-specific workflow names.
- Do not use a partial match for the status; require an exact `In Progress` status name.
- Do not stop after moving the ticket to `In Progress` if actionable implementation work is possible.
- Do not overwrite the issue description when adding a plan; preserve the original ticket content and update only the plan section you own.
- Do not post progress comments for trivial noise; prefer fewer, meaningful updates that help a teammate understand the work.
- Do not claim a Linear plan update or progress comment happened unless the MCP call actually succeeded.
- Do not summarize subagent work as complete if you still need to review or integrate it; describe the real status.
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
- If an implementation plan is approved, update the Linear issue description with that approved plan.
- As meaningful edit batches land, post concise Linear progress comments.
- If a subagent completes useful work, post a concise Linear comment summarizing it.
- Run the narrowest useful validation for the changes made.
- Reply with the outcome and implementation summary.
- After each later git commit for `SER-142`, add a concise summary comment to the same Linear issue.
