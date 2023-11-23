# Git FAQ

## GitHub Cheat Sheet

[GitHub Cheat Sheet](https://github.com/tiimgreen/github-cheat-sheet)

esp. usefull information about hidden features of GitHub

## Squash bunch of commits

"Melting" some commits into one.

```
$ git log

* (5) ... HEAD
* (4) ...
* (3) ...
* (2) ...
* (1) ...
```

- squash `(5)` to `(3)` into `(3)`: `git rebase -i HEAD~3` (`HEAD~3 == (2)` which is
  one behind the `(3)`), this will open an editor,

  ```gitcommit
  pick (3) ...
  squash (4) ...
  squash (5) ...
  ```

  - the oldest commit that all commit eventually "melt" into it should be `pick`.
  - save it and again editor will open. write your message and commit.
  - to squash all the way to the root: `git rebase -i --root`

- squash `(4)`, `(3)`, and `(2)` into commit that replace `(2)`:

  ```
  $ git rebase -i HEAD~4

  pick (2) ...
  squash (3) ...
  squash (4) ...
  pick (5) ...
  ```

## Change previous commit messages

- amending
  - `git commit --amend`
- reword
  - (`git rebase -i HEAD~<one before the commit you want to remove>`)
  - then mark the commit you want to reword as `reword`.
  - save and quit. in the new editor window change your commit messages.
  - don't do this if you've already pushed your commits to remote, for others to use.


## Add new changes to the last commit without editing the message

```sh
git add <changes>
git commit --amend --no-edit
```

## Fixup commits

first you have to incorporate the changes you wish you had previously in the old
commit in a "fixup" commit and then interactively rebase it.

```
git add <changes>
git commit --fixup=<old commit hash>
git rebase -i --autosquash HEAD~<one before the old commit>
# save and quit. done
```

`--autosquash` means fixup commits will be automatically squashed into the
commits they reference ([ref]()).

You can have this behaviour by default, which seems safe and sensible, by setting
autosquash = true in your ~/.gitconfig.

```gitconfig
[rebase]
        autosquash = true
```

I use --fixup so much that I have a helper alias in my ~/.gitconfig.

```gitconfig
[alias]
        fixup = "!git log -n 50 --pretty=format:'%h %s' --no-merges | fzf | cut -c -7 | xargs -o git commit --fixup"
```

This lets me type git fixup and presents a list of my 50 most recent commits and
allows me to search the list using fzf. Once a commit is selected, the SHA is
passed to git commit `--fixup`
([ref](https://jordanelver.co.uk/blog/2020/06/04/fixing-commits-with-git-commit-fixup-and-git-rebase-autosquash/)).


## Unstage with `restore` and `rm --cached`

- unstage before the first commit (No `HEAD`)

  ```sh
  git rm --cached <file>
  ```

- unstage after the first commit

  - undo/unstage changes to be committed

  ```sh
  git restore --staged <file>

  # only equivalent with the following if the file is newly added (previously
  # untracked)
  git reset --cached <file>
  ```
  this will replace the content of index with `HEAD`. that means newly added file
  will be untracked. you can pass (unless you pass `--source=<hash>` to bring
  content of a certain commit)

  - undo changes not staged for commit (discard changes in the working tree)

  ```sh
  git restore <file>
  ```
  this brings the content of index into working tree. note that you'll lose
  the content of working tree for ever for that particular file.


## Find a commit that introduced a string

```sh
# quote search string if it contains white space
git log -S needle --source --all
git log -S 'hello world' --source --all
git log -S "systray" --source --all --reverse

# Search with regex
git log -G "^(\s)*function foo[(][)](\s)*{$" --source --all
```

The `--all` parameter means to start from every branch and `--source` means to show
which of those branches led to finding that commit. It's often useful to add `-p`
to show the patches that each of those commits would introduce as well.
[ref](https://stackoverflow.com/a/5816177/13041067)

`--reverse` is also helpful since you want the first commit that made the change.
[ref](https://stackoverflow.com/a/31621921/13041067)


## Change the author of commits

to change the author of the most recent commit

```sh
git commit --amend --author "<New Author Name> <email@address.com>"
```

to change the aurhor of a commit other than the most recent commit

```sh
git rebase -i --rebase-merges <some HEAD before all of your bad commits>

# mark desired commit as "edit"/"e"; save and exit

git commit --amend --author "<New Author Name> <email@address.com>" --no-edit

git rebase --continue
```

[ref](https://stackoverflow.com/a/74856838/13041067)

## Git internal resources

Docs:

- [Git book -- Ch.10. Git Internals](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)

Videos:

- [Advanced Git: Graphs, Hashes, and Compression, Oh My!](https://www.youtube.com/watch?v=ig5E8CcdM9g)
- [OSCON 2016: Dissecting Git's Guts - Git Internals - Emily Xie](https://www.youtube.com/watch?v=YUCwr1Y6bFI)
- [Git From the Bits Up](https://www.youtube.com/watch?v=MYP56QJpDr4)
- [Git Internals - How Git Works - Fear Not The SHA!](https://www.youtube.com/watch?v=P6jD966jzlk)
- [Andrey Syschikov - A journey into Git internals with Python](https://www.youtube.com/watch?v=bZ4WbPnNPCs)

## Git add and restore (--patch)

Docs:

- [The Art of Manually Editing Hunks](https://kennyballou.com/blog/2015/10/art-manually-edit-hunks/index.html)

Common commands:

```sh
git add --patch                                                        # from: index, to: working-directory
git restore --patch                                                    # from: index, to: working-directory
git restore --patch --staged                                           # from: HEAD,  to: index
git restore --patch --staged [-s <tree>, --source=<tree>]              # from: tree,  to: index
git restore --patch --staged --worktree [-s <tree>, --source=<tree>]   # from: tree,  to: index and working-directory
```

Hunk identifiers:

```
@@ -line number[,context] +line number[,context] @@
```

- `-`: source/from file
- `+`: new/to file
- `context` is the number of lines in the hunk for each file
  - can be different sometimes

diff syntax:

- `+`: denotes a line that will be added to the first file
- `-`: deontes a line that will be removed from the first file
- ` `: not changed
- Note that to diff, updating a line is the same as removing the original line
  and adding a new line (with the changes).

usefull operations:

- `?`: print help
- `q`: quit; do not stage this hunk or any of the remaining ones

![Comparison of git-add, and git-restore [ref](https://stackoverflow.com/a/77143766/13041067)](./img/git-add-restore.png)

Note for editing patches in:

- `git add`: adds changes in working-tree to index
  - the `+` lines will be added to the index after the command.
    - thus removing them means we will not add them (they are still in working tree)
  - the `-` lines will be removed from index after the command.
    - thus replacing them with ` ` (i.e. making the same as index), they will stay
      in index, (although removed from working tree)
- `git restore`: restores changes in working tree from index
  - the `+` lines will be restored from index,
    - thus replacing them with ` ` means they are the same as index, and no need
      to revert the change in working tree
  - the `-` lines will be restored from index,
    - thus deleting them means the index is the same as working tree and no need
    to revert the change in working tree.

- `git restore --staged`: just like above but restores changes in index from HEAD

- `git restore --staged --worktree`: Not sure how to correctly edit the patch

Tip: ([ref](https://www.youtube.com/watch?v=UJ5fpaeZWsI):): A
good companion to "`git add -p`" is "`git stash --keep-index`", so you can
preserve your index but stash the remaining changes, so you can run your tests
(for example), just before committing. After doing your partial commit, just "`git
stash pop`", and repeat.
  - note that it is also easy to lose your changes with git-stash
