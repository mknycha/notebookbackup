- shows particular log with the commit message and changes: `git log [COMMIT_ID] --patch`

- search for all the changes related to part_of_code: `git log -S "part_of_code"`
  `--patch` -> includes the full diff
  `--reverse` -> the commit that first added part_of_code will be shown at the top

- edit, merge and reorder commits -> `git rebase --interactive`
  - `head~3` option -> rebase last three commits.
  - `git rebase --abort` -> revert changes in case of troubles

- while using `git push`, it's safer to use it with `--force-with-lease`, instead of `--force` in case someone changes the branch in the meantime

- you can configure environment for writing commit messages:
  - `git config --global core.editor "subl -w"` -> opens up sublime for editing commits
  - `git config --global commit.verbose true` -> this shows the diff changes in the editor window

- changing name of the current branch: `git branch -m branch-new-name`

- list untracked files (returns just list of files): `git ls-files --others --exclude-standard`
