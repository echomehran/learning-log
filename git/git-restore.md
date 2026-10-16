# Git Restore: A Deep and Complete Guide

`git restore` is a relatively modern Git command introduced in **Git 2.23 (August 2019)**. It was created to solve a long-standing problem: `git checkout` was overloaded, doing two completely different things (switching branches and restoring files). `git restore` gives file restoration its own dedicated, focused command.

Understanding it deeply means understanding **where** you're restoring **from** and **to**.

---

## 1. The Core Mental Model

`git restore` moves content **from a source** to a **destination**. That's it.

```
   SOURCE                    DESTINATION
   ┌─────────────┐           ┌─────────────┐
   │  --source   │──────────▶│  --worktree │  (your files)
   │  (default:  │           │  --staged   │  (the index)
   │   index or  │           │   or both   │
   │   HEAD)     │           └─────────────┘
   └─────────────┘
```

**Default behavior:**
- **Without `--staged`**: source is the **index**, destination is the **working tree**
- **With `--staged`**: source is **HEAD**, destination is the **index**
- **With both `--staged --worktree`**: source is **HEAD**, destination is **both**

The command can also be used to restore the content in the index with `--staged`, or restore both the working tree and the index with `--staged --worktree`.

---

## 2. Syntax Overview

```bash
git restore [<options>] [--source=<tree>] [--staged] [--worktree] [--] <pathspec>...
git restore [<options>] [--source=<tree>] [--staged] [--worktree] --pathspec-from-file=<file>
git restore (-p|--patch) [<options>] [--source=<tree>] [--staged] [--worktree] [--] [<pathspec>...]
```

The `<pathspec>` is what you're restoring — a file, a directory, or `.` for everything.

---

## 3. The Three Primary Use Cases

### Use Case 1: Discard Unstaged Changes in Working Directory

This is the most common use. You edited a file, haven't staged it, and want to throw away those edits.

```bash
git restore filename.txt
```

This restores the file from the **index** (the last staged version) to the **working tree**. If you haven't staged anything, the index matches HEAD, so the file goes back to its last committed state.

**Example:**
```bash
echo "good code" > app.py
git add app.py
git commit -m "initial"

echo "bad code" >> app.py
git restore app.py        # "bad code" is gone
cat app.py                # "good code" only
```

**Discard all unstaged changes:**
```bash
git restore .
```
The dot means "here and in all folders below".

⚠️ **This is destructive.** Unstaged changes are **not** stored in Git anywhere. Once discarded, they are gone forever.

### Use Case 2: Unstage Changes (Remove from Staging Area)

You ran `git add`, then changed your mind. You want to unstage without losing your work.

```bash
git restore --staged filename.txt
```

This restores the **index** from **HEAD**, effectively removing the file from the staging area. Your working directory changes are **preserved**.

**Example:**
```bash
echo "new feature" >> app.py
git add app.py
git status                # "Changes to be committed"

git restore --staged app.py
git status                # "Changes not staged for commit"
cat app.py                # "new feature" is still there!
```

**Unstage everything:**
```bash
git restore --staged .
```

**Key distinction:** Unstaging only takes a file out of the next commit and keeps your work. Nothing is lost by unstaging — you can stage the file again with `git add`.

### Use Case 3: Restore a File from a Specific Commit

You want to recover an older version of a file from history.

```bash
git restore --source=HEAD~3 filename.txt
```

This restores the **working tree** from the specified commit (HEAD~3), leaving the index alone.

**Example:**
```bash
git log --oneline
# abc1234 latest commit
# def5678 two commits ago

git restore --source=def5678 app.py
# app.py in working directory now matches its state at def5678
```

**Common mistake:** Use the commit *before* the change you want to undo. If you want to undo the most recent commit's changes, use `HEAD~1` (the parent), not `HEAD`.

---

## 4. Combining `--staged` and `--worktree`

To restore **both** the index and the working tree from a source:

```bash
git restore --source=HEAD --staged --worktree filename.txt
```

This is equivalent to the old `git checkout HEAD -- filename.txt`. The short form:

```bash
git restore -s@ -SW filename.txt
```

Where `-s@` means `--source=HEAD`.

**When to use this:** You want a file to be exactly as it was at a specific commit, both in the index and on disk. This effectively "resets" the file completely.

---

## 5. Interactive Restoration with `--patch`

The `--patch` (or `-p`) flag lets you selectively restore hunks instead of entire files.

```bash
git restore -p filename.txt
```

Git will walk you through each changed chunk and ask whether to discard it.

**Real scenario:** You made 5 changes to `index.html`. 3 are good, 2 are bad. You want to keep the good ones.

```bash
git restore -p index.html
```

Git presents each hunk:
```
@@ -10,6 +10,7 @@
 <div class="header">
-  <h1>Old Title</h1>
+  <h1>New Title</h1>
 </div>

Discard this hunk from worktree [y,n,q,a,d,j,J,g,/,e,?]? 
```

You type `n` to keep the good change, `y` to discard the bad one. After the process, only the good changes remain.

**With `--patch` and no pathspec**, Git will prompt to restore **all modified paths**.

---

## 6. Restoring from Other Sources

### From a branch or tag:

```bash
git restore --source=main~2 src/utils.py
git restore --source=v1.0.0 config.yml
```

### From the merge base of two branches:

```bash
git restore --source="feature...main" file.txt
```

This uses the special `A...B` syntax to specify the merge base.

### Short form:

```bash
git restore -s HEAD~3 app.py
```

---

## 7. Handling Merge Conflicts

During a merge or rebase conflict, files have **unmerged entries** in the index. `git restore` can help resolve them.

**Keep "our" version:**
```bash
git restore --ours conflicted.txt
```

**Keep "their" version:**
```bash
git restore --theirs conflicted.txt
```

⚠️ During `git rebase` and `git pull --rebase`, `ours` and `theirs` appear **swapped** compared to a normal merge. Always verify with `git status`.

**Recreate the conflict markers:**
```bash
git restore --merge conflicted.txt
```

**Ignore unmerged paths:**
```bash
git restore --ignore-unmerged .
```

---

## 8. Restoring Deleted Files

If you deleted a file with `rm` but haven't committed the deletion:

```bash
git restore deleted_file.txt
```

This brings it back from the index (which still has it, since you didn't stage the deletion).

If you **committed** the deletion and want it back:

```bash
git restore --source=HEAD~1 deleted_file.txt
```

This restores from the commit before the deletion.

---

## 9. Restoring in Sparse Checkout Mode

In sparse checkout mode, `git restore` by default only updates entries matched by `<pathspec>` and sparse patterns.

**Override sparse patterns:**
```bash
git restore --ignore-skip-worktree-bits file.txt
```

This unconditionally restores any files in `<pathspec>`, even if they're outside your sparse checkout scope.

---

## 10. Submodules

If `<pathspec>` names an active submodule:

```bash
git restore --recurse-submodules libs/foo
```

The submodule's working tree will be restored to the commit recorded in the superproject, and **any local modifications overwritten**. Without this flag, submodule working trees are not updated.

---

## 11. Safety: What Can You Recover?

| Scenario | Recoverable? | How |
|----------|-------------|-----|
| Unstaged changes discarded with `git restore` | ❌ No | Never stored in Git |
| Staged changes unstaged with `git restore --staged` | ✅ Yes | Still in working directory |
| Committed changes restored over | ✅ Yes | In Git history |
| Accidentally deleted file, not staged | ✅ Yes | `git restore` from index |
| Accidentally deleted file, staged | ✅ Yes | `git restore --staged` then `git restore` |

**The dangerous case** is `git restore <file>` without `--staged`. If you haven't staged the changes, they exist **nowhere in Git** — only in your working directory. Once overwritten, they're gone.

**Safety net:** Use `git stash` before destructive operations:
```bash
git stash push -m "backup before restore"
git restore .
```
If something goes wrong, `git stash pop`.

---

## 12. Real-World Scenarios

### Scenario A: Undo a Bad Edit

```bash
# You're editing config.py and accidentally delete an important line
vim config.py

# Realize the mistake
git status
# modified: config.py

git restore config.py
# config.py is back to its committed state
```

### Scenario B: Unstage a File You Added by Mistake

```bash
# You meant to add only app.py but accidentally added everything
git add .

git status
# Changes to be committed:
#   modified: app.py
#   modified: secrets.env    <- oops!

git restore --staged secrets.env
git status
# Changes not staged for commit:
#   modified: secrets.env    <- now unstaged, work preserved
```

### Scenario C: Recover an Older Version

```bash
# You refactored utils.py and broke something
git log --oneline
# a1b2c3d refactor utils
# e4f5g6h working utils

git restore --source=e4f5g6h src/utils.py
# src/utils.py is now the working version from before the refactor
```

### Scenario D: Selectively Discard Hunks

```bash
# You made multiple changes to index.html, some good some bad
git restore -p index.html

# Git walks you through each hunk
# Type 'n' to keep good changes, 'y' to discard bad ones
```

---

## 13. Comparison: `restore` vs `checkout` vs `reset`

| Task | Modern (`restore`) | Old (`checkout`/`reset`) |
|------|-------------------|--------------------------|
| Discard working dir changes | `git restore file` | `git checkout -- file` |
| Unstage | `git restore --staged file` | `git reset HEAD file` |
| Restore from commit (working dir) | `git restore -s commit file` | `git checkout commit -- file` |
| Restore from commit (both) | `git restore -s commit -SW file` | `git checkout commit -- file` |

**Why the change?** `git checkout` had too many meanings. `git restore` is focused exclusively on file restoration. `git switch` handles branch switching. This separation makes commands safer and clearer.

---

## 14. Cheat Sheet

| Command | What it does |
|---------|--------------|
| `git restore file` | Discard unstaged changes in working directory |
| `git restore .` | Discard all unstaged changes |
| `git restore --staged file` | Unstage file (keep working changes) |
| `git restore --staged .` | Unstage everything |
| `git restore --source=HEAD~2 file` | Restore file from 2 commits ago (working dir only) |
| `git restore -s commit -SW file` | Restore both index and working dir from commit |
| `git restore -p file` | Interactively discard hunks |
| `git restore --ours file` | Keep "our" version in conflict |
| `git restore --theirs file` | Keep "their" version in conflict |
| `git restore --merge file` | Recreate conflict markers |
| `git restore --recurse-submodules lib` | Restore submodule working tree |

---

## 15. Practice Exercises

1. **Basic discard:** Create a file, commit it, edit it, then `git restore` it. Verify the edit is gone.

2. **Unstage:** Create a file, `git add` it, then `git restore --staged` it. Verify the file is still on disk but unstaged.

3. **From commit:** Make three commits changing the same file. Use `git restore --source=HEAD~2 file` to bring back the old version.

4. **Interactive:** Make multiple unrelated changes to a file. Use `git restore -p` to discard only one hunk.

5. **Both destinations:** Use `git restore -s HEAD~1 -SW file` and verify both index and working directory match the old commit.

6. **Deleted file:** Delete a committed file with `rm`, then `git restore` it.

7. **Conflict:** Create a merge conflict, then use `git restore --ours` to resolve.

---

`git restore` is a precision tool. The key is always asking: **What is my source? What is my destination?** Once you internalize that model, every flag combination makes sense.