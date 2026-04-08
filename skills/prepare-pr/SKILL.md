---
name: prepare-pr
description: Inspect the actual branch diff, reconcile open GitHub pull request review comments, bring the branch to production-ready quality, verify related documentation and changelog updates, update referenced Linear issues when possible, and rewrite PR titles and bodies so they accurately match the current branch scope. Use when Codex needs to prepare, refresh, or correct PR metadata before review or merge, especially when the existing PR title/body are stale, too narrow, misleading, or inconsistent with the diff, when open review threads still need action, or when the branch may need final polish before merge.
---

# Prepare PR

Use this skill to replace stale or misleading PR metadata with a concise summary grounded in the real diff, clear unresolved review threads by either fixing valid comments or replying with a rationale before resolving them, leave the branch in a production-ready state with docs and changelog coverage updated when needed, and update referenced Linear issues with concise status comments when the Linear skill is available.

## Workflow

1. Gather current repository state first.
- Read the branch name with `git branch --show-current`.
- Inspect `git status --short` before editing anything.
- Preserve unrelated user changes in a dirty worktree. Work around them; do not revert them.
- If the base branch is known, refresh it with `git fetch origin <base>` before treating `origin/<base>` as truth when the environment allows network access.

2. Gather PR context from GitHub when possible.
- Prefer `gh pr view --json number,title,body,baseRefName,headRefName,url` when a PR exists.
- If `gh` is unavailable, unauthenticated, or no PR exists for the branch, fall back to local-only preparation.
- Treat the current branch diff against the PR base as the source of truth.
- Inspect scope with `git diff --stat origin/<base>...HEAD`, `git diff origin/<base>...HEAD`, and `git log --oneline origin/<base>..HEAD`.
- Use conversation history only to understand the user's request or find likely files and commands, never as evidence of PR scope.
- If multiple PRs were discussed earlier, reset scope for this run and derive it again from the current branch, current PR metadata, current review state, and current diff only.
- If the branch has no PR, prepare a proposed title and body locally instead of editing anything remote.

3. Reconcile open review comments when a PR exists.
- Inspect PR-level comments with `gh pr view <number> --comments`.
- Inspect inline review comments with `gh api repos/<owner>/<repo>/pulls/<number>/comments`.
- When thread state matters, query GraphQL `reviewThreads` so unresolved threads are distinguishable from resolved or outdated comments.
- Ignore resolved or outdated threads unless a still-open thread depends on the same code.
- For each unresolved review comment, decide whether the request is correct based on the current code and diff.
- If the comment is correct, implement the fix, run the narrowest useful validation, and resolve the thread after updating the code.
- If the comment should not be addressed, reply with a concise technical rationale and then resolve the thread. Do not resolve a thread silently.
- If thread-resolution APIs are unavailable, leave a clear note in the final response that replies or resolution could not be completed remotely.

4. Do a production-readiness pass before finalizing the PR.
- Review the changed code for obvious correctness, safety, maintainability, and operational issues rather than limiting the work to PR text updates.
- Fix straightforward production-readiness gaps that are in scope for the branch, such as missing error handling, unsafe assumptions, incomplete tests around the change, or follow-through on valid review feedback.
- If a significant issue remains that you cannot safely fix in this pass, call it out plainly in the PR body or final response instead of presenting the branch as ready.

5. Verify the branch before finalizing the PR.
- Inspect repository scripts or build tooling to find the real build command instead of guessing.
- Prefer a full project build when the repo exposes one.
- Run verification after review-driven fixes and before rewriting the final PR summary.
- If the build fails, fix straightforward regressions when feasible.
- If the branch still does not build, mention the exact failing command and blocking error in the final PR body instead of implying the branch is ready.
- If no full build command exists, use the strongest realistic verification available and say exactly what ran.
- Prefer repository-declared entry points such as `package.json` scripts, `Makefile`, or documented CI commands over ad hoc commands.

6. Do a documentation and changelog pass.
- Inspect documentation that is affected by the diff: README files, docs pages, API docs, configuration examples, migration notes, runbooks, or inline developer documentation when those surfaces are touched by the branch.
- Update documentation when behavior, setup, APIs, data flows, or operational expectations changed.
- Keep documentation accurate, clear, and concise. Remove stale statements rather than layering contradictory caveats on top.
- Update the repository changelog when the project keeps one and the branch introduces user-facing or operationally meaningful changes.
- If a changelog file does not exist, do not create one unless the repository already has a clear release-notes convention that the branch should follow.
- Mention documentation or changelog updates in the PR body when they are material to review.

7. Update referenced Linear issues when possible.
- If the `linear` skill is available, use it to inspect and update any Linear issues explicitly referenced by the branch name, commit messages, PR title, PR body, or linked issue references in the repository context.
- Add concise comments that reflect meaningful progress, such as review-driven fixes, production-readiness work, docs updates, changelog updates, or validation results.
- When the branch is actually ready for handoff based on the work completed in this pass, move the referenced issue to `Ready to Deploy`.
- If validation is blocked, critical issues remain, or the branch is not actually ready, do not advance the Linear issue state just to match the PR workflow.
- Keep comments factual and scoped to work actually performed in this pass.
- If no Linear issue is referenced, do not guess.
- If Linear access is unavailable or the `linear` skill cannot be used cleanly, mention that limitation in the final response instead of fabricating an update.

8. Derive scope from the diff.
- Group the work into 2-5 high-level themes.
- Call out migrations, background jobs, API changes, major UI surfaces, and follow-up bug fixes when they are present.
- Call out documentation and changelog updates when they are meaningful parts of the branch.
- Ignore stale framing from the current PR title or body if it no longer matches the branch.
- Do not include work unless it appears in the current diff, commit range, or active review discussion for this PR.
- When a prior PR from the same thread covered related work, assume it is out of scope unless the current branch still contains it.

9. Write a better title.
- Keep it short and specific.
- Cover the dominant shipped scope, not every minor fix.
- Prefer plain action-oriented phrasing such as `Add ...`, `Improve ...`, or `Fix ...`.

10. Rewrite the body.
- Use this default structure:
  ```md
  ## Summary
  ## Included
  ## Migrations
  ## Testing
  ## Notes
  ```
- Include `## Migrations` only when applicable.
- Include `## Notes` only when needed.
- Keep bullets flat and high signal.
- List only tests or checks that actually ran.
- Explicitly mention when full lint, typecheck, or end-to-end validation did not run.
- Include exact migration filenames when schema changes are part of the branch.
- Mention meaningful review-driven fixes if they materially changed the branch.
- Mention documentation and changelog updates when they were part of the preparation work.
- Keep the body reviewer-facing. Summarize behavior and scope, not implementation minutiae.

11. Apply the change.
- If `gh` is available and a PR exists, use `gh pr edit <number> --title ...` and `gh pr edit <number> --body-file ...`.
- Write the new body to a temporary file instead of trying to escape a large multiline string inline.
- If review-thread APIs are available, reply and resolve threads through GitHub after handling each comment.
- If remote editing is unavailable, return the proposed title and body instead.

## Guardrails

- Do not invent verification, rollout details, or risk analysis that did not happen.
- Do not ignore unresolved review comments while updating PR metadata if they are accessible.
- Do not leave a rejected review comment unresolved without a reply.
- Do not mark a thread resolved until either the fix is in place or a rationale has been posted.
- Do not stop at PR copy if the branch still has obvious production-readiness gaps that are reasonably fixable in the current pass.
- Do not skip the repository build step when a realistic full build command exists unless the environment blocks it. If blocked, say that explicitly.
- Do not leave changed behavior undocumented when repository docs are supposed to cover it.
- Do not skip the changelog when the repository keeps one and the branch includes a meaningful entry-worthy change.
- Do not update unrelated Linear issues or infer ticket references from weak signals.
- Do not move a Linear issue to `Ready to Deploy` unless the current pass leaves the branch genuinely ready for that state.
- Do not claim a Linear comment was posted unless the Linear update actually succeeded.
- Do not let earlier conversation turns broaden the PR title or body beyond the current branch diff and current review state.
- Do not mention changes from earlier PRs, abandoned experiments, or follow-up work unless they are part of the current branch.
- Do not turn the body into a file-by-file changelog.
- Preserve required repository-specific PR template structure only when the repo clearly depends on it. Otherwise, prefer an accurate clean rewrite.
- Mention important limitations plainly, especially missing dependency installs or blocked validation.
- Do not claim remote PR updates, comment replies, or thread resolution succeeded unless the GitHub command actually succeeded.
- Do not force-push, rewrite branch history, or clean the worktree unless the user explicitly asks.

## Output

- Ensure the title matches what a reviewer will actually see in the diff.
- Ensure the summary helps a reviewer understand the branch in under a minute.
- Keep the testing section strictly factual.
- Ensure the branch is represented as production-ready only when code quality, docs, changelog, and validation support that claim.
