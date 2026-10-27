# Git & GitHub Master Notes

### A practical reference for daily development, collaboration, recovery, and CI/CD

> **Purpose:** A durable, task-oriented Git handbook. Commands are
> grouped by what state your repository is in and what you are trying to
> accomplish. Examples use placeholders such as `<branch>` and
> `<remote>`---replace them with your actual names.
>
> **Important safety rule:** Before a risky operation, run `git status`
> and inspect the history. Commands marked **DANGER** can discard work,
> rewrite shared history, or remove files. Prefer reversible approaches
> when possible.

------------------------------------------------------------------------

## Table of contents

1.  [Mental model and fundamentals](#1-mental-model-and-fundamentals)
2.  [Install and configure Git](#2-install-and-configure-git)
3.  [Repository state: choose the right
    command](#3-repository-state-choose-the-right-command)
4.  [Starting a repository](#4-starting-a-repository)
5.  [Inspecting changes and history](#5-inspecting-changes-and-history)
6.  [Staging and committing](#6-staging-and-committing)
7.  [Branches](#7-branches)
8.  [Merging and resolving
    conflicts](#8-merging-and-resolving-conflicts)
9.  [Remotes, fetch, pull, and push](#9-remotes-fetch-pull-and-push)
10. [Undoing, resetting, and
    recovering](#10-undoing-resetting-and-recovering)
11. [Stash and selective changes](#11-stash-and-selective-changes)
12. [Cherry-pick, rebase, and history
    cleanup](#12-cherry-pick-rebase-and-history-cleanup)
13. [Tags and releases](#13-tags-and-releases)
14. [Files, ignore rules, and
    maintenance](#14-files-ignore-rules-and-maintenance)
15. [GitHub collaboration workflows](#15-github-collaboration-workflows)
16. [Authentication and permissions](#16-authentication-and-permissions)
17. [Advanced Git: worktrees, submodules, subtrees, LFS,
    hooks](#17-advanced-git-worktrees-submodules-subtrees-lfs-hooks)
18. [GitHub CLI (`gh`)](#18-github-cli-gh)
19. [Monorepos](#19-monorepos)
20. [GitHub Actions and deployment](#20-github-actions-and-deployment)
21. [Troubleshooting handbook](#21-troubleshooting-handbook)
22. [Common end-to-end workflows](#22-common-end-to-end-workflows)
23. [Interview and revision notes](#23-interview-and-revision-notes)
24. [Master command index and cheat
    sheet](#24-master-command-index-and-cheat-sheet)

------------------------------------------------------------------------

# 1. Mental model and fundamentals

## 1.1 Git vs GitHub

-   **Git** is a distributed version-control system that tracks project
    history on your computer.
-   **GitHub** is a hosting and collaboration platform built around Git.
    It adds pull requests, issues, code review, permissions, Actions,
    releases, and project management.
-   Git can be used without GitHub. GitHub repositories are Git
    repositories with additional services.

## 1.2 The three main local areas

``` text
 Working directory          Staging area                 Repository
 (your files)                (index)                     (commits)
┌─────────────────┐       ┌─────────────────┐        ┌─────────────────┐
│ edit app.py     │ add   │ selected diff   │ commit │ snapshot +      │
│ edit README.md  ├──────►│ ready to commit ├───────►│ parent + message│
└─────────────────┘       └─────────────────┘        └─────────────────┘
       ▲                         │
       └──── restore/reset ──────┘
```

-   **Working directory:** files you currently see and edit.
-   **Staging area/index:** the exact content that will be included in
    the next commit.
-   **Repository/object database:** committed snapshots and related Git
    objects.
-   **HEAD:** usually points to the current branch, which points to a
    commit. In detached-HEAD state, HEAD points directly to a commit.

Useful first commands:

``` bash
git status
git diff
git diff --staged
git log --oneline --graph --decorate --all
```

## 1.3 A commit is a snapshot, not simply a line-by-line patch

A commit records a project tree, metadata, message, and parent
commit(s). Git stores objects identified by hashes. A branch is a
movable name pointing to a commit; a tag is a named reference commonly
used for a release.

``` text
A---B---C  main
     \
      D---E  feature/search
```

Here, `main` points to C and `feature/search` points to E. Both share
history through B.

## 1.4 The everyday lifecycle

``` text
Edit files → git status → git diff → git add → git diff --staged
    → git commit → git fetch/pull as needed → git push
```

A common safe loop:

``` bash
git status
git diff
git add <file>
git diff --staged
git commit -m "Describe the change"
git push
```

## 1.5 Core vocabulary

  -----------------------------------------------------------------------
  Term                                Meaning
  ----------------------------------- -----------------------------------
  Working tree                        Checked-out files on disk

  Index / staging area                Content selected for the next
                                      commit

  Commit                              Saved project snapshot with
                                      parent/history

  HEAD                                Current checkout reference

  Branch                              Movable pointer to a line of
                                      development

  Remote                              Named connection to another
                                      repository

  `origin`                            Conventional name for the default
                                      remote; not a special keyword

  Upstream/tracking branch            Remote branch a local branch tracks

  Fetch                               Download remote objects and update
                                      remote-tracking refs without
                                      integrating them

  Pull                                Fetch plus integration (merge or
                                      rebase, depending on
                                      configuration/options)

  Push                                Send local objects/refs to a remote

  Merge                               Integrate histories while
                                      preserving branch structure

  Rebase                              Replay commits onto another base,
                                      creating new commit IDs

  Conflict                            Git cannot automatically decide how
                                      overlapping changes should combine

  PR (pull request)                   GitHub collaboration/review
                                      proposal; not a core Git command
  -----------------------------------------------------------------------

------------------------------------------------------------------------

# 2. Install and configure Git

## 2.1 Verify installation

``` bash
git --version
git help
git help status
```

-   Windows: install Git for Windows; use Git Bash or PowerShell.
-   macOS: install Xcode Command Line Tools or Git via a package
    manager.
-   Linux: install Git using the distribution's package manager.

## 2.2 Set identity

``` bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

This identity is written into new commits. It is not the same thing as
authenticating to GitHub.

Inspect configuration:

``` bash
git config --list
git config --show-origin --list
git config user.name
git config user.email
```

Set configuration at different scopes:

``` bash
git config --system <key> <value>   # system-wide; may require admin
git config --global <key> <value>   # your user account
git config --local <key> <value>    # current repository only
```

The usual precedence is local \> global \> system.

## 2.3 Helpful defaults

``` bash
git config --global init.defaultBranch main
git config --global pull.ff only
git config --global fetch.prune true
git config --global diff.colorMoved zebra
```

Notes: - `init.defaultBranch main` makes newly initialized repositories
start on `main`. - `pull.ff only` makes `git pull` refuse a
non-fast-forward integration rather than creating a merge automatically.
It is a conservative preference; use explicit merge/rebase commands when
needed. - `fetch.prune true` removes stale remote-tracking refs during
fetch. - These are preferences, not requirements.

Set an editor if needed:

``` bash
git config --global core.editor "code --wait"   # VS Code, if code CLI is installed
git config --global core.editor "vim"
```

## 2.4 Aliases

``` bash
git config --global alias.st "status -sb"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 --stat"
git config --global alias.unstage "restore --staged --"
```

Use them as `git st`, `git lg`, `git last`, `git unstage <file>`.
Aliases are optional; learn the underlying commands too.

------------------------------------------------------------------------

# 3. Repository state: choose the right command

Start with:

``` bash
git status
```

Then use this decision guide.

``` text
What do you need to do?
|
+-- No local repository yet? ------------------ git init / git clone
|
+-- See what changed? ------------------------- git status / git diff
|
+-- Select changes for a commit? -------------- git add / git add -p
|
+-- Staged the wrong thing? ------------------- git restore --staged
|
+-- Need to save a snapshot? ------------------ git commit
|
+-- Need latest remote information? ----------- git fetch
|      |
|      +-- Integrate it? ---------------------- git merge / git rebase / git pull
|
+-- Need to share local commits? -------------- git push
|
+-- Need to undo?
       |
       +-- Uncommitted unstaged edits? -------- git restore (careful)
       +-- Staged edits? ---------------------- git restore --staged
       +-- Private local commit? -------------- git reset / amend / rebase
       +-- Shared/pushed commit? -------------- git revert (usually safest)
       +-- Lost commit? ----------------------- git reflog
```

## 3.1 State-by-state command table

  -----------------------------------------------------------------------------------
  State             Inspect                Typical action           Caution
  ----------------- ---------------------- ------------------------ -----------------
  Not a Git repo    `git status`           `git init` or            Initialize in the
                                           `git clone`              intended folder

  Untracked files   `git status --short`   `git add <path>`         Check secrets
                                                                    before adding

  Modified,         `git diff`             `git add` or             Restore discards
  unstaged                                 `git restore`            edits

  Staged            `git diff --staged`    `git commit` or          Staging is not
                                           `git restore --staged`   committing

  Clean             `git status`           Start work, fetch,       Clean does not
                                           switch                   mean up to date

  Ahead of remote   `git status -sb`       Review and `git push`    Check
                                                                    remote/branch
                                                                    first

  Behind remote     `git status -sb`       `git fetch`, inspect,    Don't blindly
                                           integrate                overwrite local
                                                                    work

  Diverged          `git status -sb`       Choose merge or rebase   Rebase rewrites
                                                                    local commit IDs

  Merge conflict    `git status`           Resolve, stage, continue Don't blindly
                                                                    accept one side

  Detached HEAD     `git status`           Switch to branch or      Commits may
                                           create one               become hard to
                                                                    find if no branch
                                                                    points to them

  Accidentally      `git reflog`           Recover by creating a    Reflog entries
  reset/deleted                            branch                   expire eventually
  commit                                                            
  -----------------------------------------------------------------------------------

------------------------------------------------------------------------

# 4. Starting a repository

## 4.1 Create a new local repository

``` bash
mkdir my-project
cd my-project
git init
git status
```

If your Git version supports it, you can specify the initial branch:

``` bash
git init -b main
```

Add a README and first commit:

``` bash
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"
```

PowerShell supports `Set-Content README.md "# My Project"`; the `echo`
example also works in common shells, but shell quoting and encoding can
differ.

## 4.2 Clone an existing repository

``` bash
git clone <repository-url>
cd <repository-folder>
```

Clone into a chosen directory:

``` bash
git clone <repository-url> my-folder
```

Clone only a branch:

``` bash
git clone --branch <branch> --single-branch <repository-url>
```

Shallow clone (limited history):

``` bash
git clone --depth 1 <repository-url>
```

A shallow clone is useful for some CI jobs, but some history-dependent
commands, merges, and `git bisect` may need more history.

## 4.3 Connect an existing local repository to GitHub

Create an empty repository on GitHub, then:

``` bash
git remote add origin <repository-url>
git remote -v
git branch -M main
git push -u origin main
```

`-u` sets the upstream/tracking branch so future `git push` and
`git pull` can often omit the remote and branch.

If `origin` already exists:

``` bash
git remote set-url origin <repository-url>
```

Do not replace a remote URL until you confirm that it is the intended
repository.

------------------------------------------------------------------------

# 5. Inspecting changes and history

## 5.1 Status

``` bash
git status
git status --short
git status -sb
```

Short status legend: - `??` untracked file - `M` unstaged modification -
`M` staged modification - `A` staged addition - `D` deletion - `R`
rename (format may include similarity) - Two columns can show both
staged and unstaged state.

## 5.2 Diffs

``` bash
git diff                         # unstaged changes
git diff --staged                # staged changes vs HEAD
git diff HEAD                    # staged + unstaged tracked changes vs HEAD
git diff <branch1>..<branch2>    # compare the tips/trees
git diff <commit1> <commit2>
git diff --stat
git diff --name-only
git diff --check                 # whitespace errors
git diff -- README.md
```

For branch comparisons, `A..B` in `git diff` compares the endpoint
trees; this differs from how `..` is interpreted in revision-set
commands such as `git log`.

## 5.3 Commit history

``` bash
git log
git log --oneline
git log --oneline --graph --decorate --all
git log -5
git log --stat
git log --name-status
git log --author="Name"
git log --since="2 weeks ago"
git log --grep="bug fix" --oneline
git log -- path/to/file
git log <base>..<branch>
```

Useful one-line graph alias:

``` bash
git log --oneline --graph --decorate --all
```

Inspect a commit:

``` bash
git show <commit>
git show --stat <commit>
git show <commit>:path/to/file
```

Find who last changed each line:

``` bash
git blame path/to/file
git blame -L 20,50 path/to/file
```

`git blame` identifies the commit associated with lines; it does not
prove who is morally or technically responsible for a bug.

## 5.4 Find changes

``` bash
git grep "search text"                # search tracked files
git grep -n "TODO"
git log -S "someFunction" -- path/to/file
git log -G "regex-pattern" -- path/to/file
git bisect start
git bisect bad
git bisect good <known-good-commit>
# Test each checkout; then:
git bisect good   # or git bisect bad
git bisect reset
```

`-S` searches for changes in the number of occurrences of a string; `-G`
searches for diff lines matching a regex. `git bisect` performs a binary
search through history and is excellent when you can reliably test
whether a revision is good or bad.

------------------------------------------------------------------------

# 6. Staging and committing

## 6.1 Stage files

``` bash
git add README.md
git add src/
git add .
git add -A
git add -u
git add -p
```

Differences: - `git add <path>` stages the selected path. - `git add .`
stages changes under the current directory, including new files and
deletions there. - `git add -A` stages additions, modifications, and
deletions across the working tree. - `git add -u` stages
modifications/deletions of already tracked files, not new untracked
files. - `git add -p` lets you stage individual hunks interactively.

Before staging, check that secrets, `.env` files, credentials, and
generated files are excluded as appropriate.

## 6.2 Unstage without discarding edits

``` bash
git restore --staged README.md
git restore --staged .
```

This removes changes from the index but leaves the working-tree edits
intact. Older equivalent syntax:

``` bash
git reset HEAD -- README.md
```

## 6.3 Commit

``` bash
git commit -m "Add login validation"
git commit
git commit -am "Update tracked configuration"
```

`-a` automatically stages modifications and deletions of tracked files,
but **does not stage new untracked files**.

Good commit messages describe the change:

-   `Add input validation to signup form`
-   `Fix retry logic for API requests`
-   `Document local development setup`

Avoid vague messages such as `changes`, `update`, or `final`.

## 6.4 Amend the last commit

Change the last commit message:

``` bash
git commit --amend -m "Better commit message"
```

Include newly staged changes in the last commit:

``` bash
git add <file>
git commit --amend --no-edit
```

**Caution:** amend creates a new commit ID. It is generally fine for a
private, not-yet-pushed commit. If the old commit has already been
pushed and others may depend on it, coordinate before rewriting history.

------------------------------------------------------------------------

# 7. Branches

## 7.1 List and create

``` bash
git branch
git branch -a
git branch -r
git branch <new-branch>
git switch -c feature/login
git switch main
```

`git switch` is the modern command focused on changing branches.
`git checkout` remains common and can also restore files, so its
multiple meanings can be confusing.

Older equivalent:

``` bash
git checkout -b feature/login
```

## 7.2 Rename and delete branches

``` bash
git branch -m old-name new-name
git branch -m new-name               # rename current branch
git branch -d feature/login          # delete merged local branch
git branch -D feature/login          # force-delete local branch
git push origin --delete feature/login
```

`-D` can remove an unmerged branch pointer. The commits may still be
recoverable via reflog for a while, but don't rely on that as a backup.

## 7.3 Track a remote branch

``` bash
git fetch origin
git switch --track origin/feature/login
```

Or create a local branch tracking the remote branch:

``` bash
git switch -c feature/login --track origin/feature/login
```

Inspect tracking:

``` bash
git branch -vv
git status -sb
```

## 7.4 Detached HEAD

A detached HEAD occurs when you check out a commit directly, rather than
a branch:

``` bash
git switch --detach <commit>
```

To keep work created there:

``` bash
git switch -c rescue/my-work
```

If you made commits while detached, create a branch before switching
away so the commits remain easy to find.

------------------------------------------------------------------------

# 8. Merging and resolving conflicts

## 8.1 Merge a feature branch into main

``` bash
git switch main
git fetch origin
git pull --ff-only
git merge feature/login
```

After verifying tests:

``` bash
git push origin main
```

In many teams, you merge through a GitHub pull request instead of
pushing directly to `main`.

## 8.2 Merge types

-   **Fast-forward:** the target branch can move forward because it has
    no unique commits to preserve.
-   **Three-way merge:** Git combines two lines of development and their
    common ancestor; it may create a merge commit.
-   **Squash merge:** combines a branch's changes into one new commit on
    the target branch. GitHub can do this through a PR; locally,
    `git merge --squash <branch>` stages the combined changes but does
    not create a commit automatically.
-   **Rebase:** replays commits on a new base and changes their IDs.

Useful options:

``` bash
git merge --ff-only <branch>
git merge --no-ff <branch>
git merge --squash <branch>
git merge --abort
```

Use `--ff-only` when you want the merge to fail rather than create a
merge commit if fast-forwarding is impossible. `--no-ff` forces a merge
commit even when fast-forwarding would be possible.

## 8.3 Resolve conflicts

A conflict often appears like this:

``` text
<<<<<<< HEAD
current-branch version
=======
incoming-branch version
>>>>>>> feature/login
```

Procedure: 1. `git status` to list conflicted files. 2. Open each file
and decide the correct combined content. 3. Remove conflict markers. 4.
Run tests and inspect the diff. 5. Stage the resolved files. 6. Continue
or commit as the operation requires.

For a merge:

``` bash
git add <resolved-file>
git merge --continue
```

If Git says a normal commit is needed, `git commit` is also valid.

Abort a merge in progress:

``` bash
git merge --abort
```

During a rebase:

``` bash
git add <resolved-file>
git rebase --continue
git rebase --skip       # omit the current patch only if that is truly intended
git rebase --abort
```

### `ours` vs `theirs`

During a merge, `--ours` generally means the current branch's version
and `--theirs` the branch being merged in:

``` bash
git restore --ours -- path/to/file
git restore --theirs -- path/to/file
```

These commands can discard the other version of the file. Review the
conflict and diff before using them. During rebase, the meanings can
feel counterintuitive because commits are being replayed onto another
base.

------------------------------------------------------------------------

# 9. Remotes, fetch, pull, and push

## 9.1 Manage remotes

``` bash
git remote -v
git remote add origin <url>
git remote add upstream <original-repository-url>
git remote show origin
git remote get-url origin
git remote set-url origin <new-url>
git remote rename origin new-origin
git remote remove <remote>
```

-   `origin` is typically your fork or the repository you cloned.
-   `upstream` is a common name for the source repository when working
    from a fork.
-   These names are conventions and can be changed.

## 9.2 Fetch vs pull

``` bash
git fetch origin
git fetch --all --prune
git pull
git pull --ff-only
git pull --rebase
```

-   `fetch` downloads remote updates but does not merge them into your
    current branch.
-   `pull` fetches and then integrates. The integration method depends
    on flags/configuration.
-   `--prune` removes remote-tracking refs that no longer exist on the
    remote.
-   Prefer `fetch` first when you want to inspect incoming changes
    before integrating.

Inspect remote changes:

``` bash
git fetch origin
git log --oneline --left-right HEAD...origin/main
git diff HEAD..origin/main
```

## 9.3 Push

``` bash
git push
git push origin main
git push -u origin feature/login
git push --tags
git push origin <tag>
```

Push a branch and set tracking with `-u`. Pushing all tags may publish
tags you did not intend to release, so inspect tags first.

**Danger: force push**

``` bash
git push --force-with-lease
```

`--force-with-lease` is safer than `--force` because it normally refuses
to overwrite remote work that changed unexpectedly. It still rewrites
remote history. Use only when rewriting is intended and coordinated.
Avoid plain `--force` on shared branches.

## 9.4 Fork workflow

``` text
Original repository (upstream)
          │ fetch / sync
          ▼
Your fork (origin)  ◄── push your branch
          │
          ▼
Feature branch → Pull request → Review → Merge
```

Typical setup:

``` bash
git clone <your-fork-url>
cd <repo>
git remote add upstream <original-repo-url>
git fetch upstream
git switch main
git merge --ff-only upstream/main
```

If local `main` has diverged, inspect before choosing merge or rebase
instead of forcing it forward.

------------------------------------------------------------------------

# 10. Undoing, resetting, and recovering

> **First inspect:** `git status`, `git diff`, `git diff --staged`, and
> `git log --oneline -5`. If the work matters, make a backup branch or
> copy before history surgery.

## 10.1 Choose the undo command

  -----------------------------------------------------------------------
  Situation                           Usually use
  ----------------------------------- -----------------------------------
  Unstage a file, keep edits          `git restore --staged <file>`

  Discard unstaged edits to a tracked `git restore <file>` ---
  file                                destructive

  Undo a commit by adding a reversing `git revert <commit>`
  commit                              

  Move local branch pointer backward  `git reset` --- can be destructive

  Recover an old branch/commit        `git reflog`
  position                            

  Apply one existing commit elsewhere `git cherry-pick <commit>`

  Change the latest private commit    `git commit --amend`
  -----------------------------------------------------------------------

## 10.2 Restore file content

Unstage but keep edits:

``` bash
git restore --staged -- <file>
```

Discard unstaged changes in a tracked file:

``` bash
git restore -- <file>
```

Restore a file from a commit:

``` bash
git restore --source=<commit> -- <file>
```

**Danger:** `git restore -- <file>` replaces the working copy with the
staged/HEAD version depending on the state and can discard local edits.
Copy the file elsewhere first if unsure. It does not generally recover
untracked files that were deleted.

## 10.3 Reset modes

``` bash
git reset --soft HEAD~1
git reset --mixed HEAD~1
git reset --hard HEAD~1
```

All move the current branch/HEAD back one commit, but treat the index
and working tree differently:

  -----------------------------------------------------------------------
  Mode              Branch/HEAD       Staging area      Working tree
  ----------------- ----------------- ----------------- -----------------
  `--soft`          Moves             Keeps changes     Keeps changes
                                      staged            

  `--mixed`         Moves             Resets index      Keeps changes
  (default)                                             unstaged

  `--hard`          Moves             Resets index      Resets tracked
                                                        files; can
                                                        destroy work
  -----------------------------------------------------------------------

Use `--soft` when you want to redo the last commit but keep its changes
staged. Use `--mixed` when you want to redo the commit and restage
selectively. Avoid `--hard` unless you intentionally want to discard
local changes.

Unstage everything while keeping working files:

``` bash
git reset
```

Reset to a known commit is a major operation:

``` bash
git reset --mixed <commit>
```

Before doing it, make a backup ref if there is any uncertainty:

``` bash
git branch backup/before-reset
```

## 10.4 Revert a commit

``` bash
git revert <commit>
git revert HEAD
git revert --no-edit <commit>
```

This creates a new commit that reverses the target commit's changes. It
is generally safer for commits already shared with teammates because it
preserves existing history.

Revert a merge commit requires choosing a mainline parent:

``` bash
git revert -m 1 <merge-commit>
```

The correct parent matters. Inspect the merge with
`git show <merge-commit>` first.

## 10.5 Recover a lost commit using reflog

``` bash
git reflog
git reflog --date=local
```

Find the old commit hash, inspect it, then create a rescue branch:

``` bash
git show <hash>
git branch rescue/recovered-work <hash>
git switch rescue/recovered-work
```

A reflog records recent local reference movements. It is local, not a
remote backup, and old entries eventually expire or become prunable.

## 10.6 Accidentally committed a secret

1.  Revoke/rotate the credential immediately. Removing it from the
    latest commit does not make an exposed secret safe.
2.  Remove it from the tracked files and add appropriate ignore rules.
3.  If the secret was pushed, assume it may have been copied.
4.  If history must be rewritten to remove it, coordinate with
    collaborators and follow the hosting provider's sensitive-data
    removal guidance.
5.  Force-push only when the history rewrite is planned; teammates may
    need to re-clone or carefully resynchronize.

Do not paste real secrets into chat, commits, logs, or issue reports.

------------------------------------------------------------------------

# 11. Stash and selective changes

## 11.1 Save work temporarily

``` bash
git stash
git stash push -m "WIP: login form"
git stash list
git stash show
git stash show -p stash@{0}
git stash apply
git stash pop
git stash drop stash@{0}
git stash clear
```

-   `apply` applies the stash and keeps it in the stash list.
-   `pop` applies it and removes it if application succeeds.
-   `drop` deletes one stash reference.
-   `clear` removes all stash entries. **Danger:** avoid unless you
    truly want to discard them.

Include untracked files:

``` bash
git stash push -u -m "WIP including untracked files"
```

Include ignored files only if you understand the consequences:

``` bash
git stash push -a -m "WIP including ignored files"
```

## 11.2 Stage only part of a file

``` bash
git add -p
```

Common interactive responses include: - `y`: stage this hunk - `n`: skip
this hunk - `s`: split into smaller hunks, when possible - `e`: manually
edit the patch - `q`: quit

This is useful when one file contains two unrelated changes and you want
separate commits.

------------------------------------------------------------------------

# 12. Cherry-pick, rebase, and history cleanup

## 12.1 Cherry-pick: copy a commit onto the current branch

This is the requested `git cherry-pick` command (spelling:
**cherry-pick**).

``` bash
git switch release/1.2
git cherry-pick <commit-hash>
```

Git applies the change introduced by that commit and creates a new
commit on the current branch. The new commit normally has a different ID
because its parent is different.

Multiple commits:

``` bash
git cherry-pick <hash1> <hash2>
git cherry-pick <oldest>^..<newest>
```

The range syntax can be confusing; verify the commits included with
`git log` before applying it.

Apply changes without committing immediately:

``` bash
git cherry-pick --no-commit <commit>
```

If conflicts occur:

``` bash
git status
# edit files, resolve conflicts
git add <resolved-files>
git cherry-pick --continue
git cherry-pick --abort
git cherry-pick --skip
```

Use `--skip` only when you deliberately do not want that commit's
change. Cherry-pick is useful for backporting a bug fix to a release
branch, but it can duplicate changes if the commit is already present
under another ID.

## 12.2 Rebase

``` bash
git fetch origin
git switch feature/search
git rebase origin/main
```

Rebase replays your branch's commits on top of the selected base. It can
make history linear, but rewritten commits receive new IDs.

Interactive rebase:

``` bash
git rebase -i HEAD~3
```

Common todo actions: - `pick`: keep commit - `reword`: keep change, edit
message - `edit`: pause to amend - `squash`: combine with previous
commit and edit message - `fixup`: combine and discard this commit's
message - `drop`: remove commit

Abort/continue:

``` bash
git rebase --continue
git rebase --abort
```

**Rule:** do not rebase commits that teammates may already have based
work on unless the team explicitly coordinates the rewrite. Rebase is
usually fine for your own unpublished feature branch.

## 12.3 Merge vs rebase

  -----------------------------------------------------------------------
  Merge                               Rebase
  ----------------------------------- -----------------------------------
  Preserves original branch topology  Replays commits onto a new base

  May create a merge commit           Usually produces linear history

  Does not rewrite existing commits   Creates new commit IDs for replayed
                                      commits

  Often suitable for shared branches  Best used carefully on
                                      private/local work
  -----------------------------------------------------------------------

Neither is universally superior. Follow your project's contribution
policy.

------------------------------------------------------------------------

# 13. Tags and releases

## 13.1 List and create tags

``` bash
git tag
git tag -l "v1.*"
git tag v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"
git show v1.0.0
```

Annotated tags store tagger metadata and a message; they are generally
preferred for releases. Lightweight tags are simple pointers.

Tag a particular commit:

``` bash
git tag -a v1.0.0 <commit> -m "Release v1.0.0"
```

Push a tag:

``` bash
git push origin v1.0.0
git push origin --tags
```

Delete a local tag:

``` bash
git tag -d v1.0.0
```

Delete a remote tag:

``` bash
git push origin --delete v1.0.0
```

Be careful deleting a published release tag: users and automation may
depend on it.

## 13.2 Versioning

Semantic Versioning commonly uses `MAJOR.MINOR.PATCH`: - MAJOR:
incompatible public changes - MINOR: backward-compatible functionality -
PATCH: backward-compatible fixes

A tag does not automatically create a GitHub Release. Create a release
on GitHub or use `gh release create` (covered below).

------------------------------------------------------------------------

# 14. Files, ignore rules, and maintenance

## 14.1 `.gitignore`

Example:

``` gitignore
# Environment secrets
.env
.env.*
!.env.example

# Python
__pycache__/
.venv/
*.py[cod]

# Node
node_modules/
dist/

# IDE / OS
.vscode/
.DS_Store
Thumbs.db
```

Patterns are relative to the `.gitignore` location, with special rules
for `/`, `**`, and `!` negation. Check your framework's recommended
ignore rules. Never ignore a file just because it is inconvenient to
manage; understand whether it is generated, machine-specific, or
sensitive.

See why a file is ignored:

``` bash
git check-ignore -v path/to/file
```

Important: `.gitignore` does not stop tracking files already committed.

Stop tracking a file but keep it on disk:

``` bash
git rm --cached .env
git commit -m "Stop tracking local environment file"
```

For a directory:

``` bash
git rm -r --cached node_modules/
```

Do this only after confirming the path; `--cached` removes it from the
index, not your working tree.

## 14.2 Move, rename, and delete files

``` bash
git mv old-name new-name
git rm path/to/file
git rm --cached path/to/file
```

Git detects renames from snapshots and similarity; rename is not a
special stored object type.

## 14.3 Clean untracked files

Preview first:

``` bash
git clean -n
git clean -nd
```

Remove untracked files/directories:

``` bash
git clean -f
git clean -fd
```

**DANGER:** `git clean` deletes untracked files; these may include work
Git cannot recover. Ignored files require additional force/options such
as `-x`, which can delete build artifacts, local data, and environment
files. Always preview first and consider making a backup.

## 14.4 Maintenance and diagnostics

``` bash
git count-objects -v
git fsck
git gc
git reflog expire --all --dry-run
```

-   `git fsck` checks object connectivity and validity; it is useful for
    diagnostics, not a magic repair button.
-   `git gc` performs housekeeping and can prune unreachable objects
    according to Git's rules. Don't run aggressive cleanup as a first
    response to lost work.
-   `git count-objects -v` reports object storage information.

------------------------------------------------------------------------

# 15. GitHub collaboration workflows

## 15.1 Recommended team flow

``` text
Issue / task
    ↓
Create feature branch
    ↓
Make small commits + run tests
    ↓
Push branch
    ↓
Open Pull Request (PR)
    ↓
Review comments → update branch
    ↓
Checks pass + approval
    ↓
Merge PR → delete feature branch
    ↓
Sync local main
```

## 15.2 Branch naming

Examples: - `feature/user-profile` - `fix/login-redirect` -
`docs/git-notes` - `chore/update-dependencies` - `hotfix/api-timeout`

Use your team's naming rules when they exist.

## 15.3 Typical contribution commands

``` bash
git switch main
git pull --ff-only
git switch -c feature/add-search
# edit files
git status
git diff
git add <files>
git commit -m "Add search endpoint"
git push -u origin feature/add-search
```

Then open a PR on GitHub or with `gh pr create`.

After the PR merges:

``` bash
git switch main
git pull --ff-only
git branch -d feature/add-search
git fetch --prune
```

Delete the remote branch only if the PR was not already configured to
delete it and the branch is no longer needed.

## 15.4 Respond to PR review feedback

``` bash
git switch feature/add-search
# edit requested files
git add <files>
git commit -m "Address search review feedback"
git push
```

The same PR updates automatically when you push to its branch.

## 15.5 Sync a fork with upstream

``` bash
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

If fast-forward fails, inspect the divergence:

``` bash
git log --oneline --graph --decorate --all
```

Do not blindly reset or force-push until you know whether local commits
must be retained.

## 15.6 Collaboration habits that prevent pain

-   Pull/fetch before starting a task.
-   Work on feature branches; avoid committing directly to a protected
    `main`.
-   Make focused commits and explain why a change exists.
-   Pull/rebase regularly according to team policy.
-   Review `git diff --staged` before every commit.
-   Never commit API keys, passwords, tokens, private certificates, or
    real `.env` files.
-   Run tests and formatters before opening/updating a PR.
-   Treat force-push as a coordinated operation.
-   Communicate when rewriting history or changing shared branches.
-   Use PR descriptions to explain purpose, implementation, tests,
    risks, and screenshots where relevant.

------------------------------------------------------------------------

# 16. Authentication and permissions

## 16.1 HTTPS vs SSH

HTTPS remote example:

``` bash
git remote set-url origin https://github.com/OWNER/REPO.git
```

SSH remote example:

``` bash
git remote set-url origin git@github.com:OWNER/REPO.git
```

GitHub generally requires a personal access token or credential manager
for HTTPS Git operations rather than an account password. SSH requires a
configured SSH key associated with your GitHub account.

## 16.2 SSH setup overview

Generate an SSH key (choose a supported modern key type):

``` bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Start/use your platform's SSH agent and add the key according to your OS
instructions. Add the **public** key (usually `.pub`) to GitHub---not
the private key.

Test:

``` bash
ssh -T git@github.com
```

Never share or upload the private key.

## 16.3 Diagnose access

``` bash
git remote -v
git config --get credential.helper
ssh -T git@github.com
git ls-remote origin
```

Common causes: - wrong remote URL or repository owner/name - missing
repository access or organization approval - expired/revoked token - SSH
key not loaded or not added to the correct account - using HTTPS
credentials for an SSH remote (or vice versa) - organization SSO
requirements or policy restrictions

Do not paste access tokens into terminal commands that may be saved in
shell history, logs, or screenshots.

------------------------------------------------------------------------

# 17. Advanced Git: worktrees, submodules, subtrees, LFS, hooks

## 17.1 Worktrees

Worktrees let you check out multiple branches from the same local
repository into separate directories.

``` bash
git worktree list
git worktree add ../hotfix-worktree hotfix/urgent-fix
git worktree add -b feature/new-ui ../new-ui-worktree main
git worktree remove ../hotfix-worktree
git worktree prune
```

Useful when you need to review or hotfix another branch without stashing
current work. Remove only worktrees you no longer need.

## 17.2 Submodules

A submodule records a specific commit of another repository inside the
parent repository.

``` bash
git submodule add <repo-url> path/to/dependency
git submodule status
git submodule update --init --recursive
git clone --recurse-submodules <repo-url>
git submodule update --remote
```

Submodules add complexity: teammates must initialize them, and the
parent repo records the selected submodule commit. Use them when pinning
an independent repository is intentional.

## 17.3 Subtrees

Subtree integration copies another project's content into a subdirectory
while keeping the project usable without special submodule
initialization.

``` bash
git remote add vendor <vendor-repo-url>
git fetch vendor
git subtree add --prefix=vendor/library vendor main --squash
git subtree pull --prefix=vendor/library vendor main --squash
git subtree push --prefix=vendor/library vendor main
```

`git subtree` may need to be installed separately depending on your Git
distribution. Check `git subtree -h`. `--squash` avoids importing the
entire upstream history into the main repository's visible history, but
changes how history is represented.

## 17.4 Git LFS

Git LFS stores pointers in Git while large binary files are stored
through the LFS service.

``` bash
git lfs install
git lfs track "*.psd"
git lfs track "*.model"
git add .gitattributes
git add <large-file>
git commit -m "Track design assets with LFS"
git lfs ls-files
git lfs pull
```

Tracking patterns should be committed in `.gitattributes`. Moving
already-committed large files into LFS may require history migration;
don't assume adding a pattern retroactively removes old blobs.

## 17.5 Hooks

Hooks are scripts triggered by Git events, often stored under
`.git/hooks`. Examples include pre-commit and commit-msg checks.

``` bash
git config core.hooksPath .githooks
```

Teams often use tools such as pre-commit or framework-specific hook
managers to share hooks. Hooks can be bypassed in some circumstances and
are not a substitute for CI validation.

## 17.6 Sparse checkout and partial clone

For large repositories:

``` bash
git clone --filter=blob:none <repo-url>
git sparse-checkout init --cone
git sparse-checkout set app docs
git sparse-checkout list
```

Partial clone downloads some objects lazily; sparse checkout limits the
working-tree paths. These are advanced optimizations and may be
unnecessary for small projects.

## 17.7 Signed commits and tags

GPG signing (after configuring your key):

``` bash
git config --global commit.gpgsign true
git commit -S -m "Add validation"
git tag -s v1.2.0 -m "Release v1.2.0"
git verify-commit <commit>
git verify-tag v1.2.0
```

Git can also use SSH-based signing on supported versions:

``` bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true
```

Follow current GitHub documentation for uploading/allowing the relevant
signing key. A signature helps verify origin/integrity under the
configured trust model; it does not prove code is safe.

------------------------------------------------------------------------

# 18. GitHub CLI (`gh`)

GitHub CLI is a separate tool from Git. Install it and authenticate:

``` bash
gh --version
gh auth login
gh auth status
gh auth logout
```

## 18.1 Repositories

``` bash
gh repo view
gh repo view OWNER/REPO
gh repo clone OWNER/REPO
gh repo fork OWNER/REPO --clone
gh repo create
gh repo list
```

## 18.2 Pull requests

``` bash
gh pr list
gh pr status
gh pr view 123
gh pr view --web
gh pr create
gh pr create --fill
gh pr checkout 123
gh pr diff 123
gh pr review 123 --approve
gh pr review 123 --comment
gh pr merge 123
gh pr checks 123
```

Review/approval commands affect shared project state and permissions.
Only approve or merge when you've actually reviewed the code and are
authorized.

## 18.3 Issues and workflows

``` bash
gh issue list
gh issue view 42
gh issue create
gh run list
gh run view
gh run view <run-id> --log
gh run rerun <run-id>
gh workflow list
gh workflow view <workflow>
gh workflow run <workflow>
```

## 18.4 Releases

``` bash
gh release list
gh release view v1.0.0
gh release create v1.0.0 --title "v1.0.0" --notes "Release notes"
gh release upload v1.0.0 dist.zip
```

Check `gh <command> --help` because CLI flags and available options can
evolve. Avoid putting secrets into command-line arguments.

------------------------------------------------------------------------

# 19. Monorepos

A **monorepo** stores multiple projects or services in one repository.
Example:

``` text
repo/
├── apps/
│   ├── frontend/
│   └── api/
├── packages/
│   └── shared/
├── infra/
├── docs/
└── .github/workflows/
```

## 19.1 Everyday commands still apply

``` bash
git status
git diff -- apps/frontend
git add apps/frontend/
git add apps/api/ packages/shared/
git log --oneline -- apps/api/
git check-ignore -v apps/frontend/node_modules/
```

## 19.2 Monorepo practices

-   Keep root configuration and project-specific configuration clearly
    separated.
-   Use path filters in CI so frontend-only changes need not rebuild
    every unrelated service.
-   Agree on package ownership and code review requirements.
-   Avoid giant commits mixing unrelated services.
-   Ensure `.gitignore` excludes generated dependencies/build output in
    every package.
-   Use workspace tooling (for example, npm/pnpm/Yarn workspaces) when
    appropriate; Git itself does not manage package dependencies.
-   Use tags/release conventions that clearly identify which application
    or package is being released.

------------------------------------------------------------------------

# 20. GitHub Actions and deployment

Git manages version history; GitHub Actions runs automated workflows in
response to repository events. Deployment also depends on your hosting
platform and credentials.

## 20.1 Typical pipeline

``` text
git push / pull request
          ↓
GitHub Actions workflow trigger
          ↓
Checkout → Install → Lint/Test → Build
          ↓
Checks pass? ── no → report failure
          │
         yes
          ↓
Publish artifact/image → Deploy (if authorized)
```

## 20.2 Workflow file location

Workflows are YAML files under:

``` text
.github/workflows/
```

Illustrative CI workflow:

``` yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Run tests
        run: pytest
```

This is an example; adjust language, versions, install commands, and
test command to your project. Review current action versions and
official guidance before production use.

## 20.3 Useful GitHub Actions concepts

-   **Trigger:** `push`, `pull_request`, `workflow_dispatch`, scheduled
    events, and others.
-   **Job:** a unit of work running on a runner.
-   **Step:** an action or shell command within a job.
-   **Runner:** hosted or self-hosted machine that executes the job.
-   **Secret:** sensitive value configured in repository/environment/org
    settings.
-   **Artifact:** output uploaded for later download or use.
-   **Environment:** deployment target with optional approvals and
    scoped secrets.
-   **Concurrency:** controls overlapping runs.
-   **Permissions:** least-privilege access for the workflow token.

## 20.4 Secrets and deployment

-   Store credentials in GitHub Actions secrets or a suitable external
    secret manager.
-   Never hardcode secrets into workflow YAML or source files.
-   Grant the workflow only the permissions it needs.
-   Separate CI (test/build) from CD (release/deploy) where useful.
-   Protect production deployment with environments, approvals, and
    branch rules.
-   For container deployment, build and test the image before publishing
    it.
-   If a workflow fails, inspect the failed step and logs before
    rerunning it.

Useful CLI commands:

``` bash
gh run list
gh run view <run-id>
gh run view <run-id> --log
gh run rerun <run-id>
```

A rerun repeats the workflow; it does not fix a deterministic bug by
itself.

------------------------------------------------------------------------

# 21. Troubleshooting handbook

  ----------------------------------------------------------------------------------------------------------------
  Error / symptom                         Likely cause                         Safer diagnosis / fix
  --------------------------------------- ------------------------------------ -----------------------------------
  `fatal: not a git repository`           Current folder is not inside a repo  `pwd` / `cd` to the project; check
                                                                               for `.git`

  `Author identity unknown`               Name/email not configured            Set `git config --global user.name`
                                                                               and `user.email`

  `remote origin already exists`          Remote already configured            `git remote -v`; then
                                                                               `git remote set-url origin <url>`
                                                                               if appropriate

  `src refspec main does not match any`   Branch name mismatch or no commit    `git branch --show-current`;
                                          yet                                  `git status`; create a commit if
                                                                               needed

  `rejected non-fast-forward`             Remote has commits you don't have    `git fetch`; inspect graph;
                                                                               integrate before pushing

  `unrelated histories`                   Histories began independently        Verify intended repositories before
                                                                               using `--allow-unrelated-histories`

  `Permission denied (publickey)`         SSH key/access issue                 `ssh -T git@github.com`; verify key
                                                                               and remote

  HTTPS authentication fails              Token/credential mismatch            Re-authenticate via credential
                                                                               manager or `gh auth login`

  `You have divergent branches`           Both local and remote have unique    Fetch, inspect graph, choose
                                          commits                              merge/rebase policy explicitly

  `CONFLICT` during merge/rebase          Overlapping changes                  `git status`; edit markers; stage;
                                                                               continue or abort

  `detached HEAD`                         Checked out a commit directly        `git switch -c rescue/my-work` if
                                                                               you made commits

  File ignored unexpectedly               Ignore pattern matches it            `git check-ignore -v <path>`

  `.env` still appears in GitHub          It was already tracked               `git rm --cached .env`; commit;
                                                                               rotate any exposed secrets

  Accidentally staged too much            Broad `git add`                      `git restore --staged <path>`;
                                                                               review before committing

  Accidentally discarded changes          Restore/reset/clean removed work     Stop writing to the disk; inspect
                                                                               IDE/local backups; reflog helps
                                                                               commits, not arbitrary uncommitted
                                                                               files

  Push rejected after amend/rebase        Remote still has old history         Coordinate; inspect remote; only
                                                                               rewrite with team agreement and
                                                                               `--force-with-lease`

  Lock file error                         Another Git process may be running   Confirm no Git process is active
                                          or stale lock remains                before considering lock removal

  Large file rejected                     Hosting size limit                   Use appropriate artifact storage or
                                                                               Git LFS; remove oversized file from
                                                                               history if needed

  CI works locally but fails on GitHub    Environment/dependency/path/secret   Read logs; pin versions; check
                                          difference                           case-sensitive paths and missing
                                                                               secrets

  `git pull` asks how to reconcile        Pull strategy not configured         Choose explicit `--rebase`,
                                                                               `--no-rebase`, or `--ff-only` based
                                                                               on team policy
  ----------------------------------------------------------------------------------------------------------------

## 21.1 Non-fast-forward recovery

``` bash
git status
git fetch origin
git log --oneline --graph --decorate --all -20
```

Then, depending on team policy and branch state:

``` bash
git rebase origin/<branch>
# or
git merge origin/<branch>
```

Resolve conflicts, run tests, and push. If rebasing a branch that has
already been shared, coordinate before updating the remote.

## 21.2 Unrelated histories

Do not immediately run:

``` bash
git pull --allow-unrelated-histories
```

First check whether you accidentally connected the local project to the
wrong remote or created separate initial commits. If joining the
histories is truly intentional, back up the repo and resolve the
resulting merge carefully.

## 21.3 Git lock files

If Git says an index lock exists: 1. Check whether another Git
operation, IDE, or GUI is running. 2. Wait for the operation to finish.
3. Only if no Git process is active, investigate the exact lock file
mentioned. 4. Remove a stale lock only after confirming it is stale. Do
not indiscriminately delete files inside `.git`.

## 21.4 Recovery priority

If work seems lost: 1. Stop destructive commands. 2. Copy the current
project directory if feasible. 3. Check `git status`. 4. Check
`git reflog` for moved/deleted commit references. 5. Create a rescue
branch at any useful commit. 6. Check editor local history, backups, and
OS file recovery for uncommitted/untracked files. 7. Avoid `git gc`,
`git clean`, or further resets until recovery options are understood.

------------------------------------------------------------------------

# 22. Common end-to-end workflows

## 22.1 Start a feature from updated main

``` bash
git switch main
git fetch origin
git pull --ff-only
git switch -c feature/new-feature
# make changes
git status
git diff
git add <files>
git diff --staged
git commit -m "Add new feature"
git push -u origin feature/new-feature
```

Open a PR, respond to review, then sync main after merge.

## 22.2 Fix a typo in the latest private commit

``` bash
git status
# edit the file
git add <file>
git commit --amend --no-edit
```

Only amend if the commit is yours and rewriting it won't disrupt
collaborators.

## 22.3 Move one bug fix to a release branch

``` bash
git log --oneline feature/fix
git switch release/1.2
git pull --ff-only
git cherry-pick <fix-commit>
# resolve if needed
git push origin release/1.2
```

Confirm the fix's dependencies are present on the release branch.

## 22.4 Undo a bug fix already merged into a shared branch

``` bash
git switch main
git pull --ff-only
git log --oneline
git revert <bad-commit>
git push origin main
```

If the bad change was part of a merge commit, inspect it and determine
the correct mainline parent before using `git revert -m`.

## 22.5 Save current work and switch tasks

``` bash
git stash push -u -m "WIP: current feature"
git switch hotfix/urgent
# fix, test, commit, push
git switch feature/current
git stash list
git stash pop
```

Resolve conflicts if the branch changed while you were away.

## 22.6 Recover after an accidental reset

``` bash
git reflog
git show <recovered-hash>
git branch rescue/before-reset <recovered-hash>
git switch rescue/before-reset
```

Inspect the recovered branch before merging/cherry-picking its changes
into your main line.

## 22.7 Publish a release

``` bash
git switch main
git pull --ff-only
git status
# ensure tests and release checks pass
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin main
git push origin v1.0.0
```

Create the corresponding GitHub Release and release notes. Follow the
team's release process; don't tag a commit before you know it is the
intended release snapshot.

------------------------------------------------------------------------

# 23. Interview and revision notes

## 23.1 Frequently asked concepts

**What is Git?** A distributed version-control system that records
snapshots and lets developers branch, merge, and share history.

**What is the staging area?** The index: a proposed snapshot assembled
before a commit.

**What is the difference between `fetch` and `pull`?** Fetch downloads
remote updates without integrating them. Pull fetches and then merges or
rebases according to the chosen strategy.

**What is the difference between `merge` and `rebase`?** Merge joins
histories and preserves branch topology; rebase replays commits on a new
base and changes their IDs.

**What is the difference between `reset` and `revert`?** Reset moves a
branch/HEAD and may change the index/working tree. Revert creates a new
commit that reverses a previous commit's changes.

**What does `git cherry-pick` do?** Applies the change from a selected
commit onto the current branch, creating a new commit in most cases.

**What is HEAD?** The current checkout reference---usually the current
branch, but it can point directly to a commit in detached-HEAD state.

**What is a remote-tracking branch?** A local reference such as
`origin/main` representing the last fetched state of a remote branch.

**Why use feature branches?** They isolate work, support parallel
development, and provide a clean unit for review and testing.

**Why is force-push risky?** It can replace remote branch history and
disrupt work based on the overwritten commits.

**What is `reflog`?** A local record of recent movements of references,
useful for recovering commit pointers after reset/rebase/branch changes.

**What is `.gitignore` for?** It prevents untracked matching files from
being offered for tracking; it does not untrack files already committed.

## 23.2 Scenario questions

1.  You staged the wrong file.\
    **Answer:** `git restore --staged <file>`; then inspect
    `git status`.

2.  You committed a change but have not pushed it and want to add one
    more file.\
    **Answer:** stage the file, then `git commit --amend --no-edit` if
    it belongs in that commit.

3.  A bad commit is already on shared `main`.\
    **Answer:** normally use `git revert <commit>`.

4.  Your push is rejected because remote has new commits.\
    **Answer:** fetch, inspect history, integrate using the team policy,
    test, then push.

5.  You need one fix from another branch but not all its commits.\
    **Answer:** `git cherry-pick <commit>` after checking dependencies.

6.  You accidentally ran reset and lost a commit.\
    **Answer:** inspect `git reflog`, create a rescue branch at the old
    commit.

7.  You need to discard local edits.\
    **Answer:** inspect the diff and back up if needed;
    `git restore <file>` discards unstaged tracked-file edits.

8.  You need to remove a file from Git but keep it locally.\
    **Answer:** add an ignore rule if appropriate, then
    `git rm --cached <file>` and commit.

## 23.3 Explain the three reset modes in one sentence

-   `--soft`: move HEAD; keep index and working tree.
-   `--mixed`: move HEAD and reset index; keep working-tree files.
-   `--hard`: move HEAD, reset index, and overwrite tracked working-tree
    files.

------------------------------------------------------------------------

# 24. Master command index and cheat sheet

## Daily essentials

``` bash
git status
git status -sb
git diff
git diff --staged
git add <file>
git add -p
git restore --staged <file>
git commit -m "message"
git log --oneline --graph --decorate --all
git show <commit>
```

## Branches and collaboration

``` bash
git switch -c <branch>
git switch <branch>
git branch -vv
git fetch origin
git merge <branch>
git rebase origin/main
git push -u origin <branch>
git pull --ff-only
git cherry-pick <commit>
```

## Undo and recovery

``` bash
git restore --staged <file>   # unstage, keep edits
git restore -- <file>         # discard unstaged edits (destructive)
git revert <commit>           # new reversing commit
git reset --soft HEAD~1       # undo commit, keep staged
git reset HEAD~1              # undo commit, keep edits unstaged
git reflog                    # locate prior HEAD positions
git branch rescue/name <hash>
```

## Remote inspection

``` bash
git remote -v
git fetch --all --prune
git branch -a
git branch -vv
git log --oneline --left-right HEAD...origin/main
git ls-remote origin
```

## Stash

``` bash
git stash push -u -m "WIP"
git stash list
git stash show -p stash@{0}
git stash apply
git stash pop
```

## Releases

``` bash
git tag -a v1.0.0 -m "Release v1.0.0"
git show v1.0.0
git push origin v1.0.0
gh release create v1.0.0 --title "v1.0.0" --notes "Release notes"
```

## Inspect before you act

``` bash
git status
git diff
git diff --staged
git log --oneline --graph --decorate --all -20
git reflog
```

## Commands requiring extra caution

  -----------------------------------------------------------------------
  Command                             Why it needs care
  ----------------------------------- -----------------------------------
  `git restore <file>`                Discards unstaged tracked-file
                                      edits

  `git reset --hard`                  Overwrites tracked working-tree
                                      content and resets the index

  `git clean -fd`                     Deletes untracked files/directories

  `git clean -fdx`                    Also removes ignored files; can
                                      delete local dependencies and
                                      secrets

  `git branch -D`                     Deletes a branch even if unmerged

  `git push --force`                  Can overwrite remote history

  `git push --force-with-lease`       Safer guard, but still rewrites
                                      remote history

  `git rebase` on shared commits      Rewrites commit IDs and may disrupt
                                      collaborators

  `git stash clear`                   Removes all stash entries

  `git gc` / aggressive pruning       Can make unreachable objects harder
                                      or impossible to recover
  -----------------------------------------------------------------------

## Final rules of thumb

1.  **Inspect before changing:** `git status`, `git diff`, `git log`.
2.  **Fetch before integrating:** it lets you inspect remote updates
    first.
3.  **Use `revert` for shared history; use `reset` thoughtfully for
    local history.**
4.  **Use `cherry-pick` to move a specific commit, not an entire
    branch.**
5.  **Use `restore --staged` to unstage without losing edits.**
6.  **Do not force-push shared branches without coordination.**
7.  **A clean working tree is not a backup.** Push important work and
    keep backups where appropriate.
8.  **Never commit secrets.** Rotate them if exposed.
9.  **Small, meaningful commits and clear PRs make collaboration
    easier.**
10. **When in doubt, stop and inspect.** Git is powerful; haste is the
    real foot-gun.

------------------------------------------------------------------------

*This is a practical reference, not a substitute for your team's
branch/release policy. For commands with version-specific options, check
`git <command> --help` or `gh <command> --help`.*
