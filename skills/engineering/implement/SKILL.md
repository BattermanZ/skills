---
name: implement
description: "Claim, implement, validate, and ship one GitHub issue or spec."
disable-model-invocation: true
---

Implement exactly one GitHub issue or spec through to a pushed commit. For issue work, the GitHub issue is the completion ledger.

## 1. Establish the guardrails

Read every applicable `AGENTS.md`, from the repository root down into the directories the work touches. Re-check for nested instructions as the implementation surface becomes known.

Before changing code, identify from those instructions how to start or reach the app's live test environment and how to exercise its public interface. A production environment is not a test environment unless the user explicitly authorizes it. If no usable live-test procedure is documented, stop and report the missing guardrail.

Inspect the worktree and branch before claiming the work. Require a clean worktree, record the starting `HEAD` as the review fixed point, and confirm pushing this branch will not include unrelated unpushed commits. Stop and ask the user to resolve or explicitly include any pre-existing work.

## 2. Resolve and claim the work

- **An issue was supplied:** resolve its GitHub repository, fetch it with `gh`, and confirm its title and repository. If it already has any assignee, require the user's explicit green light for that assigned issue; invoking this skill alone is not that green light.
- **No work was supplied:** resolve the current GitHub repository from its Git remote, then use `gh` to list open issues carrying the `ready-for-agent` label. Exclude assigned issues and issues with open blockers, then choose the oldest eligible issue (lowest issue number). Stop if none is available.
- **The chosen issue is unassigned:** assign it to the authenticated GitHub user with `gh issue edit --add-assignee @me` before editing code, then fetch it again and verify the assignment succeeded.
- **A spec with no GitHub issue was supplied:** continue without GitHub issue actions.

**Read the ledger whole** — the body and every comment (`gh issue view <number> --comments`). The brief specifying the work, and the decisions that reshaped it, are often comments, so an issue read from its body alone can miss the contract entirely.

GitHub issue work requires an authenticated `gh` session and an unambiguous GitHub repository. If either is unavailable, stop before claiming or changing anything. Do not substitute another issue tracker.

## 3. Implement and check

Implement the agreed work without reopening its design. Use `/tdd` where possible at pre-agreed seams. Run typechecking and focused tests regularly, then run every repository-required validation and the full test suite.

All automated checks must pass before review.

## 4. Review and fix

Run `/code-review` against the complete change since the recorded starting `HEAD`, including committed, staged, unstaged, and untracked work. Supply the issue or spec explicitly so the Spec axis does not depend on commit messages.

Resolve every actionable finding: fix it or record why it does not apply. Rerun affected focused checks and every repository-required validation. No hard Standards finding or Spec finding may remain unresolved.

## 5. Run final live acceptance

After review fixes, start or reach the documented live test environment. Exercise the changed behaviour through the app's real public interface and verify every applicable acceptance criterion. Record the commands, paths, URLs, or other evidence used.

If live acceptance exposes a defect, fix it and return to automated checks and review before repeating live acceptance. The final live pass must exercise the reviewed code. A failed or skipped live pass blocks every remaining step.

## 6. Complete and ship

For GitHub issue work:

1. Check off each implementation or acceptance checkbox only when the live-pass evidence proves it complete. The checkboxes sit wherever the brief does, a comment as readily as the body; tick them in place there.
2. Fetch the issue again, comments included, and verify every required checkbox is checked. Any unchecked required item blocks the release.

Then inspect the final diff, commit only the intended work to the current branch, push that branch, and verify the remote contains the commit. Follow all repository-specific commit, push, deployment, and post-push validation gates in `AGENTS.md`.

Close the GitHub issue with `gh issue close` only after validation, checkbox completion, commit, push, and any required post-push checks have all succeeded. Fetch it once more and verify it is closed. On any failure, leave the issue open and report the exact blocking gate.
