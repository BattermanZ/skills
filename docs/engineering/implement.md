## What it does

`implement` takes one GitHub issue or a supplied [spec](https://www.aihero.dev/ai-coding-dictionary/spec) from claim to closure. It implements with [tdd](https://aihero.dev/skills-tdd) where possible, runs automated checks, reviews the complete change, proves the reviewed result against the app's live test environment, completes the GitHub issue checklist, commits, pushes, and closes the issue.

The issue is the completion ledger. Assignment prevents two agents taking the same work, acceptance checkboxes are evidence-backed release gates, and the issue stays open until the validated commit is on the remote.

## When to reach for it

You invoke this by typing `/implement` — the agent won't reach for it on its own.

| The work is… | Reach for |
| --- | --- |
| A specific GitHub implementation issue | `/implement <GitHub issue URL or owner/repo#number>` |
| The next available GitHub agent-ready issue | `/implement` with no argument |
| A small, settled spec with no GitHub issue | `/implement <spec path>` |
| A large spec that needs slicing | [to-tickets](https://aihero.dev/skills-to-tickets) first |
| One concrete behaviour you want test-first | [tdd](https://aihero.dev/skills-tdd) directly |
| Already built work that only needs review | [code-review](https://aihero.dev/skills-code-review) directly |

One run owns one work item. With no argument, the skill uses `gh` to select the oldest open, unassigned, unblocked GitHub issue carrying `ready-for-agent`.

## Prerequisites

GitHub issue work needs an authenticated GitHub CLI (`gh`) and a Git remote that resolves unambiguously to the intended GitHub repository. The `ready-for-agent` label must exist for automatic selection; [setup-matt-pocock-skills](https://aihero.dev/skills-setup-matt-pocock-skills) can create it. This personalized workflow does not use GitLab, Linear, or a local issue tracker.

The repository's applicable `AGENTS.md` files must document a usable live test environment and how to exercise the app through its public interface. That procedure is a hard gate: without it, implementation stops before code changes rather than substituting unit tests or production access for live acceptance.

`implement` commits and pushes the current branch. Repository-specific approval, branch, deployment, and post-push rules still govern those actions.

The run starts from a clean worktree and records the starting commit as its review boundary. Pre-existing changes or unrelated unpushed commits require an explicit decision before the issue is claimed, preventing the final commit or push from sweeping in somebody else's work.

## The claim

Assignment is a lock on the work item:

- An unassigned GitHub issue is claimed with `gh issue edit --add-assignee @me` before code changes.
- An issue already assigned to anyone is left alone until the user gives explicit approval for that exact issue. Typing `/implement` by itself is not approval.
- An invocation with no argument cannot select an assigned issue, because there is no explicit approval to override its lock.

The claim is fetched again after assignment, so a failed or racing assignment stops the run instead of allowing two implementers to proceed.

## The final gate

The leading idea is **live acceptance**: automated tests establish fast confidence while building, but the last pre-release check uses the running app as a user or API consumer would.

[Code review](https://aihero.dev/skills-code-review) comes before live acceptance and includes committed, staged, unstaged, and untracked changes. Review fixes go back through the affected automated checks before the agent starts or reaches the test environment documented in `AGENTS.md`. A defect found there returns the work to checks and review, so the final live pass always covers the reviewed code.

Only evidence from that final pass can complete implementation or acceptance checkboxes. Every required box must be checked before the commit. The push must succeed—and any repository-required post-push checks must pass—before the issue closes.

## Common questions

**What happens if I invoke it without an issue number?**

It scans the current GitHub repository for open `ready-for-agent` issues, removes anything assigned or blocked, and takes the oldest eligible issue by issue number. If none is available, it stops without changing anything.

**Which issue tracker does it use?**

GitHub only, through the authenticated `gh` CLI. It stops if the repository cannot be resolved to GitHub and never silently falls back to GitLab, Linear, or local files.

**Will it take an issue that somebody else is already working on?**

Only after you explicitly approve that exact assigned issue. Assignment is treated as a concurrency guardrail, not a hint.

**When does it test the running application?**

After code review and all review fixes. Live acceptance is the final pre-commit check, not an early development loop.

**Do passing tests satisfy the acceptance checkboxes?**

Not on their own. Tests and typechecking must pass, but each implementation or acceptance checkbox is completed only when the reviewed behaviour has been exercised successfully in the documented live test environment.

**Does it commit and push?**

Yes. Both are part of the run. The issue closes only after the remote contains the commit and any post-push gates required by the repository have passed.

**Can code review see changes that have not been committed yet?**

Yes. The review surface includes changes since the fixed point plus staged, unstaged, and untracked work, which preserves review-before-live-test-before-commit ordering.

## It's working if

- An unassigned issue shows the authenticated user as assignee before the first code edit.
- An assigned issue produces an approval stop instead of an implementation diff.
- Automated checks pass before review, and the trace shows review findings resolved before live acceptance begins.
- The final acceptance evidence comes from the running app's public interface and maps to every required checkbox.
- The issue's required boxes are checked before the commit, the commit appears on the remote, and only then does the issue close.
- Any failed gate leaves the issue open and names the exact blocker.

## Where it fits

`implement` is the build-and-release step of the main chain:

```txt
grill-with-docs → to-spec → to-tickets → implement
```

[to-tickets](https://aihero.dev/skills-to-tickets) creates the agent-ready issues it consumes, [tdd](https://aihero.dev/skills-tdd) drives the implementation at agreed seams, and [code-review](https://aihero.dev/skills-code-review) supplies the two-axis review before live acceptance. [ask-matt](https://aihero.dev/skills-ask-matt) is the router over the whole set.
