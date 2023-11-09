# Git FAQ

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
