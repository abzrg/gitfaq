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
