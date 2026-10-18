# Lazygit: A Complete Practical Textbook
## From Git Fundamentals to Advanced Workflows

**Level:** Beginner to advanced  
**Approach:** Learn the concepts, practice the workflow, understand the consequences.  
**Prerequisites:** Basic terminal usage and familiarity with files and folders.

---

# Table of Contents

1. [Introduction to Lazygit](#1-introduction-to-lazygit)
2. [Installation and First Launch](#2-installation-and-first-launch)
3. [Understanding Git Before Using Lazygit](#3-understanding-git-before-using-lazygit)
4. [The Lazygit Interface](#4-the-lazygit-interface)
5. [Keyboard Navigation](#5-keyboard-navigation)
6. [Inspecting Repository Changes](#6-inspecting-repository-changes)
7. [Staging and Unstaging Changes](#7-staging-and-unstaging-changes)
8. [Creating Good Commits](#8-creating-good-commits)
9. [Branches and Switching Between Them](#9-branches-and-switching-between-them)
10. [Understanding Commit History](#10-understanding-commit-history)
11. [Undoing Changes Safely](#11-undoing-changes-safely)
12. [Stashing Changes](#12-stashing-changes)
13. [Working with Remotes](#13-working-with-remotes)
14. [Merging Branches](#14-merging-branches)
15. [Rebasing and Interactive Rebase](#15-rebasing-and-interactive-rebase)
16. [Cherry-Picking Commits](#16-cherry-picking-commits)
17. [Resolving Merge Conflicts](#17-resolving-merge-conflicts)
18. [Fixup Commits and Squashing](#18-fixup-commits-and-squashing)
19. [Working with Tags](#19-working-with-tags)
20. [A Professional Pull Request Workflow](#20-a-professional-pull-request-workflow)
21. [Lazygit Configuration](#21-lazygit-configuration)
22. [Safety, Recovery, and Troubleshooting](#22-safety-recovery-and-troubleshooting)
23. [Recommended Daily Workflow](#23-recommended-daily-workflow)
24. [Practical Exercises](#24-practical-exercises)
25. [Quick Reference](#25-quick-reference)
26. [Official Resources](#26-official-resources)

---

# 1. Introduction to Lazygit

## 1.1 What is Lazygit?

Lazygit is an interactive terminal user interface for Git. It presents repository information in navigable panels and lets you perform Git operations through keyboard shortcuts.

Official repository: https://github.com/jesseduffield/lazygit

Without Lazygit, you might use these commands:

```bash
git status
git diff
git add src/main.py
git commit -m "feat: add configuration support"
git switch -c feat/configuration
git log --oneline --graph
git push
```

With Lazygit, you can inspect files, stage changes, compose commits, switch branches, examine history, and push from the interface.

However, Lazygit does not replace Git. It is a frontend that helps you operate Git more efficiently.

## 1.2 Why use Lazygit?

The main advantages are:

- **Visual staging:** Inspect and stage individual files, hunks, or lines.
- **History navigation:** Inspect commits and their diffs interactively.
- **Branch management:** Create, switch, merge, and rebase branches.
- **Interactive rebasing:** Reorder, edit, squash, and fix up commits.
- **Conflict resolution:** Inspect and resolve conflicting changes.
- **Contextual shortcuts:** Discover available operations from the interface.
- **Reduced command memorization:** Perform many operations without remembering every command-line option.

The main trade-off is that a graphical terminal interface can hide some of the underlying Git mechanics. You should learn what each operation does rather than blindly pressing keys.

## 1.3 Lazygit is not a replacement for Git knowledge

Suppose you accidentally select an operation that rewrites commit history.

Knowing where the button or shortcut is will not tell you whether the operation is safe.

Understanding Git allows you to answer questions such as:

- Will this operation discard uncommitted work?
- Will it rewrite commits already pushed to a shared branch?
- Will it change the branch pointer?
- Will it modify the working tree?
- Can I recover the previous state?

The objective is to become a Git user who can use Lazygit confidently, not merely a user who knows a collection of keyboard shortcuts.

# 2. Installation and First Launch

## 2.1 Install on Fedora Linux

If you are using Fedora and the relevant third-party COPR repository is available and trusted in your environment, the upstream README documents this installation route:

```bash
sudo dnf copr enable dejan/lazygit
sudo dnf install lazygit
```

Alternatively, obtain an appropriate release binary from the official project.

Official installation instructions:

https://github.com/jesseduffield/lazygit#installation

Because package sources and versions change, verify the package source before enabling an additional repository.

## 2.2 Install on Windows

Using WinGet:

```powershell
winget install -e --id=JesseDuffield.lazygit
```

Alternatively, download a release binary from the official repository.

## 2.3 Verify the installation

```bash
lazygit --version
```

If the command is not found, check whether the executable is installed and available through your `PATH`.

## 2.4 Launch Lazygit

First, navigate to an existing Git repository:

```bash
cd path/to/your/repository
```

Then launch the interface:

```bash
lazygit
```

You can also create a short shell alias, such as `lg`, if you frequently use Lazygit.

For example, in Fish:

```fish
alias lg='lazygit'
```

Add that to your Fish configuration if you want it to persist.

## 2.5 Practice in a disposable repository

Before experimenting with destructive operations, create a practice repository.

```bash
mkdir lazygit-practice
cd lazygit-practice

git init
git config user.name "Practice User"
git config user.email "practice@example.com"

echo "# Lazygit Practice" > README.md

git add README.md
git commit -m "docs: add initial README"

lazygit
```

The identity above is only for this practice repository. It does not change your global Git identity.

This repository is a safe environment for learning how Lazygit behaves.

# 3. Understanding Git Before Using Lazygit

To use Lazygit well, understand the four important parts of a Git repository.

## 3.1 The working tree

The **working tree** is the set of files you currently see and edit.

For example, suppose you modify `README.md`:

```markdown
# Lazygit Practice

Learning Git through Lazygit.
```

The modification initially exists in your working tree.

It is not automatically included in a commit.

## 3.2 The staging area

The **staging area**, also called the index, records the changes selected for the next commit.

The usual Git workflow is:

```text
Edit files
    |
    v
Working tree
    |
    | Stage selected changes
    v
Staging area
    |
    | Create commit
    v
Repository history
```

Staging allows you to decide exactly what the next commit will contain.

This is one of the most important concepts in Git.

## 3.3 The repository history

The repository stores commits, which represent snapshots of the tracked project state and metadata connecting them to previous commits.

A simplified history looks like this:

```text
A --- B --- C
```

Each letter represents a commit.

Commit `C` normally has `B` as its parent, and `B` has `A` as its parent.

Git commits are identified by object IDs, commonly displayed as abbreviated hashes.

## 3.4 Branches and `HEAD`

A branch is a movable reference to a commit.

For example:

```text
A --- B --- C  main
              ^
             HEAD
```

Here, `main` points to commit `C`, and `HEAD` indicates the currently checked-out location.

Now create a feature branch:

```text
A --- B --- C  main
              \
               D  feature/config
               ^
              HEAD
```

The feature branch can advance independently of `main`.

When you commit on the feature branch, Git advances that branch's reference.

## 3.5 The four areas in practice

| Area | Meaning | Typical operation |
|---|---|---|
| Working tree | Current file contents | Edit files |
| Staging area | Selected changes for the next commit | Stage or unstage |
| Local repository | Commits and local references | Commit, branch, rebase |
| Remote repository | Another repository, such as GitHub | Fetch, pull, push |

A common beginner mistake is to assume that saving a file, staging it, committing it, and pushing it are the same action.

They are four different steps.

# 4. The Lazygit Interface

The exact layout depends on your terminal dimensions, version, and configuration. The following is a conceptual representation.

```text
+----------------------+--------------------------+
| Status / Files       |                          |
|                      |                          |
| Changed files        | Diff / Commit details    |
|                      |                          |
+----------------------+--------------------------+
| Branches / Commits / Stash / Remotes / Other     |
+--------------------------------------------------+
| Contextual keybindings and status messages       |
+--------------------------------------------------+
```

Lazygit organizes related Git information into panels.

## 4.1 The files panel

The files panel is where you inspect modified, staged, and untracked files.

You can use it to:

- Select a file.
- Inspect its changes.
- Stage or unstage it.
- Discard selected changes.
- Open or edit the file.
- Enter a more detailed staging view.

## 4.2 The branches panel

The branches panel shows local and, where available, remote branches.

Typical tasks include:

- Creating a branch.
- Checking out a branch.
- Renaming or deleting a branch.
- Merging another branch.
- Rebasing onto another branch.
- Tracking a remote branch.

## 4.3 The commits panel

The commits panel displays history.

It helps you:

- Inspect commit messages.
- View the files changed by a commit.
- Examine a commit's diff.
- Copy commit hashes.
- Start interactive history operations.
- Revert or cherry-pick commits.

## 4.4 The stash panel

The stash panel lets you inspect and manage saved working changes.

Stashes are useful when you need to temporarily put work aside without creating a normal commit.

## 4.5 The remotes panel

The remotes panel helps you work with remote repositories and their branches.

Depending on the version and context, available operations can include fetching, pulling, pushing, and inspecting remote references.

## 4.6 The contextual keybinding panel

The bottom of the interface typically shows operations available in the current context.

**Always trust the shortcuts displayed by your installed version over a shortcut list copied from an older tutorial.**

Lazygit supports extensive customization, and keybindings can vary.

# 5. Keyboard Navigation

## 5.1 The essential navigation keys

Common default bindings include:

| Key | Typical action |
|---|---|
| `j` / `k` | Move down / up |
| `h` / `l` | Move left / right |
| Arrow keys | Navigate in the corresponding direction |
| `Tab` | Move between panels or views |
| `Enter` | Open, confirm, or enter a selected item |
| `Esc` | Return or cancel in many contexts |
| `?` | Show contextual help |
| `/` | Search in supported views |
| `q` | Quit in many contexts |

The exact effect of a key depends on which panel or dialog is active.

For example, `j` may select another file in one panel but navigate a list of commits in another.

## 5.2 Why context matters

Lazygit is not a conventional graphical application with a mouse-first interface.

The meaning of a shortcut is often contextual.

For example, pressing `d` can have different consequences depending on the selected panel and item. In a staging context, it can discard unstaged changes or unstage staged changes. In other contexts, it may represent a different operation.

Never assume a shortcut has the same meaning everywhere.

## 5.3 How to discover a shortcut

When you want to perform an operation:

1. Navigate to the relevant panel.
2. Select the file, branch, commit, or other item.
3. Read the contextual keybindings.
4. Press `?` if more detail is needed.
5. Read any confirmation prompt before accepting.

Learning to discover shortcuts is more useful than memorizing a huge key list.

# 6. Inspecting Repository Changes

Before staging or committing anything, inspect your changes.

This is a fundamental software engineering habit.

## 6.1 Understand the file statuses

Git commonly distinguishes these situations:

| Status | Meaning |
|---|---|
| Modified | A tracked file differs from its recorded version |
| Untracked | A file is not yet tracked by Git |
| Staged | Changes have been selected for the next commit |
| Renamed | Git detects a rename in the relevant comparison |
| Deleted | A tracked file has been removed |
| Conflicted | A merge or related operation needs conflict resolution |

Some status displays combine multiple conditions, such as a file that has both staged and unstaged changes.

## 6.2 Inspect a file's diff

A diff shows how file content differs between two states.

For example:

```diff
- DEBUG = True
+ DEBUG = False
```

The minus line represents removed content, while the plus line represents added content.

A diff can help you detect:

- Accidental changes.
- Debug statements.
- Temporary files.
- Unrelated formatting changes.
- Secrets or credentials.
- Missing tests.
- Incorrect configuration values.

## 6.3 Inspect untracked files

Untracked files deserve attention because they are not necessarily intended for version control.

For example:

```text
.env
__pycache__/
coverage.xml
src/config.py
```

A source file may need to be committed, while a local environment file or generated cache usually should not be.

A `.gitignore` file helps Git ignore matching untracked files, but adding a path to `.gitignore` does not automatically remove a file that Git already tracks.

## 6.4 Your first inspection workflow

In your practice repository:

1. Edit `README.md`.
2. Create another file, such as `notes.txt`.
3. Launch Lazygit.
4. Inspect the modified and untracked files.
5. Select each file and examine its diff where available.

Do not stage anything yet. First learn how Lazygit represents your repository state.

# 7. Staging and Unstaging Changes

Staging is where Lazygit becomes particularly useful.

## 7.1 Stage a whole file

Select a modified or untracked file in the files panel.

Use the displayed stage shortcut. In the default configuration, `Space` is commonly used to stage or toggle changes.

Staging a file means selecting its current relevant changes for the next commit.

Equivalent Git command for a tracked file:

```bash
git add README.md
```

For an untracked file, `git add` also begins tracking it.

## 7.2 Unstage a file

If you staged a file accidentally, you can unstage it.

The staged changes are removed from the index, but ordinarily remain in your working tree.

Equivalent command:

```bash
git restore --staged README.md
```

This is different from discarding the working-tree changes.

## 7.3 Stage only part of a file

Suppose you modify a Python file:

```python
DEBUG = True

def calculate_total(items):
    return sum(items)
```

You change it to:

```python
DEBUG = False

def calculate_total(items):
    return sum(items)
```

Then you also add an unrelated debugging statement.

You might want to stage the configuration correction without including the debugging statement.

Lazygit supports interactive staging of selected hunks and lines through its staging interface.

A **hunk** is a contiguous section of a diff. A file can contain multiple hunks.

The general workflow is:

1. Select the modified file.
2. Enter the staging or patch view.
3. Navigate to the relevant hunk.
4. Use the displayed shortcut to stage that hunk.
5. If supported in the current view, select individual lines.
6. Review the staged diff before committing.

Common default bindings in the staging interface include `Space` to stage a selected change and `Tab` to switch between staged and unstaged views. The exact behavior depends on the current context.

## 7.4 Why partial staging matters

Suppose you modify two unrelated things:

- Fix a database configuration bug.
- Reformat an unrelated module.

You can stage only the bug fix and create:

```text
fix(database): correct connection configuration
```

Then stage the formatting change separately and create:

```text
style: format database module
```

This produces smaller, clearer commits.

## 7.5 Staged and unstaged changes can coexist

A file can contain staged changes and additional unstaged changes at the same time.

For example:

```text
Original file
    |
    +-- Change A: staged
    |
    +-- Change B: unstaged
```

The next commit includes Change A but not Change B.

This is one of the most powerful aspects of Git's staging area.

**Important:** Always inspect the staged diff, not merely the full working-tree diff, before committing.

# 8. Creating Good Commits

A commit is a recorded snapshot of selected changes, accompanied by metadata and a message.

## 8.1 Create a commit

The normal workflow is:

1. Make changes.
2. Inspect the diff.
3. Stage the intended changes.
4. Review the staged diff.
5. Select the commit operation.
6. Enter a meaningful message.
7. Confirm the commit.

In Lazygit's default configuration, `c` commonly opens the commit operation from the staging context. Other commit options may have separate shortcuts.

Always check the contextual help before confirming.

## 8.2 Write meaningful commit messages

A good commit message communicates the intent of a change.

Weak examples:

```text
update
fix
changes
final
test
```

Better examples:

```text
feat(database): add connection pooling
fix(config): validate database URL
test(repository): cover missing user
docs: explain local development setup
refactor(settings): separate database configuration
ci: run tests on pull requests
```

These follow the Conventional Commits style.

Common types include:

| Type | Purpose |
|---|---|
| `feat` | Introduce functionality |
| `fix` | Correct a defect |
| `test` | Add or improve tests |
| `docs` | Documentation-only changes |
| `refactor` | Restructure code without intended behavior change |
| `style` | Formatting or style changes without intended logic change |
| `ci` | Continuous integration changes |
| `build` | Build system or dependency-related changes |
| `chore` | Maintenance that does not fit a more specific type |

A scope, such as `database` or `config`, is optional unless your project requires it.

## 8.3 The commit should represent one logical change

A logical change is a coherent unit of work.

For example:

```text
feat(database): add async session factory
test(database): cover session lifecycle
docs(database): document session usage
```

These can be separate commits when they represent distinct units of work. In some projects, tests and implementation naturally belong in the same commit.

The objective is not to maximize the number of commits. It is to make history understandable and reviewable.

## 8.4 Amend a commit

Suppose you committed a change but forgot to include a small related modification.

You may amend the latest commit rather than creating a separate commit.

In Lazygit, select the relevant commit or staging operation and inspect the available amend action.

The equivalent Git command is:

```bash
git commit --amend
```

Amending replaces the latest commit with a new commit. The resulting commit ID changes.

**Safety rule:** Amending a commit that has already been pushed can complicate collaboration because the remote branch still points to the old commit. Avoid rewriting shared history unless the workflow explicitly permits it.

# 9. Branches and Switching Between Them

Branches let you work on independent changes without modifying the branch used for another line of development.

## 9.1 Create a branch

Suppose you want to add database configuration support.

Create a branch such as:

```text
feat/database-config
```

In Lazygit:

1. Open the branches panel.
2. Invoke the create-branch action.
3. Enter the branch name.
4. Confirm whether you want to check it out immediately, if prompted.

The equivalent Git commands are:

```bash
git switch -c feat/database-config
```

The command creates the branch and switches to it.

## 9.2 Switch branches

Select the desired branch and use the checkout or switch action shown by Lazygit.

Equivalent command:

```bash
git switch main
```

Switching branches can fail or require additional decisions when uncommitted changes conflict with the target branch.

Do not force a checkout just to dismiss a warning. First understand what changes would be overwritten.

## 9.3 Delete a branch

Once a feature branch has been merged, you may no longer need it.

Lazygit provides a branch deletion operation. Inspect the selected branch carefully before confirming.

Equivalent Git command:

```bash
git branch -d feat/database-config
```

The `-d` option normally refuses to delete a branch whose changes have not been merged.

The force-delete form is:

```bash
git branch -D feat/database-config
```

This can delete an unmerged branch reference. It should not be your default cleanup operation.

Deleting a branch does not necessarily delete the underlying commit objects immediately, but it removes a convenient reference to the branch's history.

## 9.4 Local and remote branches

A local branch is a reference in your local repository.

A remote-tracking branch is a local reference that records the last observed state of a branch on a remote.

For example:

```text
main
origin/main
feat/database-config
```

Here, `origin/main` is not the live remote branch itself. It is your local record of the remote branch's state.

A fetch updates these remote-tracking references.

## 9.5 A sensible branch naming convention

Examples:

```text
feat/user-registration
fix/database-timeout
refactor/settings
test/authentication
docs/installation
ci/github-actions
```

Use your project's established naming convention if it has one.

# 10. Understanding Commit History

History is more than a list of commit messages. It is a graph of commits and parent relationships.

## 10.1 Inspect a commit

In the commits panel:

1. Select a commit.
2. Read its message and metadata.
3. Inspect its changed files.
4. Examine the diff.
5. Determine whether the change belongs to the current task.

This is useful when reviewing your own history or examining someone else's branch.

## 10.2 Understand the history graph

Consider:

```text
A --- B --- C  main
       \
        D --- E  feature
```

The branches have diverged.

- `main` contains commits `A`, `B`, and `C`.
- `feature` contains `A`, `B`, `D`, and `E`.

The branches share a common ancestor, `B`.

If you want to integrate `feature`, you might merge it into `main` or rebase it onto `main`.

## 10.3 Compare commits

Comparing two commits shows the net changes between their snapshots.

This is useful for answering questions such as:

- What did this feature introduce?
- Did a refactor change behavior?
- Which files changed between releases?
- What will a branch contribute to the target branch?

Remember that a diff between two commits represents the difference between their snapshots, not necessarily a simple concatenation of all changes in between.

## 10.4 Find the source of a regression

Suppose a test passed in an earlier commit but fails now.

A useful process is:

1. Inspect recent commits.
2. Examine changes to the relevant file.
3. Identify commits that could have introduced the behavior.
4. Reproduce the issue.
5. Use Git's history tools or bisect if necessary.

Lazygit makes commit inspection convenient, but understanding the causal relationship between commits and behavior remains your responsibility.

# 11. Undoing Changes Safely

This is one of the most important sections in the textbook.

Git has several operations that people casually call "undo," but they do different things.

## 11.1 Discard unstaged changes

Suppose you modified a tracked file but have not staged the changes.

You want to discard the working-tree modifications.

In Lazygit, select the appropriate unstaged change and invoke the discard operation.

The equivalent Git command is:

```bash
git restore -- path/to/file.py
```

This restores the working-tree file from the index.

**Warning:** The discarded unstaged changes can be difficult or impossible to recover if they were never saved elsewhere.

## 11.2 Unstage changes

If changes are staged but not committed, you can unstage them.

```bash
git restore --staged -- path/to/file.py
```

This changes the staging area without ordinarily discarding the working-tree changes.

## 11.3 Reset a commit

`git reset` moves the current branch reference and can also change the staging area or working tree, depending on the mode.

| Mode | Main effect |
|---|---|
| `--soft` | Move `HEAD`; keep the index and working tree |
| `--mixed` | Move `HEAD`; reset the index; keep working-tree files |
| `--hard` | Move `HEAD`, reset the index, and reset tracked working-tree files |

For example:

```bash
git reset --soft HEAD~1
```

This moves the current branch back one commit while keeping the changes from that commit staged.

By contrast:

```bash
git reset --mixed HEAD~1
```

keeps the changes in the working tree but removes them from the index.

And:

```bash
git reset --hard HEAD~1
```

discards tracked working-tree changes that differ from the target commit and moves the branch reference backward.

**Never use a hard reset casually.** Uncommitted changes may be permanently lost. Untracked files are not necessarily removed by `git reset --hard`, so the operation should not be confused with cleaning every file in the directory.

In Lazygit, reset operations are usually available through the commits context. Read the operation description carefully before confirming.

## 11.4 Revert a commit

A revert creates a new commit that applies the inverse of an earlier commit's changes.

```bash
git revert <commit-hash>
```

Unlike reset, revert does not normally move the current branch backward to erase a commit from history.

This makes revert especially useful for changes that have already been pushed to a shared branch.

For example:

```text
A --- B --- C --- D
                  |
                  D is reverted by a new commit E
```

The history remains visible:

```text
A --- B --- C --- D --- E
```

Commit `E` reverses the relevant changes introduced by `D`, subject to conflicts and any subsequent modifications.

## 11.5 Reset versus revert

| Situation | Usually appropriate |
|---|---|
| Unstage accidental changes | Restore the index or unstage in Lazygit |
| Discard local unstaged changes | Restore or discard the selected changes |
| Undo the latest local commit but keep its work | Soft or mixed reset |
| Undo a commit already shared with others | Revert |
| Rewrite a private feature branch | Reset or interactive rebase, when appropriate |

These are guidelines, not absolute rules. The correct choice depends on whether the commit has been shared and whether its changes should be retained.

# 12. Stashing Changes

A stash temporarily saves work from the working tree and index so you can switch tasks without making a normal commit.

## 12.1 Why stash?

Imagine you are implementing a feature, but an urgent bug needs attention.

Your current work is incomplete. You do not want to create a misleading commit just to switch branches.

A stash provides a temporary storage mechanism.

## 12.2 Create a stash

In Lazygit, open the stash-related interface and use the available stash operation.

The equivalent command is:

```bash
git stash push -m "work in progress: database config"
```

This saves applicable tracked working-tree and staged changes and restores the working tree toward a clean state.

By default, untracked files are not included. To include them, use:

```bash
git stash push -u -m "include untracked work"
```

Ignored files are not included by `-u`.

## 12.3 Apply a stash

You can restore a saved stash while keeping it in the stash list:

```bash
git stash apply
```

To apply a specific stash:

```bash
git stash apply stash@{1}
```

The exact stash identifier may change as the stash list changes.

## 12.4 Pop a stash

```bash
git stash pop
```

This attempts to apply the stash and remove it from the stash list if the application succeeds.

If conflicts occur, inspect the working tree carefully. Do not assume that every stash operation will apply cleanly.

## 12.5 Drop a stash

```bash
git stash drop stash@{0}
```

Dropping a stash removes its reference from the stash list.

Avoid dropping a stash until you have verified that its contents are no longer needed.

# 13. Working with Remotes

A remote is another Git repository, commonly hosted on GitHub or a similar service.

## 13.1 Understand `origin`

A repository cloned from GitHub often has a remote named `origin`.

Inspect remotes using:

```bash
git remote -v
```

The name `origin` is conventional, not mandatory.

## 13.2 Fetch

Fetching downloads remote objects and updates remote-tracking references.

```bash
git fetch origin
```

Fetching does not ordinarily integrate the downloaded changes into your current branch.

This is useful because it lets you inspect remote changes before deciding what to do with them.

## 13.3 Pull

Pull combines fetching with an integration step, usually a merge or a rebase depending on configuration and options.

```bash
git pull
```

Because pulling can change your current branch and working tree, understand your repository's configured pull strategy.

A more deliberate workflow is often:

1. Fetch the remote.
2. Inspect the updated remote-tracking branch.
3. Decide whether to merge or rebase.
4. Perform the chosen integration.

## 13.4 Push

Push publishes commits from your local repository to a remote repository.

```bash
git push
```

If the branch has no upstream configured, Git may ask you to establish one.

A common first push is:

```bash
git push -u origin feat/database-config
```

The `-u` option establishes the upstream relationship for future operations.

## 13.5 Force push and force-with-lease

A normal push can be rejected when the remote branch has advanced in a way that cannot be handled by a fast-forward update.

After rewriting your own feature branch, you might need a force push.

The safer alternative is generally:

```bash
git push --force-with-lease
```

This checks that the remote reference has not changed unexpectedly relative to the local information Git is using.

It is safer than unconditional `--force`, but it is not risk-free. It can still overwrite work if used incorrectly.

Avoid force-pushing shared branches unless your team explicitly agrees to the history rewrite.

## 13.6 A professional remote workflow

Before pushing:

1. Inspect the branch you are on.
2. Review your commits.
3. Fetch the remote.
4. Integrate the target branch if needed.
5. Run tests.
6. Push the branch.
7. Open or update the pull request.

This is more reliable than repeatedly pulling and pushing without inspecting the resulting history.

# 14. Merging Branches

A merge integrates the histories of two branches.

## 14.1 Basic merge

Suppose you have:

```text
A --- B --- C  main
       \
        D --- E  feature
```

To integrate the feature into `main`, you normally check out `main` and merge the feature branch.

```bash
git switch main
git merge feature
```

If the histories diverged and Git creates a merge commit, the result may look like:

```text
A --- B --- C ------- M  main
       \             /
        D --- E ----/    feature
```

A merge can also be a fast-forward, in which Git simply advances a branch reference without creating a merge commit.

## 14.2 Merge in Lazygit

The typical workflow is:

1. Open the branches panel.
2. Select the branch to integrate.
3. Invoke the merge action.
4. Confirm the target and source branches.
5. Resolve conflicts if necessary.
6. Inspect the result.

Pay close attention to which branch is currently checked out. A merge integrates the selected source branch into the current branch, not the other way around.

## 14.3 When to merge

Merging is appropriate when you want to preserve the existing branch history and integrate the work of another branch.

It is often used for:

- Integrating a feature branch into a mainline.
- Combining shared work.
- Preserving an explicit integration point.
- Following a team policy that prefers merge commits.

# 15. Rebasing and Interactive Rebase

Rebasing is one of the most powerful features in Git, and Lazygit makes it easier to explore interactively.

## 15.1 What is rebase?

Rebase takes commits and replays their changes onto another base commit.

Suppose your history looks like this:

```text
A --- B --- C  main
       \
        D --- E  feature
```

You want your feature branch to be based on the latest `main`.

A rebase can produce:

```text
A --- B --- C  main
              \
               D' --- E'  feature
```

The commits `D'` and `E'` are new commits with new identities.

They may represent the same logical changes as `D` and `E`, but their parents differ.

## 15.2 Rebase in Lazygit

The general workflow is:

1. Check out the feature branch.
2. Open the branches panel.
3. Select the intended base branch.
4. Invoke the rebase action.
5. Inspect any prompts or conflicts.
6. Complete the rebase only after understanding the result.

The equivalent Git command is:

```bash
git rebase main
```

This rebases the current branch onto `main`.

## 15.3 Interactive rebase

Interactive rebase lets you edit a sequence of commits.

Typical operations include:

- Reordering commits.
- Rewording commit messages.
- Editing a commit.
- Squashing commits together.
- Fixing up commits.
- Dropping commits.

A conceptual interactive-rebase list might look like:

```text
pick   abc1234 feat: add database settings
pick   def5678 fix: correct database URL
pick   ghi9012 test: cover invalid settings
```

You can change the actions to produce a cleaner history.

The exact UI and shortcuts depend on your Lazygit version and configuration. Use the interactive rebase interface and contextual help to identify the available actions.

## 15.4 Reordering commits

Suppose you have:

```text
A --- B --- C --- D
```

You realize that the logical order should be:

```text
A --- B --- D' --- C'
```

Interactive rebase can reorder commits, but doing so replays changes in a different order. Conflicts may arise if the commits depend on each other.

## 15.5 Squashing commits

Suppose your feature branch contains:

```text
feat: add database settings
fix: correct database settings
test: cover database settings
```

You want one logical commit:

```text
feat(database): add database settings
```

Interactive rebase can combine these commits.

A squash operation combines changes and lets you construct a commit message. A fixup operation also combines changes but typically discards the fixup commit's message in favor of the target commit's message.

## 15.6 Editing a commit

Interactive rebase can pause at a selected commit so you can amend it.

This is useful when you want to:

- Add a missing file.
- Correct a message.
- Remove an accidental change.
- Split a mixed commit into logical pieces.

Because rebase rewrites commits, the affected commit IDs and those of their descendants change.

## 15.7 The golden rule of rebase

**Do not casually rebase commits that other people already depend on.**

Rebasing published history can force collaborators to reconcile divergent histories.

Rewriting your own unpublished feature branch is generally much easier to manage.

# 16. Cherry-Picking Commits

Cherry-picking applies the change introduced by a selected commit onto the current branch.

## 16.1 Example

Suppose:

```text
A --- B --- C  main
       \
        D --- E  feature
```

Commit `E` contains an independent bug fix that you need on `main`, but you do not want to merge the entire feature branch.

You can cherry-pick `E` onto `main`.

```bash
git switch main
git cherry-pick <commit-hash>
```

Conceptually:

```text
A --- B --- C --- E'  main
       \
        D --- E       feature
```

`E'` is a new commit that applies the selected change.

## 16.2 Cherry-pick in Lazygit

The typical workflow is:

1. Check out the destination branch.
2. Open the commits panel.
3. Select the desired commit.
4. Invoke the cherry-pick action.
5. Resolve conflicts if necessary.
6. Inspect the resulting commit.

## 16.3 When to cherry-pick

Cherry-picking is useful for:

- Applying a specific fix to a maintenance branch.
- Moving a small independent change between branches.
- Backporting a bug fix.
- Recovering a useful commit without integrating an entire feature branch.

It should not be used as a substitute for understanding branch relationships. Repeated cherry-picking between long-lived branches can create duplicated changes and confusing history.

# 17. Resolving Merge Conflicts

A merge conflict occurs when Git cannot automatically combine changes.

Conflicts can arise during merging, rebasing, cherry-picking, and other operations.

## 17.1 Example conflict

Suppose one branch contains:

```python
timeout = 10
```

Another branch changes the same line to:

```python
timeout = 30
```

Git may produce conflict markers:

```python
<<<<<<< HEAD
timeout = 10
=======
timeout = 30
>>>>>>> feature
```

The exact labels depend on the operation and repository state.

These markers indicate competing versions of the affected content.

## 17.2 Understand the sides

In a typical merge conflict:

- `HEAD` represents the current branch's version.
- The other side represents the branch being merged.

During a rebase, the interpretation of the current and incoming changes can differ from what a beginner expects. Do not assume that "ours" always means the same logical branch in every operation.

## 17.3 Resolve the conflict

A reliable process is:

1. Open the conflicted file.
2. Read both versions.
3. Decide what the intended final behavior should be.
4. Edit the file to remove conflict markers.
5. Run tests or inspect the result.
6. Mark the file resolved by staging it.
7. Continue the merge, rebase, or cherry-pick as appropriate.

For a merge, staging the resolved files is normally part of completing the merge.

For a rebase, you must continue the rebase after resolving the conflicts.

Lazygit provides conflict-related operations in the relevant context. Inspect the available actions before choosing one.

## 17.4 Do not blindly choose one side

A conflict is not simply a competition in which one side is always correct.

For example, one side may introduce a timeout value while the other adds validation. The correct result might need both changes.

The goal is to preserve intended behavior, not merely eliminate conflict markers.

## 17.5 Abort an operation

If you discover that you started the wrong operation or cannot safely resolve the conflicts, consider aborting it.

For a merge:

```bash
git merge --abort
```

For a rebase:

```bash
git rebase --abort
```

For a cherry-pick:

```bash
git cherry-pick --abort
```

These commands attempt to return the repository to its state before the respective operation began.

They are not universal recovery mechanisms for unrelated changes. Inspect the repository state before aborting.

# 18. Fixup Commits and Squashing

Fixup commits are particularly useful when maintaining a clean feature-branch history.

## 18.1 The problem

Suppose your history is:

```text
feat(database): add connection settings
fix(database): handle empty URL
test(database): cover connection settings
```

After reviewing the code, you realize the second commit belongs with the first.

You could create another corrective commit, but the history might become cluttered.

## 18.2 Create a fixup commit

Git provides:

```bash
git commit --fixup <target-commit>
```

The resulting commit is marked as a fixup for the target commit.

Lazygit offers fixup-related functionality, including a way to find a suitable base commit and create a fixup.

## 18.3 Autosquash

Git's interactive rebase supports autosquashing fixup commits:

```bash
git rebase -i --autosquash main
```

The fixup commit is placed near its target and marked for combination.

Lazygit can help you carry out the corresponding interactive rebase workflow.

## 18.4 When to use it

Fixup commits are useful when:

- Responding to code review.
- Correcting a small mistake in an earlier commit.
- Improving the logical structure of unpublished history.
- Keeping the final pull request history concise.

Do not rewrite shared history merely to make it look cleaner unless your team has agreed on that workflow.

# 19. Working with Tags

Tags are references used to mark specific points in history, often for releases.

For example:

```text
v1.0.0
v1.1.0
v2.0.0
```

## 19.1 Create a tag

A lightweight tag can be created with:

```bash
git tag v1.0.0
```

An annotated tag includes metadata and a message:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

## 19.2 Push a tag

```bash
git push origin v1.0.0
```

Tags are not automatically pushed by every ordinary branch push.

## 19.3 Delete a tag

Deleting a local tag:

```bash
git tag -d v1.0.0
```

Deleting a remote tag:

```bash
git push origin --delete v1.0.0
```

Be cautious when deleting or moving published release tags. Other people and automation may rely on them.

Lazygit provides tag-related operations in supported versions; use the tags view and its contextual shortcuts.

# 20. A Professional Pull Request Workflow

Let's combine the concepts into a realistic development workflow.

Assume you are working on a Python backend and want to add database configuration.

## Step 1: Inspect the repository

Launch Lazygit:

```bash
lazygit
```

Inspect:

- The current branch.
- The working tree.
- Existing staged changes.
- Recent commits.
- Whether the repository is already in the middle of a merge or rebase.

Do not begin by assuming the repository is clean.

## Step 2: Create a feature branch

Create:

```text
feat/database-config
```

Make sure you are working on the intended branch before editing files.

## Step 3: Implement the feature

Edit the relevant files in your editor.

For example:

```text
src/
    config.py
    database.py
tests/
    test_config.py
```

The exact structure depends on your project.

## Step 4: Review the changes

Return to Lazygit.

Inspect each modified file and read its diff.

Ask:

- Is every change related to this feature?
- Did I accidentally modify a generated file?
- Did I include a secret?
- Are tests missing?
- Did the changes introduce an unrelated formatting diff?

## Step 5: Stage selectively

Stage only the intended changes.

If the implementation and tests are logically related, they may belong in one commit. If the changes represent distinct logical units, consider separating them.

## Step 6: Create a meaningful commit

For example:

```text
feat(database): add database configuration
```

Review the commit in the history panel.

## Step 7: Run the tests

Use your project's test command outside Lazygit.

For a Python project using pytest:

```bash
pytest
```

For projects with formatting, linting, or type checks, run the relevant configured tools as well.

A clean Git diff does not prove that the code works.

## Step 8: Fetch and inspect the target branch

Fetch the remote:

```bash
git fetch origin
```

Inspect whether the target branch has advanced.

If the team requires the feature branch to be updated, integrate the target branch using the team's merge or rebase policy.

## Step 9: Push the feature branch

For a new branch:

```bash
git push -u origin feat/database-config
```

Inspect the remote state and confirm that the expected commits were published.

## Step 10: Open a pull request

Create a pull request through your Git hosting platform.

The pull request should communicate:

- What changed.
- Why it changed.
- How it was tested.
- Any limitations or follow-up work.

## Step 11: Address review feedback

Make additional changes, inspect the diff, and create follow-up commits or fixup commits according to the project's history policy.

Do not amend or rebase published commits without understanding the team's workflow.

## Step 12: Merge the pull request

Follow the project's policy for merge commits, squash merges, or rebasing.

Lazygit manages your local Git operations, but the hosting platform usually manages pull requests and their reviews.

---

# 21. Lazygit Configuration

Lazygit is configurable. You can customize keybindings, appearance, commands, and other behavior.

## 21.1 Find the configuration location

The configuration path depends on the operating system and environment.

On Linux, a common location is:

```text
~/.config/lazygit/config.yml
```

The exact path can differ according to the version and configuration directory settings.

Consult the official documentation before editing the file.

## 21.2 Configure custom keybindings

Lazygit allows supported keybindings to be customized.

Before changing them:

1. Learn the defaults.
2. Identify a real usability problem.
3. Check whether the intended action is configurable.
4. Make a small change.
5. Restart or reload as required.
6. Verify that the new binding works as expected.

Avoid changing many bindings before you understand the default interface. Otherwise, learning resources and troubleshooting become harder to follow.

## 21.3 Custom commands

Lazygit supports custom commands in its configuration.

These can integrate project-specific workflows, such as running tests or invoking a script.

For example, a project might benefit from a command that runs its test suite.

Before adding a custom command, understand:

- What command it executes.
- Which directory it runs from.
- Whether it modifies files.
- Whether it blocks the interface.
- Whether it depends on environment variables.

Never add an opaque shell command simply because it appears in an online configuration.

## 21.4 Keep configuration maintainable

A good configuration should improve your workflow without making Lazygit difficult to use on another machine.

Prefer a small number of well-understood customizations over a large collection of shortcuts you rarely use.

# 22. Safety, Recovery, and Troubleshooting

## 22.1 The most important safety rule

Before performing a destructive operation, answer three questions:

1. What exact state am I changing?
2. Which changes or commits could be lost?
3. How would I recover if the operation is wrong?

If you cannot answer these questions, stop and inspect the repository.

## 22.2 Understand the reflog

The reflog records changes to local references such as `HEAD`, helping you identify earlier positions of branches and checkouts.

Inspect it using:

```bash
git reflog
```

Suppose you reset a branch and later realize that a commit is missing. The reflog may reveal the previous commit ID.

You can inspect the commit:

```bash
git show <commit-hash>
```

If appropriate, you can create a recovery branch:

```bash
git branch recovery/my-lost-work <commit-hash>
```

This creates a reference to the commit so you can inspect it safely.

The reflog is not a permanent backup. Its entries expire, and it cannot recover work that was never recorded in Git or another storage system.

## 22.3 What if a commit disappeared?

Do not immediately run more reset or cleanup commands.

Instead:

1. Stop making changes.
2. Inspect the current branch and commit history.
3. Run `git reflog`.
4. Identify the previous commit.
5. Inspect that commit with `git show`.
6. Create a recovery branch if appropriate.
7. Decide how to restore the work.

## 22.4 What if Lazygit shows unexpected changes?

Inspect the working tree and staging area separately.

Possible causes include:

- Partially staged files.
- Line-ending changes.
- Generated files.
- File permission changes.
- Changes caused by formatters.
- Files ignored by one environment but not another.

Do not discard the changes until you understand their origin.

## 22.5 What if a push is rejected?

A push may be rejected because the remote branch has advanced.

A safe response is:

1. Fetch the remote.
2. Inspect the remote-tracking branch.
3. Determine whether the branches diverged.
4. Integrate the remote changes according to your team's policy.
5. Run tests.
6. Push again.

Avoid treating force push as the universal solution to a rejected push.

## 22.6 What if you enter a rebase by mistake?

Check the current repository state.

If a rebase is in progress and you have not completed it, the appropriate abort operation may return you to the state before the rebase began:

```bash
git rebase --abort
```

If the rebase has already completed, abort is no longer the appropriate mechanism. Inspect the history and reflog instead.

## 22.7 Lazygit undo and redo

Some Lazygit versions provide undo/redo support for certain operations.

Do not assume every Git operation can be reversed through the same mechanism.

The safest approach is to understand the underlying Git operation and inspect the repository after undoing or redoing it.

---

# 23. Recommended Daily Workflow

This is a concise routine you can use for normal development.

## Before coding

- Inspect the current branch.
- Check for uncommitted changes.
- Fetch remote updates when relevant.
- Create or switch to the correct feature branch.

## During coding

- Make one coherent change at a time.
- Inspect the diff regularly.
- Stage selectively.
- Avoid mixing unrelated changes.
- Keep temporary files and secrets out of commits.

## Before each commit

- Inspect the staged diff.
- Check for debugging code and accidental files.
- Run relevant tests.
- Write a descriptive commit message.

## Before pushing

- Inspect the commit history.
- Confirm the destination branch.
- Integrate required remote changes.
- Run the test suite.
- Push to the intended remote.

## After pushing

- Verify that the branch is published.
- Open or update the pull request.
- Address review feedback.
- Clean up obsolete local branches when safe.

This routine is more valuable than memorizing every advanced Lazygit feature.

# 24. Practical Exercises

Complete these exercises in your disposable practice repository.

## Exercise 1: Inspect a diff

1. Modify `README.md`.
2. Create an untracked file.
3. Open Lazygit.
4. Identify the modified and untracked files.
5. Inspect their diffs where available.

**Goal:** Understand the difference between modified and untracked files.

## Exercise 2: Stage only part of a file

1. Modify two separate parts of a file.
2. Enter the staging interface.
3. Stage only one hunk, or individual lines if supported.
4. Inspect the staged and unstaged views.
5. Create a commit.
6. Confirm that only the intended change was committed.

**Goal:** Understand the index as a separate layer between the working tree and repository history.

## Exercise 3: Create and switch branches

1. Create `feat/example`.
2. Make a change.
3. Commit it.
4. Switch back to `main`.
5. Inspect the history.
6. Switch to the feature branch again.

**Goal:** Understand that branches are references to commits and that each branch can have a different history.

## Exercise 4: Undo a staged change

1. Modify a tracked file.
2. Stage it.
3. Unstage it without discarding the working-tree modification.
4. Confirm that the file remains modified.

**Goal:** Distinguish unstage from discard.

## Exercise 5: Revert a commit

1. Create a small change and commit it.
2. Revert the commit using Lazygit.
3. Inspect the history.
4. Compare the revert commit with the original commit.

**Goal:** Understand how a revert preserves history while reversing a change.

## Exercise 6: Practice a merge

1. Create a feature branch.
2. Add and commit a change.
3. Return to `main`.
4. Merge the feature branch.
5. Inspect the resulting history.

**Goal:** Understand merge integration and fast-forward versus merge-commit behavior.

## Exercise 7: Practice a conflict

1. Create two branches from the same base.
2. Change the same line differently on both branches.
3. Commit each change.
4. Attempt to merge one branch into the other.
5. Inspect the conflict.
6. Resolve it and run the relevant checks.
7. Complete the merge.

**Goal:** Learn that conflict resolution requires understanding the intended final code.

## Exercise 8: Practice interactive rebase

1. Create a feature branch.
2. Make three small commits.
3. Open the interactive rebase interface.
4. Inspect the available actions.
5. Squash two related commits.
6. Review the rewritten history.

**Goal:** Understand how interactive rebase changes commit structure.

Perform this exercise only in your disposable repository until you understand the consequences of rewriting history.

## Exercise 9: Recover a commit using reflog

1. Create a commit.
2. Record its hash.
3. Move the branch backward using a suitable reset operation.
4. Inspect the reflog.
5. Create a recovery branch pointing to the earlier commit.
6. Verify that the commit's contents are available.

**Goal:** Understand how Git's reference history can help recover commits after certain mistakes.

## Exercise 10: Complete a pull request workflow

1. Create a feature branch.
2. Make a logical change.
3. Stage and commit it.
4. Run tests.
5. Fetch the remote.
6. Inspect the history.
7. Push the feature branch.
8. Open a pull request.

**Goal:** Combine Lazygit operations with a realistic software engineering workflow.

---

# 25. Quick Reference

The following table lists common Git operations and their underlying commands. Lazygit provides corresponding operations in context-sensitive panels, but shortcuts can vary by version and configuration.

| Task | Git command or concept |
|---|---|
| Launch Lazygit | `lazygit` |
| Inspect repository state | `git status` |
| Inspect unstaged changes | `git diff` |
| Inspect staged changes | `git diff --staged` |
| Stage a file | `git add <file>` |
| Unstage a file | `git restore --staged <file>` |
| Discard unstaged changes | `git restore -- <file>` |
| Create a commit | `git commit` |
| Amend the latest commit | `git commit --amend` |
| Create and switch branches | `git switch -c <branch>` |
| Switch branches | `git switch <branch>` |
| Delete a merged branch | `git branch -d <branch>` |
| Inspect history | `git log --oneline --graph` |
| Create a stash | `git stash push` |
| Apply a stash | `git stash apply` |
| Fetch a remote | `git fetch origin` |
| Pull changes | `git pull` |
| Push changes | `git push` |
| Merge a branch | `git merge <branch>` |
| Rebase onto a branch | `git rebase <branch>` |
| Cherry-pick a commit | `git cherry-pick <hash>` |
| Revert a commit | `git revert <hash>` |
| Inspect reference history | `git reflog` |
| Create a tag | `git tag -a <tag>` |

## Five rules to remember

1. **Inspect before modifying.** Read the diff before staging or committing.
2. **Understand the index.** Staged and unstaged changes are different states.
3. **Prefer small, logical commits.** Clear history improves review and debugging.
4. **Be cautious with history rewriting.** Reset and rebase can change branch history; force push can overwrite remote references.
5. **Verify the result.** After a complex operation, inspect the branch, history, working tree, and test results.

---

# 26. Official Resources

Use the official documentation as the source of truth for version-specific shortcuts and features.

- [Lazygit GitHub repository and installation instructions](https://github.com/jesseduffield/lazygit)
- [Lazygit official keybindings reference](https://github.com/jesseduffield/lazygit/blob/master/docs/keybindings/Keybindings_en.md)
- [Lazygit documentation directory](https://github.com/jesseduffield/lazygit/tree/master/docs)
- [Git official documentation](https://git-scm.com/doc)
- [Git branching and merging documentation](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell)
- [Git interactive staging documentation](https://git-scm.com/book/en/v2/Git-Tools-Interactive-Staging)

The official Lazygit repository also links to tutorials covering basic usage, advanced features, and rebase workflows.

---

# Final Learning Strategy

Do not try to memorize the entire interface in one sitting.

Learn Lazygit in this order:

1. Navigation and repository inspection.
2. Diffs, staging, and unstaging.
3. Commits and commit messages.
4. Branch creation and switching.
5. History inspection and safe undo operations.
6. Fetching, pulling, and pushing.
7. Merge and conflict resolution.
8. Interactive rebase, cherry-picking, and fixup commits.
9. Recovery, configuration, and advanced workflows.

**Your first milestone:** Become comfortable reviewing changes, staging only the intended content, creating clean commits, and switching branches without losing work.

Once those operations become routine, Lazygit will be much easier to use for advanced Git workflows and real software engineering projects.