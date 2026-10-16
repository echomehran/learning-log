# `git checkout` vs `git switch`: A Deep and Complete Guide

The relationship between `git checkout` and `git switch` is really a story about **Git's evolution toward clarity and safety**. Understanding this deeply means understanding *why* the split happened, *what each command is actually for*, and *how to use them correctly in real workflows*.

---

## 1. The Historical Problem: `git checkout` Did Too Much

For most of Git's history, `git checkout` was a "Jack of all trades" command. It could:

- **Switch branches**
- **Create and switch to a new branch** (`-b` flag)
- **Detach HEAD** to inspect an old commit
- **Restore files** from the index or another commit
- **Discard working directory changes**

This overload created real dangers. Consider this scenario:

```bash
# You have a branch called "test" AND a file called "test.txt"
# You're on main and want to switch to the branch
git checkout test
```

What did you just do? Switch to the branch? Restore the file? The ambiguity was a source of confusion and accidental data loss .

The Git manual itself acknowledges this issue: "Because checking out a branch is so common an operation, however, `git checkout abc` takes 'abc' as a `<tree-ish>` in such a situation" — meaning Git had to guess what you meant .

---

## 2. The Solution: Git 2.23 Splits the Command

In **August 2019 (Git 2.23)**, the Git team introduced two new commands to separate `git checkout`'s responsibilities:

| New Command | Purpose | Replaces |
|-------------|---------|----------|
| **`git switch`** | Switch branches, create branches, detach HEAD | Branch operations of `checkout` |
| **`git restore`** | Restore files, discard changes, unstage | File operations of `checkout` |

The core design principle: **One command should do one thing** .

`git checkout` still works (for backward compatibility), but it's now considered the "legacy" command. New users are encouraged to learn `switch` and `restore` instead .

---

## 3. What `git switch` Actually Does

`git switch` is exclusively for **branch operations**. It cannot restore files. Its scope is:

- Switch to an existing branch
- Create and switch to a new branch
- Detach HEAD to inspect a commit
- Create an orphan branch

### 3.1 Switch to an Existing Branch

```bash
git switch main
git switch feature/login
```

Output:
```
Switched to branch 'main'
```

This updates your working tree and index to match the target branch. Your HEAD now points to that branch .

### 3.2 Create and Switch to a New Branch

The `-c` flag stands for **create** (intuitive and memorable):

```bash
git switch -c feature/login
```

Output:
```
Switched to a new branch 'feature/login'
```

You can specify a starting point:

```bash
git switch -c hotfix v2.1.0      # branch from a tag
git switch -c experiment main    # branch from another branch
git switch -c test abc1234       # branch from a commit hash
```

The `-c` flag has a longhand: `--create` .

### 3.3 Switch Back to Previous Branch

```bash
git switch -
```

This is a shortcut for `@{-1}`, which refers to the previously checked-out branch. Useful for bouncing between two branches .

### 3.4 Detach HEAD

```bash
git switch --detach abc1234
git switch -d v2.0.0
```

This puts you in a **detached HEAD state** — you're looking at a specific commit, not a branch. Any commits you make here aren't on any branch and can be lost .

### 3.5 Create an Orphan Branch

```bash
git switch --orphan gh-pages
```

Creates a branch with **no parent commits** — a completely disconnected history. Useful for `gh-pages` or starting a fresh history .

---

## 4. What `git checkout` Still Does

`git checkout` remains a **multi-purpose command**. It can do everything `git switch` does, plus file operations.

### 4.1 Branch Operations (Same as `switch`)

```bash
git checkout main                    # switch branch
git checkout -b feature/login        # create + switch
git checkout -                     # previous branch
git checkout --detach abc1234        # detach HEAD
```

The `-b` flag is the **old equivalent** of `switch -c`. The `b` presumably stands for "branch," but it's not as intuitive as `-c` for "create" .

### 4.2 File Operations (NOT in `switch`)

This is the critical difference. `git checkout` can **overwrite files**:

```bash
# Discard unstaged changes to a file
git checkout -- filename.txt

# Restore a file from a specific commit
git checkout HEAD~3 -- filename.txt

# Get a file from another branch without switching
git checkout other-branch -- path/to/file.js
```

**Real-world use case:** You need one file from another branch but don't want to switch:

```bash
git checkout feature/new-api -- src/api/client.js
```

This copies `client.js` from the `feature/new-api` branch into your current working tree and stages it, while you stay on your current branch .

---

## 5. The Critical Safety Difference

This is the **most important practical difference** between the two commands.

### The Ambiguity Problem

```bash
# Scenario: You have both a branch named "test" and a file named "test.txt"
# You type:
git checkout test
```

Git has to decide: are you switching to the branch `test`, or checking out the file `test`? If a branch `test` exists, Git assumes you mean the branch. But if you *meant* to restore the file `test`, you could accidentally switch branches instead .

With `git switch`:

```bash
git switch test        # ALWAYS means "switch to branch test"
git restore test       # ALWAYS means "restore file test"
```

No ambiguity. No guessing. This is why `switch` is considered **safer** .

### The "Did I Just Delete My Work?" Problem

```bash
# You have uncommitted changes in filename.txt
git checkout filename.txt    # ← Did this switch branches or discard my changes?
```

With `git checkout`, this **discards your uncommitted changes**. With `git restore filename.txt`, the intent is explicit. With `git switch filename.txt`, the command **fails** because `switch` doesn't do file operations .

---

## 6. Command Comparison Table

| Operation | `git checkout` (legacy) | `git switch` (modern) | `git restore` (modern) |
|-----------|------------------------|----------------------|----------------------|
| Switch to branch | `git checkout <branch>` | `git switch <branch>` | N/A |
| Create + switch | `git checkout -b <name>` | `git switch -c <name>` | N/A |
| Switch to previous | `git checkout -` | `git switch -` | N/A |
| Detach HEAD | `git checkout <commit>` | `git switch -d <commit>` | N/A |
| Discard file changes | `git checkout -- <file>` | N/A | `git restore <file>` |
| Unstage file | `git reset HEAD <file>` | N/A | `git restore --staged <file>` |
| Restore from commit | `git checkout <commit> -- <file>` | N/A | `git restore -s <commit> <file>` |
| Get file from branch | `git checkout <branch> -- <file>` | N/A | `git restore -s <branch> <file>` |

---

## 7. Real-World Examples

### Example 1: Daily Branch Switching

```bash
# Old way
git checkout feature/user-auth
git checkout -b hotfix/login-bug

# Modern way
git switch feature/user-auth
git switch -c hotfix/login-bug
```

The modern way is clearer: `switch` means you're moving between branches. `-c` means you're creating one .

### Example 2: The Dangerous Ambiguity

```bash
# You accidentally created a branch named "config" and also have config.json
# You want to discard changes to config.json

# DANGEROUS: Git might interpret this as switching to branch "config"
git checkout config

# SAFE: Unambiguous file restoration
git restore config
```

This exact scenario is why `git switch` and `git restore` were created .

### Example 3: Discarding Local Changes

```bash
# You made a mess in app.py and want to start over

# OLD (works, but less clear)
git checkout -- app.py

# MODERN (explicit intent)
git restore app.py
```

The `--` in the old command was needed to disambiguate "app.py the file" from "app.py the branch" (if one existed). With `restore`, no `--` is needed .

### Example 4: Fetching a File from Another Branch

```bash
# You're on main but need the latest version of config.py from develop
# You don't want to switch branches

# OLD
git checkout develop -- config.py

# MODERN
git restore --source=develop config.py
```

Both copy the file into your working tree and stage it. The modern version is more explicit about the **source** .

### Example 5: Detached HEAD for Debugging

```bash
# You want to inspect the code at a specific tag

# OLD
git checkout v1.5.0

# MODERN
git switch --detach v1.5.0
```

Both give you the same result: a detached HEAD. The `--detach` flag makes it **explicit** that you're intentionally detaching .

---

## 8. When to Use Which

### Use `git switch` when:
- You want to **switch branches** (the most common operation)
- You want to **create a new branch** and switch to it
- You want to **detach HEAD** for inspection
- You value **clarity and safety** over backward compatibility

### Use `git restore` when:
- You want to **discard working directory changes**
- You want to **unstage files**
- You want to **restore a file from a specific commit or branch**

### Use `git checkout` when:
- You're working with **old scripts or documentation** that use it
- You're on a **very old Git version** (pre-2.23)
- You need a **one-liner** that does something `switch`/`restore` can't (rare)

For **new projects and learning**, the modern commands are strongly recommended. The Git community and documentation now present `switch` and `restore` as the preferred tools .

---

## 9. Mental Model

Think of it this way:

**`git checkout`** = A Swiss Army knife. It does many things, but you have to be careful which blade you're using.

**`git switch`** = A dedicated door. It moves you from one branch to another. Nothing else.

**`git restore`** = A dedicated eraser/undo tool. It fixes files. Nothing else.

The split isn't about new *functionality* — it's about **intent**. When you type `git switch`, you (and anyone reading your terminal history) know exactly what you meant. When you type `git checkout`, it could be anything .

---

## 10. Practice Exercises

1. **Basic switch:** Create a branch, switch between it and main using both `git checkout` and `git switch`. Notice the output differences.

2. **The ambiguity:** Create a branch named `config` and a file named `config.txt`. Try `git checkout config` vs `git switch config` vs `git restore config.txt`. Observe which does what.

3. **Create and switch:** Use `git checkout -b feature/a` and `git switch -c feature/b`. Compare the flags (`-b` vs `-c`).

4. **Detach HEAD:** Use `git checkout v1.0` (or any tag) and `git switch --detach v1.0`. Both work, but one is explicit.

5. **File restore:** Make changes to a file, then use `git checkout -- file` and `git restore file`. Both discard changes, but one is clearer.

6. **Cross-branch file copy:** On main, use `git checkout develop -- config.py` and `git restore -s develop config.py`. Verify both work identically.

---

## Summary

`git checkout` and `git switch` aren't competitors — they're **generations** of the same idea. `git checkout` was the original multi-tool; `git switch` (and `git restore`) are the modern, focused replacements created to eliminate ambiguity and prevent accidental data loss.

Use `git switch` for branches. Use `git restore` for files. Know `git checkout` for legacy compatibility. That's the complete picture.