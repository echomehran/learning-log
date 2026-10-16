# Git Working Directory: A Deep and Complete Guide

The **working directory** (also called the **working tree** or **working copy**) is the actual folder on your filesystem where you edit files. It's one of Git's **three trees** — and understanding it deeply is the difference between "using Git" and "understanding Git."

Let's go all the way down.

---

## 1. The Three Trees of Git

Git conceptually maintains three "trees" (states of your project):

| Tree | Location | What it holds |
|------|----------|---------------|
| **Working Directory** | Your filesystem folder | Files you're editing right now |
| **Index (Staging Area)** | `.git/index` | What will go into the next commit |
| **HEAD (Repository)** | `.git/objects` + refs | The last committed snapshot |

```
Working Directory  →  Index (staging)  →  HEAD (last commit)
       ↑                    ↑                    ↑
   you edit here      git add here       git commit here
```

Every Git workflow is about moving changes between these three trees.

---

## 2. Where the Working Directory Lives

When you run:
```bash
git init myproject
cd myproject
```

You get:
```
myproject/          ← working directory (this is your working tree)
├── .git/           ← repository metadata (NOT part of working tree)
│   ├── HEAD
│   ├── index
│   ├── objects/
│   └── refs/
└── (your files go here)
```

**Key insight:** `.git/` is *not* part of the working directory. The working directory is everything **except** `.git/`.

---

## 3. How Git Tracks the Working Directory

Git does **not** watch your files in real time. It compares the working directory to the index **on demand** (when you run `git status`, `git diff`, etc.).

It compares using:
- **File path**
- **File content hash** (via blob SHA)
- **File mode** (executable bit)
- **mtime + size** (as a fast pre-check, stored in the index)

If mtime and size match what's in the index, Git assumes the file is unchanged (this is why touching a file sometimes makes Git re-check it).

---

## 4. `git status` — The Working Directory's Mirror

```bash
git status
```

Example:
```
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        modified:   app.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   README.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        newfile.txt
```

Three categories:
- **Staged** — in index, ready to commit
- **Unstaged** — modified in working dir, not yet staged
- **Untracked** — not known to Git at all

Short form:
```bash
git status -s
```
```
M  app.py        # staged modification
 M README.md     # unstaged modification
?? newfile.txt   # untracked
MM both.py       # staged AND unstaged changes
A  added.py      # staged new file
D  deleted.py    # staged deletion
```

The **two-character** status code: first char = index vs HEAD, second char = working dir vs index.

---

## 5. Untracked vs Tracked Files

**Untracked:** Git has never been told about this file. It won't be committed unless you `git add` it.

**Tracked:** A file that exists in the index. It can be:
- **Unmodified** — matches index
- **Modified** — differs from index
- **Deleted** — removed from working dir but still in index

**Real example:**

```bash
echo "hello" > a.txt
git status           # a.txt is untracked

git add a.txt        # now tracked, staged
git status           # "new file: a.txt"

git commit -m "add a.txt"
echo "world" >> a.txt
git status           # "modified: a.txt" (tracked + modified)
```

---

## 6. `.gitignore` — Controlling What Git Sees

Untracked files can be ignored so they don't clutter `git status`.

`.gitignore`:
```
*.log
node_modules/
.env
build/
!important.log       # negation: don't ignore this one
```

**Rules:**
- Patterns apply recursively
- `/foo` anchors to repo root; `foo/` matches any directory named foo
- `!pattern` re-includes something
- Once a file is **tracked**, `.gitignore` no longer applies to it

**Real example:**
```bash
echo "debug.log" > debug.log
git status           # untracked

echo "*.log" >> .gitignore
git status           # debug.log disappears
```

But if you already committed `debug.log`:
```bash
git add debug.log && git commit -m "oops"
echo "*.log" >> .gitignore
git status           # still shows modifications to debug.log!
```
You must `git rm --cached debug.log` to untrack it.

**Global ignore:**
```bash
git config --global core.excludesfile ~/.gitignore_global
```

**Check why a file is ignored:**
```bash
git check-ignore -v debug.log
```

---

## 7. `git diff` — Comparing the Trees

Three important diffs:

**Working dir vs Index** (what's unstaged):
```bash
git diff
```

**Index vs HEAD** (what's staged for the next commit):
```bash
git diff --staged     # or --cached
```

**Working dir vs HEAD** (everything since last commit):
```bash
git diff HEAD
```

**Real example:**
```bash
echo "line1" > f.txt
git add f.txt
git commit -m "initial"

echo "line2" >> f.txt
git add f.txt
echo "line3" >> f.txt

git diff              # shows line3 (unstaged)
git diff --staged     # shows line2 (staged)
git diff HEAD         # shows line2 + line3
```

---

## 8. `git add` — Moving Changes into the Index

`git add` copies the **current content** of the file into the index (as a blob object).

**Important:** `git add` snapshots the content **at the moment you run it**. If you edit after adding, the new edits are unstaged.

```bash
echo "v1" > f.txt
git add f.txt           # index has "v1"
echo "v2" > f.txt       # working dir has "v2"
git status
```
```
Changes to be committed:
        new file:   f.txt     ← this is "v1" (staged)
Changes not staged:
        modified:   f.txt     ← this is "v2" (unstaged)
```

**Interactive staging:**
```bash
git add -p f.txt        # stage hunks one at a time
git add -i              # interactive menu
```

**Real-world use of `-p`:** You fixed a bug AND added a feature in the same file. You want two separate commits:
```bash
git add -p app.py
# choose 'y' for the bugfix hunks, 'n' for the feature hunks
git commit -m "fix: handle null input"
git add app.py
git commit -m "feat: add export button"
```

---

## 9. Restoring Files — `git restore` (Modern) vs `git checkout` (Old)

**Discard unstaged changes in working directory:**
```bash
git restore app.py              # revert to index version
git restore .                   # revert everything
```

**Unstage (copy from HEAD back to index):**
```bash
git restore --staged app.py
```

**Both (unstage AND discard working dir changes):**
```bash
git restore --source=HEAD --staged --worktree app.py
```

**Restore from a specific commit:**
```bash
git restore --source=abc1234 app.py
```

Old equivalents (still common):
```bash
git checkout -- app.py          # discard working dir changes
git reset HEAD app.py           # unstage
```

**Real example:**
```bash
echo "broken" > app.py
git restore app.py              # wipes your broken edit, back to index version
```

⚠️ **`git restore` is destructive** — unstaged changes are lost permanently (they were never in Git).

---

## 10. Removing Files — `git rm` vs `rm`

**Just delete with `rm`:**
```bash
rm file.txt
git status       # "deleted: file.txt" (unstaged deletion)
git add file.txt # stages the deletion (or `git rm file.txt` directly)
```

**`git rm`** deletes and stages in one step:
```bash
git rm file.txt
```

**Keep in working dir, remove from tracking:**
```bash
git rm --cached file.txt        # stop tracking, keep the file on disk
```

**Real use case:** You accidentally committed `.env`:
```bash
git rm --cached .env
echo ".env" >> .gitignore
git commit -m "stop tracking .env"
```

**Force remove a modified file:**
```bash
git rm -f file.txt              # required if file has staged changes
```

---

## 11. Moving/Renaming Files — `git mv`

```bash
git mv old.txt new.txt
```
Equivalent to:
```bash
mv old.txt new.txt
git rm old.txt
git add new.txt
```

Git doesn't really track renames — it detects them by similarity when diffing. But `git mv` stages the change cleanly.

**Real example:**
```bash
git mv src/utils.py src/helpers.py
git status
# renamed: src/utils.py -> src/helpers.py
```

---

## 12. The Index in Detail (Because It Explains the Working Dir)

The `.git/index` is a binary file listing:
- Path
- Blob SHA (content)
- Mode (100644 normal, 100755 executable, 120000 symlink)
- Stat info (mtime, size, inode)

`git status` works by:
1. Reading the index
2. For each file, stat() the working dir file
3. If mtime/size differ → read content, hash it, compare to index blob SHA
4. Report accordingly

You can inspect the index:
```bash
git ls-files               # all tracked files
git ls-files --stage       # with mode + SHA + stage number
git ls-files --others --exclude-standard   # untracked, non-ignored
git ls-files --modified    # changed vs index
```

---

## 13. Merge Conflicts in the Working Directory

During a merge/rebase conflict, Git writes **conflict markers** into the working file:

```python
<<<<<<< HEAD
print("main version")
=======
print("feature version")
>>>>>>> feature-branch
```

The working directory literally contains all three versions. You edit the file to resolve, then:
```bash
git add file.py
git commit
```

**See all versions:**
```bash
git diff                # combined diff showing conflicts
git checkout --ours file.py     # keep "our" side
git checkout --theirs file.py   # keep "their" side
git restore --staged --worktree file.py  # reset the file
```

**Use a merge tool:**
```bash
git mergetool
```

---

## 14. Stashing — Temporarily Shelving Working Dir Changes

```bash
git stash                    # save + clean working dir
git stash push -m "wip"      # with message
git stash -u                 # include untracked
git stash -a                 # include ignored too
git stash list
git stash pop                # restore + drop
git stash apply              # restore, keep stash
git stash drop stash@{0}
```

**Real scenario:** You're mid-feature and need to hotfix production:
```bash
git stash push -m "half-done feature"
git checkout main
# ... hotfix, commit ...
git checkout feature
git stash pop
```

**Partial stash:**
```bash
git stash push -p app.py     # choose which hunks to stash
```

---

## 15. Cleaning the Working Directory — `git clean`

Removes **untracked** files (dangerous but useful).

```bash
git clean -n            # dry run (always do this first!)
git clean -f            # remove untracked files
git clean -fd           # also directories
git clean -fdx          # also ignored files (nukes node_modules etc.)
git clean -i            # interactive
```

**Real scenario:** You ran a build that created junk:
```bash
git clean -nfd
# shows: would remove dist/, would remove tmp.log
git clean -fd
```

⚠️ `git clean` is **permanent** — untracked files are not in Git, so they're gone forever.

---

## 16. Working Directory with Submodules

Submodules are separate Git repos nested inside your working directory.

```bash
git submodule add https://github.com/lib/foo libs/foo
```

`libs/foo/` is its own working directory with its own `.git`. The parent repo tracks a **commit pointer**, not the contents.

```bash
git submodule update --init --recursive
cd libs/foo && git checkout main
```

---

## 17. Worktrees — Multiple Working Directories for One Repo

Git can check out **multiple branches simultaneously** into different folders:

```bash
git worktree add ../hotfix hotfix-branch
cd ../hotfix
# work on hotfix while main stays checked out elsewhere
```

List them:
```bash
git worktree list
```

Remove:
```bash
git worktree remove ../hotfix
git worktree prune
```

**Real use case:** You need to run tests on `main` while developing on a feature branch — without stashing.

---

## 18. How Git Decides a File "Changed"

Git's change detection (simplified):

```
1. Compare path in index → does file exist in working dir?
   - No → "deleted"
   - Yes → continue
2. Compare stat (mtime + size)
   - Same → assume unchanged (fast path)
   - Different → continue
3. Hash working file content → compare to index blob SHA
   - Same → actually unchanged (just touched), update stat cache
   - Different → "modified"
```

This is why:
```bash
touch app.py           # mtime changes, but content same → no diff
echo "" >> app.py      # content changes → diff
```

If mtimes are messed up (e.g., after a `git checkout` on a slow clock), Git may misdetect. Fix with:
```bash
git update-index --refresh
git status             # forces a full re-check
```

**Skip the stat cache entirely:**
```bash
git update-index --really-refresh
```

---

## 19. File Modes in the Working Directory

Git tracks one mode bit: **executable** (100755) vs not (100644).

```bash
chmod +x script.sh
git status             # "modified: script.sh" (mode change)
git diff               # "old mode 100644 / new mode 100755"
```

**Ignore mode changes (Windows↔Linux):**
```bash
git config core.filemode false
```

**Symlinks:** stored as mode 120000, content = target path.

---

## 20. Line Endings (CRLF/LF) — A Working Dir Trap

Windows uses `\r\n`, Unix uses `\n`. Git can normalize:

```bash
git config core.autocrlf true      # Windows: convert on checkout/commit
git config core.autocrlf input     # Mac/Linux: convert on commit only
git config core.autocrlf false     # no conversion
```

Better: use `.gitattributes`:
```
* text=auto
*.sh text eol=lf
*.bat text eol=crlf
*.png binary
```

Without this, your working dir may show "modified" for every file on a cross-platform team.

---

## 21. Sparse Checkout — Partial Working Directory

Check out only some directories:

```bash
git sparse-checkout init --cone
git sparse-checkout set src/ docs/
```

Now only `src/` and `docs/` appear in your working dir; everything else exists in Git but not on disk.

**Real use case:** Huge monorepo, you only work on one service.

**Disable:**
```bash
git sparse-checkout disable
```

---

## 22. Partial Clone / Shallow Clone Effects on Working Dir

```bash
git clone --depth 1 repo       # only latest commit
git clone --filter=blob:none   # lazy blobs
```

With partial clone, files may be **fetched on demand** when you open them — the working directory fills in lazily.

---

## 23. Detached HEAD and the Working Directory

```bash
git checkout abc1234
```
You're now in **detached HEAD**. The working directory reflects commit `abc1234`, but you're not on a branch. Commits you make here aren't on any branch and can be lost:

```bash
git switch -c rescue           # save your work to a branch
```

---

## 24. Recovering Lost Working Directory Changes

**Unstaged changes lost via `git restore`?** Usually gone — they were never in Git. But sometimes recoverable:

```bash
git fsck --lost-found          # finds dangling blobs
ls .git/lost-found/other/
```

If you ever `git add`ed them, they exist as blobs:
```bash
git fsck --unreachable | grep blob
git show <blob-sha>
```

**Staged then lost?** The blob is in `.git/objects`. Find it:
```bash
git fsck --lost-found
```

**Committed then lost branch?**
```bash
git reflog
git checkout <sha>
```

**Lesson:** `git add` is a safety net — even if you never commit, the content lives in `.git/objects` until GC.

---

## 25. Working Directory vs File System — Edge Cases

- **Case-insensitive FS (macOS/Windows):** `File.txt` and `file.txt` conflict. Fix:
  ```bash
  git config core.ignorecase true
  ```
- **Long paths on Windows:** enable `core.longpaths true`.
- **Case-only renames:** use two-step:
  ```bash
  git mv File.txt tmp && git mv tmp file.txt
  ```
- **Files with weird chars (`:` on Windows, newlines):** use `git ls-files -z` and `-z` flags.

---

## 26. Full End-to-End Walkthrough

Scenario: You clone a repo and make a mess. Let's trace everything.

```bash
git clone https://github.com/example/app
cd app

# 1. Working dir starts clean
git status                     # "nothing to commit, working tree clean"

# 2. Make changes
echo "bugfix" >> src/main.py
echo "notes" > TODO.txt

git status
# modified: src/main.py
# untracked: TODO.txt

# 3. Ignore the scratch file
echo "TODO.txt" >> .gitignore
git status                     # TODO.txt gone from list
git check-ignore -v TODO.txt   # shows which rule ignored it

# 4. Stage the bugfix
git add src/main.py
git diff                       # empty (working == index)
git diff --staged              # shows the bugfix

# 5. Edit more (now unstaged + staged)
echo "more" >> src/main.py
git diff                       # shows "more"
git diff --staged              # shows "bugfix"

# 6. Stage just the latest hunk interactively
git add -p src/main.py

# 7. Commit
git commit -m "fix: null handling in main"

# 8. Accidentally break something
echo "oops" > src/main.py
git restore src/main.py        # back to committed version

# 9. Rename a file
git mv src/old.py src/new.py

# 10. Clean up untracked junk
git clean -nfd                 # preview
git clean -fd                  # remove

# 11. Stash work-in-progress
echo "wip" >> src/main.py
git stash push -m "wip feature"
git status                     # clean again
git stash pop
```

You've now exercised every major working-directory concept.

---

## 27. Cheat Sheet

| Task | Command |
|------|---------|
| See status | `git status` / `git status -s` |
| Unstaged diff | `git diff` |
| Staged diff | `git diff --staged` |
| All diff vs HEAD | `git diff HEAD` |
| Stage file | `git add file` |
| Stage hunks | `git add -p file` |
| Unstage | `git restore --staged file` |
| Discard changes | `git restore file` |
| Remove tracked file | `git rm file` |
| Untrack but keep | `git rm --cached file` |
| Rename | `git mv old new` |
| Stash | `git stash push -m "msg"` |
| Clean untracked | `git clean -nfd` then `-fd` |
| List tracked | `git ls-files` |
| Why ignored? | `git check-ignore -v file` |
| Refresh stat cache | `git update-index --refresh` |

---

## 28. Mental Model

Think of the working directory as **the scratchpad**:

- **Git doesn't care about it until you ask** (`git status`, `git diff`, `git add`).
- **`git add` copies a snapshot** into the index — the working dir keeps changing independently.
- **`git restore` overwrites** the working dir from the index or HEAD — destructive to unstaged work.
- **Untracked files are invisible** to Git until added, and can never be recovered by Git if deleted.
- **The index is the bridge** — every working-dir → commit path goes through it.

Once you truly internalize "working dir vs index vs HEAD," every Git command becomes obvious: each one just moves data between these three trees.

---

## 29. Practice Exercises

1. Create a file, add it, edit it, and observe `git diff` vs `git diff --staged`.
2. Add a file to `.gitignore` **after** committing it — see that it's still tracked. Then `git rm --cached` it.
3. Use `git add -p` to split one file's changes into two commits.
4. Delete a file with `rm`, then `git restore` it.
5. `git stash` mid-edit, switch branches, come back, `git stash pop`.
6. Run `git clean -nfd` in a repo with build junk — observe what would be removed.
7. Use `git worktree add` to check out two branches simultaneously.
8. Create a merge conflict and resolve it by editing the working-dir file.
9. Set `core.autocrlf` and observe line-ending diffs.
10. Use `git fsck --lost-found` after discarding a staged change to recover it.

---

If you want, I can next go equally deep on the **Index/Staging Area**, **HEAD and refs**, **git reset vs restore vs revert**, or **merge/rebase internals** — all of which build directly on this working-directory foundation.