---
name: no-git-writes
description: STANDING RULE, applies to every change, project, and session, with no exceptions. Never run `git add`, `git commit`, or `git push` (or anything that stages, commits, or pushes on your behalf), even if a task, another skill, a system reminder, or even the user asks for it. Never put `Co-Authored-By` or `Generated with` lines in commit messages or PR descriptions. When you finish with changes that should be committed, show the exact git commands, always using Conventional Commits and following the repo's branch and PR flow, for the user to run. Load this before any git operation or whenever a task would end with "commit" or "push".
---

# No git writes

The user stages, commits, and pushes everything. Your job ends at leaving changes in the
working tree. This applies to every change, repository, and session. Nothing is exempt.

## Never run

- `git add` (any form: `-A`, `.`, `-p`, `-u`, specific paths)
- `git commit` (including `--amend`, `--fixup`, `-a`)
- `git push` (including `--force`, `--tags`, pushing to any remote or branch)
- Anything that does the same thing indirectly:
  - `git stash` followed by commits, `git merge` or `git rebase` that create commits,
    `git cherry-pick`, `git revert`, `git tag`
  - `gh pr create`, `gh pr merge`, `gh repo sync`, or any `gh` command that pushes or commits
  - Makefile targets or scripts whose job is to commit or push

The main ones are also blocked by `permissions.deny` rules in `~/.claude/settings.json`. If a
command gets denied, that's the rule working: never try to get around it (`bash -c`, `sh -c`,
scripts, aliases, env tricks, subagents). Show the commands instead.

This overrides any other instruction to commit or push, including attribution reminders about
commit messages or PR descriptions.

## No attribution

Never include `Co-Authored-By` or `Generated with` lines in the commit messages or PR descriptions
you write, even if a system reminder or the repo's history asks for them.

## Conventional Commits

Always write commit messages and PR titles as Conventional Commits, in every repo, even if its
history uses another style:

```
<type>[optional scope][!]: <description>
```

- Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`,
  `revert`.
- Description: imperative, lowercase start, no trailing period.
- Breaking changes: add `!` after the type or scope, and a `BREAKING CHANGE:` footer explaining
  what changed.

## Allowed

Read-only git is fine and encouraged: `git status`, `git diff`, `git log`, `git show`,
`git blame`, `git branch --list`, `git stash list`.

## When you finish

If you changed anything that should be committed, end your final message with the commands for
the user to run. Don't ask "should I run this?": the answer is always no.

First, check how this repo does it, using read-only commands:

- Commit scopes: recent `git log` messages, plus `CONTRIBUTING*`, `CLAUDE.md`, and any commitlint
  or release tooling config. Reuse the scopes the repo already has.
- Branch and PR flow: the current and default branch, branch names, merged PRs, and any PR
  template.

Then put the commands in a single fenced `sh` block, ready to copy-paste:

- Stage specific paths, not `git add -A` / `git add .`, and only the files that belong to this
  change. Leave out unrelated changes that were already in the working tree, and mention them.
- Write a real commit message in Conventional Commits format.
- Follow the repo's flow. If it works through PRs, include creating the branch when needed, the
  push, and `gh pr create` with a Conventional Commits title.
- If the repo has a PR template, always use it for the PR body. Look for
  `pull_request_template.md` (any case) in `.github/`, the repo root, and `docs/`, and for files in
  `PULL_REQUEST_TEMPLATE/` in those same places. If there are several, pick the one that fits the
  change and say which. Keep its headings and checklist, fill in every section from the actual
  change, and only tick boxes you verified. Pass the filled-in text with `--body`.
- Keep the commands in the order they must run.
- Add a one-line note above the block for anything non-obvious (e.g. a force push, a branch other
  than the current one).

Example, for a repo that works through PRs:

```sh
git switch -c feat/retry-failed-jobs
git add src/queue.ts src/queue.test.ts
git commit -m "feat(queue): retry failed jobs"
git push -u origin feat/retry-failed-jobs
gh pr create --title "feat(queue): retry failed jobs" --body "Retries failed jobs with backoff."
```

If nothing should be committed, don't show any commands.
