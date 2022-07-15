---
layout: default
---

# Git Cheatsheet

## git merge upstream
```
git fetch upstream
git checkout main
git merge upstream/main
```
ref: [git fork and upstream tips](https://www.atlassian.com/git/tutorials/git-forks-and-upstreams)

## git check remote log
```
git log remotename/branchname
```

## git stash
```
git stash
git stash pop
git stash pop stash@{1}
```

## Undo a commit & redo
```
$ git commit -m "Something terribly misguided" # (0: Your Accident)
$ git reset HEAD~                              # (1)
[ edit files as necessary ]                    # (2)
$ git add .                                    # (3)
$ git commit -c ORIG_HEAD                      # (4)
```
ref: [how to undo the most recent local commit](https://stackoverflow.com/questions/927358/how-do-i-undo-the-most-recent-local-commits-in-git)
## To see the previous git action
```
git reflog
git reset --hard HEAD@{2}  # go back to a specific history point
```

ref: [undoing a git rebase](https://stackoverflow.com/questions/134882/undoing-a-git-rebase)

[back](.././)