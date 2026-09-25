# homelab-workspaces

XR instance files for every workspace app on the [homelab](https://github.com/cujarrett/homelab) cluster. Directory layout, the `namespace.yaml` requirement and the `guest-*` sandbox rules are in the [README](./README.md).

## Rules

- **Never run `git add`, `git commit`, `git push`, or any git command that writes to or modifies the index, repository history, or remotes.** Output the commands for the user to run. Staging is part of their review.
- **Never add a `Co-Authored-By` trailer or a "Generated with Claude Code" line** to commit messages or PR descriptions, including in suggested commit messages. Commits are authored by the user alone.
- **Whenever a task requires a commit, always give a suggested commit message.** Give `git add` and the commit as two separate steps, listing every file explicitly. Never output a `git push` command.
- **Never hand-edit a `guest-*` directory.** `launchpad-api` owns them.
- **Cheapest rung that works.** Before writing code go down the ladder and stop at the first rung that solves it: skip the feature, reuse code already here, standard library, native platform feature, a dependency already installed, one line, then build the minimum.

### Pre-commit safety check

Before telling the user to commit, always run `/security-review`. Once it confirms the changes are safe, offer a suggested commit message.
