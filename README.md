# no-git-writes

Claude Code never stages, commits, or pushes. When a change needs one of those, Claude shows the
exact commands in a copy-paste block and you run them yourself.

This applies to every project on this machine. It has two layers:

| Layer | File | What it does |
|---|---|---|
| Instruction | `~/.claude/skills/no-git-writes/SKILL.md` | Tells Claude the rule and how to hand the commands over instead. |
| Enforcement | `~/.claude/settings.json` → `permissions.deny` | Claude Code blocks the commands before they run, whatever Claude decides. |

The skill alone is only a request: nothing technically stops Claude from ignoring it. The deny
rules are the guarantee. The skill is still useful because it makes Claude show the commands
instead of trying to run them, hitting a denial, and then trying something else.

## The skill

Claude always sees the skill's one-line description in its skill listing, and the rule is written
into that description, so it's in effect without being invoked. The full `SKILL.md` covers:

- **Never run:** `git add`, `git commit`, `git push`, plus indirect routes (`cherry-pick`,
  `revert`, `tag`, merges or rebases that create commits, `gh pr create`, `gh repo sync`, scripts
  or Makefile targets that commit or push).
- **No attribution:** commit messages and PR descriptions never include `Co-Authored-By` or
  `Generated with` lines.
- **Allowed:** read-only git (`status`, `diff`, `log`, `show`, `blame`, `branch --list`,
  `stash list`).
- **When Claude finishes:** if something should be committed, Claude checks the repo's commit and
  PR conventions and ends with the commands in one `sh` block. The block stages specific paths
  (never `git add -A` / `git add .`), always uses Conventional Commits,
  follows the repo's branch or PR flow, fills in the repo's PR template when it has one, keeps the
  commands in order, and adds a one-line note above anything risky such as a force push.
- **No exceptions:** the rule covers every change and project. Claude shows the commands even when
  you ask it to commit, and it doesn't ask "should I run this?".
- **Priority:** the rule wins over other instructions, including Claude Code's built-in commit
  attribution reminders.
- **Denials:** if a command is denied, Claude must not work around it (`bash -c`, `sh -c`,
  scripts, aliases, env tricks).

## The deny rules

In `~/.claude/settings.json`:

```json
"permissions": {
  "deny": [
    "Bash(git add *)",
    "Bash(git * add *)",
    "Bash(git commit *)",
    "Bash(git * commit *)",
    "Bash(git push *)",
    "Bash(git * push *)",
    "Bash(git cherry-pick *)",
    "Bash(git revert *)",
    "Bash(git merge *)",
    "Bash(git rebase *)",
    "Bash(git am *)",
    "Bash(gh pr create *)",
    "Bash(gh pr merge *)",
    "Bash(gh repo sync *)"
  ]
}
```

What they cover:

- **`git add` / `commit` / `push`**, with or without arguments.
- **Options before the subcommand**, via the `git * <subcommand> *` rules: `git -C fork commit`,
  `git -c user.name=x commit`, and so on.
- **Commands that create or rewrite commits:** `cherry-pick`, `revert`, `merge`, `rebase`, `am`.
- **GitHub CLI** commands that open or merge PRs, or push to a fork.
- **Chained commands:** Claude Code checks each part of `a && b`, `a; b` and `a | b`
  separately, so `cd fork && git commit -m x` is blocked too.

Read-only git commands are not matched and still run normally.

### Side effects

The `git * add *` / `git * commit *` / `git * push *` rules match those words anywhere after
`git`, so a few unrelated commands are blocked too:

- `git remote add`, `git submodule add`, `git worktree add`
- `git stash push`
- read-only commands with one of those words as a bare argument, e.g. `git log --grep add`
  (quoting it as `--grep="add"` avoids the match)

Blocking these is intended: catching `git -C <dir> commit` is worth it.

### Not covered

- **`git tag`** isn't blocked, since that would also block listing tags. The skill still forbids
  creating tags.
- **Commands hidden inside something else:** a script, a Makefile target, `bash -c "git commit"`,
  a git alias. Deny rules only see the command line Claude runs. The skill forbids these
  workarounds, but only the skill.
- **Plumbing commands** like `git commit-tree` and `git update-ref`.
- **Your own commands.** The rules only apply to Claude's tool calls. Anything you run in your
  terminal, or with `!` in the Claude Code prompt, is unaffected.

## Checking and changing it

- **See the active rules:** `/permissions` in Claude Code.
- **Allow one of the commands again:** remove its line from `permissions.deny` in
  `~/.claude/settings.json`. A deny rule beats any allow rule, so adding an allow isn't enough.
- **Turn the whole thing off:** delete the `permissions.deny` entries and the
  `~/.claude/skills/no-git-writes/` directory.
- **Keep the rules consistent:** if you edit the rules, update the "Never run" list in `SKILL.md`
  to match, and vice versa.
