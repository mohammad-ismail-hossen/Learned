# Git

## Index

1. [Local Repository](#1-local-repository)
2. [Tracking Changes](#2-tracking-changes)
3. [Git Branching](#3-branching-in-git)
4. [Remote Repository](#4-remote-repository)
5. [.gitignore](#5-gitignore)
6. [Reverting or Resetting Changes](#6-reverting-or-resetting)
7. [Git Log & History](#7-git-log--history)
8. [Conventional Commits](#8-conventional-commits)
9. [Merge Conflicts](#9-merge-conflicts)
10. [Useful One-Liners & Aliases](#10-useful-one-liners--aliases)
11. [Advanced Features](#11-advanced-features---diff---stash---rebase)
12. [Tags & Releases](#12-tags--releases)
13. [Detached HEAD](#13-detached-head)
14. [Force Push Recovery](#14-force-push-recovery)


<details>
  <summary><strong>Resources</strong></summary>

1. [GitHub tutorial by freeCodeCamp](https://youtu.be/mAFoROnOfHs?si=tp2vgE0ygFhb9Zp7)
2. [Official Git Cheatsheet](https://git-scm.com/cheat-sheet)

</details>

<details>
  <summary><strong>Notes</strong></summary>

1. Key terms:
    - **HEAD** - Points to the last commit on your current branch.
    - **Working Directory** - What's currently in your file system / editor, including unsaved or unstaged edits.
    - **Staging Area (Index)** - A middle zone between your editor and a commit.
    - **Origin** - The default name for a remote repository.
    - **Tracked file** - A file Git is aware of (was added or committed at least once).
    - **Upstream** - The remote branch your local branch is linked to, used by `push` and `pull` by default.

2. Most Git commands act on your **current branch** by default.

</details>

---

## 1. Local Repository

To set up a local Git repository we need to configure identity before making commits.

- **Set up Git identity** - Git tags every commit with your name and email. Set this once globally.
    ```bash
    git config --global user.name "Your Name"
    git config --global user.email "you@example.com"
    ```

- **Initialize a new repository** in your project folder.
    ```bash
    git init
    ```

- **Check repository status** - shows what's staged, unstaged, and untracked.
    ```bash
    git status
    ```

---

## 2. Tracking changes

Once a repo is initialized, we track changes in a three-step cycle: edit, stage, commit.

- **Stage files** - Move changes into the staging area.
    ```bash
    git add <file>        
    # stage a specific file

    git add .
    # stage everything in current directory and subdirectories

    git add --all
    # stage all changes including deletions, anywhere in the repo
    ```

- **Unstage files** - Remove from staging area without discarding edits. (the files needs to be a tracked file)
    ```bash
    git restore --staged <file>
    # unstage a specific file

    git restore --staged . 
    # unstage everything
    ```
> Tip - If you hadn't commited yet, it's your first add, use "git reset &lt;file&gt;"

- **Commit changes** - Save a snapshot of the staged files to history.
    ```bash
    git commit -m "tittle" -m "description"
    # commit with title and description.

    git commit -am "message"
    # stage and commit in one step
    # only stages tracked file

    git commit
    # to use a editor to write title and desctiption.
    ```

> **Tip - Write good commit messages.** Use the imperative form: `"Add login route"` not `"Added login route"`. One line for small changes; use a body for complex ones.

- **Untrack files**
    ```bash
    git rm --cached <file> 
    # remove <file> from tracking file list.
    ```
---

## 3. Branching in Git

Branches are just different time lines of the same repo. It let's you work on features, or experiments in isolation from the main codebase. In real teams, working directly on `main` is almost always off-limits.

- **List branch**:
    ```bash
    git branch
    # list all local branches
    git branch -a
    # list local + remote-tracking branches
    ```

- **Switch to branch**:
    ```bash
    git switch <branch-name>
    # switch to a branch
    ```

- **Create a branch:**
    ```bash
    git branch <branch-name> 
    # create a new branch

    git switch -c <branch-name>   
    # create and switch

    git switch -c <branch> origin/<Branch> 
    # create a local <branch> from remote <Branch>

    git switch -c <new-branch> <commit-hash>
    # create a local <branch> from a specific commit
    ```

- **Remote branch**
    ```bash
    git pull origin <branch>    
    # pulls <branch> from remote to local
    ```

- **Merge a branch into the current branch:**
    ```bash
    git merge <branch> 
    # merges <branch> into your current branch
    git merge --no-ff <branch>
    # force a merge commit even if fast-forward is possible. 
    # fast-forward: add commits in linear, if no new commit is in between.
    ```

- **Deleting branch**:
    ```bash
    git branch -d <branch-name>   
    # delete a branch
    git branch -D <branch-name>   
    # force delete (even if unmerged)
    ```

- **Detached HEAD mode:**
    ```bash
    git switch --detach <commit-hash>
    # explore history without being on a branch
    ```

> **Typical branch naming conventions in teams:**
> - `feat/add-login`
> - `fix/broken-redirect`
> - `chore/update-dependencies`
> - `release/v1.2.0`
> - `refactor/simplify-logic`

---

## 4. Remote Repository

Connect your local repo to a hosted remote service (GitHub, GitLab) to back up things, collaborate, or trigger CI/CD.

- **Connect local repo to remote:**
    ```bash
    git remote add origin <repo-url>
    git remote -v
    # see remote status
    ```

- **Clone remote repo to local:**
    ```bash
    git clone <repo-url>
    # downloads repo into a new folder
    git clone <repo-url> <folder-name>   
    # downloads into a specific folder name
    ```

- **Push changes:**
    ```bash
    git push -u origin <branch>   
    # push + set upstream as origin (-u)
    git push
    # push to set upstream

    git push origin <local-branch-name>
    # push specific local branch to origin
    git push --all        
    # push all local branches to remote

    git push --force-with-lease
    # force push safely
    # fails if remote has new commits
    # safer than --force, use when rebasing a shared branch
    ```

- **Fetch** (update the remote-tracking branches, origin/):
    ```bash
    git fetch origin
    # update remote-tracking refs (e.g. origin/main)
    ```

- **Pull**(fetch + merge or rebase):
    ```bash
    git pull
    # pull from tracked upstream
    git pull origin <branch>
    # pull specific remote branch
    
    git pull --rebase
    # pull and rebase instead of merge 
    ```

---

## 5. .gitignore

`.gitignore` tells Git which files not to track.

Create a file named `.gitignore`. Use * as pointer, x/ for folders and .x for extension.

Example:
```
node_modules/
.env
*.pem

# now git will not track, node_modules folder;
# ".env" file;
# any file ends with .pem extension
```

```bash
git rm --cached <file>
# stops tracking file that was already committed 
# use it after adding it to .gitignore
```
---

## 6. Reverting or Resetting
`Restore`: resets working/staging to staging/HEAD

`Reset`: move HEAD pointer

`Revert`: new commit for undoing a commit.

### Restore: Used for undoing local edits
```bash
git restore <file>
git restore .
git restore --worktree <file>
# discard unstaged changes
# working dir resets to staging area

git restore --staged <file> 
# unstage a staged file
# staging area resets to HEAD

git restore --staged --worktree <file>  
# undo both staged and unstaged changes
# staging and working area resets to HEAD
```

### Reset: Used for moving HEAD
```bash
git reset <file>
# unstage a file 
# resets staging to head (same as restore --staged)

git reset <commit>
git reset --mixed <commit>
# move HEAD to <commit>
# clear staging area, keep working dir

git reset --soft <commit>
# move HEAD to <commit>
# keep staging and working dir
  
git reset --hard <commit>   
# move HEAD to <commit>
# clear staging and working dir

git reset --hard    
# nuke all uncommitted changes :)
# resets to HEAD, clear staging and working dir.
```

### Revert: Undo the changes of a commit by a new commit
```bash
git revert <commit>     
# Creates a new commit that undoes the changes made in <commit>
```

**Usecases:**

| Situation | Command |
|---|---|
| Undo unsaved local edits | `git restore` or `git restoere --worktree` |
| Unstage something | `git restore --staged` or `git reset` |
| Undo a commit that's already on remote | `git revert` |

---

## 7. Git Log & History:

Logs show, history top to bottom

```bash
git log  
# full commit history (author, date, message)

git log --oneline   
# compact view, one commit per line
git log --oneline --graph   
# shows simplified graph of logs, useful for merges
git log --oneline --all     
# include all branches including remotes

git log --author="name"     
# filter commits by author
git log -- <file>           
# filter commit by file affected

git shortlog    
# show summary of commits grouped by author

git show <commit-hash>
# full details of one specific commit
```

> **Tip:** `git log --oneline --graph --all` is popular for checking repo state. Consider aliasing it. [(see section 10)](#10-useful-one-liners--aliases)

---

## 8. Conventional Commits

Conventional Commits are structured way to write commit messages. CI/CD tools, changelogs, and release pipelines use this.

**Format:**
```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```
>**Note**: *For short description use imperative form.*

**Common types:**

| Type | When to use |
|---|---|
| `feat` | A new feature |
| `fix` | A bug fix |
| `chore` | Maintenance, dependencies, config (no production code change) |
| `docs` | Documentation only |
| `refactor` | Code restructure with no behavior change |
| `test` | Adding or fixing tests |
| `ci` | CI/CD pipeline changes |
| `perf` | Performance improvement |

Examples:
```
feat(auth): add login endpoint
fix(api): handle null response from payment service
chore: update dependencies to latest versions
docs: add setup instructions to README
ci: add Docker build step to GitHub Actions
```

**Breaking changes**:

 add `!` after the type or a `BREAKING CHANGE:` footer:
```
feat!: remove deprecated /v1 API endpoints

BREAKING CHANGE: /v1 routes have been removed. Migrate to /v2.
```

---

## 9. Merge Conflicts

A merge conflict happens when two branches edit the same part. Git can't automatically decide which version to keep.

### Scenarios:
```bash
- git merge <branch>
- git pull # pull is fetch + merge
- git rebase
- git cherry-pick # pick a specific commit from a branch
```

### What a conflict looks like in the file
```
<<<<<<< HEAD
  const port = 3000;
=======
  const port = 8080;
>>>>>>> feature/update-port
```
- Everything between `<<<<<<< HEAD` and `=======` is current branch's version
- Everything between `=======` and `>>>>>>>` is incoming branch version

### Resolving:

1. Open the conflicted file
2. Delete the conflict markers (`<<<<<<<`, `=======`, `>>>>>>>`) and all unwanted code
3. Stage and merge the resolved file:
    ```bash
    git add <file> # stage change
    
    git commit  # finish the merge
    Or,
    git rebase --continue   # if conflict during rebase
    ```

### Aborting:
```bash
git merge --abort
# cancel the in-progress merge
# restore to pre-merge state

git rebase --abort
# cancel an in-progress rebase
```

### Notes for conflict resolution:
```bash
git status
# during conflict, shows which files are conflicted

git diff
# shows remaining conflicts

git log --merge
# shows commits from both branches that caused the conflict
```
---

## 10. Useful One-Liners & Aliases

**Helpful one-liners:**
```bash
git log --oneline --graph --all 
# visual overview of all branches

git diff --stat HEAD
# summary of changed files in compact view
# working dir vs head

git branch --merged
# branches already merged into current
git branch --no-merged
# branches not yet merged
```

**Set up aliases**: making shortcuts for long commands.
```bash
git config --global alias.x <cmd>
# the git <cmd> would be alias as x 
# we can use it as git x
```

Examples:

```bash
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```
```bash
git st
# git status
git lg
# git log --oneline --graph --all
```

To see and edit all global variable:
```bash
git config --global --edit
```

---

## 11. Advanced Features - Diff - Stash - Rebase

### Diff

A file can exist in three areas: HEAD, the staging area and working directory. `git diff` compares any two of these.

git diff compares file like, how to turn this to this with + for additions and - for subtractions.

```bash
git diff
# working dir vs staging area
git diff --staged           
# staging area vs HEAD
git diff HEAD
# working dir vs HEAD

git diff left..right
# left vs right, how to turn left to right
# any two refs: branches, commits, tags
git diff HEAD..origin/main  
# how to turn HEAD to origin/main
```
>*Note: diff by default targets the next stage(working dir vs staging, staging vs Head)*

### Stashing

Stashing saves working dir + staging area and resets everything to HEAD. Only saves tracked files.

```bash
git stash
# save working dir + staging (tracked files)
# resets everything to HEAD
git stash list
# see all saved stashes

git stash apply
# apply the latest stash
git stash apply stash@{n}
# apply the nth stash

git stash pop
# apply and delete latest stash from list
git stash pop stash@{n}
# apply and delele nth stash from list

git stash drop stash@{n}  
# delete a specific stash entry
```

> **Note:** `git stash -u` to include untracked files.

### Rebasing

Rebase replays commits on top of another branch. Preferred over merge in many teams.
```
Before:
  main:    A - B - C
  feature: A - B - D - E

After: git rebase main (while on feature branch):
  
  feature: A - B - C - D - E
```

```bash
git rebase <branch>
# replay current branch's commits on top of <branch>'s commits

git rebase -i < >
# interactive rebase: edit, squash, reorder, or drop commits

git pull --rebase
# fetch then rebase instead of merge
```

> Don't rebase commits that have already been pushed to a shared branch. It rewrites history. Use case, self-local.

### Professional Pattern

```bash
git fetch
git diff HEAD..origin/main     
# to see what changed on remote
git stash
# stash local work
git pull --rebase
# pull cleanly without a merge commit
git stash pop
# Re-apply local work on top
```

### Removing files

```bash
git clean -fd
# deletes all untracked files

git rm --cached
# removes file from tracking list
```

---

## 12. Tags & Releases

Tags mark specific commits, usually for version releases.

```bash
git tag
# list all tags

git tag v1.0.0
# create a tag on current commit

git tag -a v1.0.0 -m "Release"
# annotate tag

git push origin v1.0.0
# push a specific tag to remote
git push origin --tags
# push all tags to remote

git tag -d v1.0.0
# delete a local tag
git push origin --delete <tag-name> 
# delete a remote tag
```

---

## 13. Detached HEAD

A Git detached HEAD state means you are not on any branch. Instead, your HEAD points to an old commit.

Won't save any commits, until you make new branch from it. Used for old commit exploration or experimentation. 

Entering:

```bash
git switch --detach <commit-hash>
# detached head state to <commit-hash>
```

While in it:

```bash
git switch <branch>
 # go back to <branch> 
 # experimental commits will be discarded eventually
```
Or,
```bash
git switch -c <new-branch-name>
# create a branch from current position 
# commits will be saved
```

---

## 14. Force Push Recovery

Force pushing (`git push --force`) can rewrite remote history. Here's how to recover from common force push disasters.

### Precautions:
```bash
git push --force-with-lease   
# fails if someone else pushed since last fetch
# recommended using this instead of --force whenever possible
```

### Scenario 1 -  force pushed and now to undo it

```bash
git reflog 
# shows every movement of HEAD 
# find commit remote was on before the force push

git push origin <commit-hash>:main --force-with-lease
# restore remote to that commit
```

### Scenario 2 - remote is force pushed and now local branch is behind

```bash
git fetch origin

git log HEAD..origin/main
# commit difference of local and remote HEAD
git log origin/main..HEAD
# commit difference of remote and local HEAD
```
```bash
# Accept the remote version
git reset --hard origin/main

# Recover your commits and re-apply them
git rebase origin/main
```

### Scenario 3 - lost commits from reset --hard

```bash
git reflog 
# shows every movement of HEAD 
# find commit remote was on before the force push

git switch -c recovery-branch <commit-hash>
# create branch at that commit to recover
```

> Reflog is local only

### Contingency Plans:

| Situation | Fix | Reason |
|---|---|---|
| Committed to wrong branch | `git reset --soft HEAD~1` <br> `git stash` <br> `git switch <branch>` <br> `git stash pop` <br> `git commit` | Staging area is local bound not branch bound |
| Committed sensitive file | `git reset --soft HEAD~1` <br> `git rm --cached <file>` <br> `add to .gitignore` <br> `git commit -m "fix"` | git rm --cached <file> removes the file from Git's tracking system |
| Accidentally deleted a branch | `git reflog` <br>  `git switch -c <branch> <hash>` | find previous commit, when the branch existed, create branch with that hash |
| Merge gone wrong | `git merge --abort` <br> or <br> `git reset --hard ORIG_HEAD` | Abort the merge or resets to ORIG_HEAD (acts as an automatic safety backup pointer)|

---