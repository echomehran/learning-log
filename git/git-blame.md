# Git Blame: A Deep and Complete Guide

`git blame` is one of Git's most misunderstood commands. Most people think it just shows "who wrote this line" — but it's actually a forensic tool for understanding *history, intent, and evolution* of code. Let's go deep.

---

## 1. What `git blame` Actually Does

At its core, `git blame` annotates each line of a file with:
- The **commit hash** that last modified that line
- The **author** of that commit
- The **date** of that commit
- The **line number** in the original commit

It answers: *"For each line in this file, which commit introduced or last changed it?"*

```bash
git blame filename.txt
```

Example output:
```
a1b2c3d4 (Alice  2024-01-15 10:23:45 -0500 1) def calculate_tax(income):
e5f6g7h8 (Bob    2024-02-03 14:11:22 -0500 2)     rate = 0.15 if income < 50000 else 0.25
a1b2c3d4 (Alice  2024-01-15 10:23:45 -0500 3)     return income * rate
```

Each line tells you *who last touched it and when*.

---

## 2. The Basic Anatomy of Output

```
<commit-hash> (<author> <date> <timezone> <line-number>) <content>
```

- **`^` prefix** on a hash (e.g., `^a1b2c3d`) means that line came from the **initial commit** (the root commit).
- **`00000000`** means the line is **uncommitted** (working directory change).

---

## 3. Real-World Example: Finding Who Broke Production

Imagine you're debugging a production issue in `payments.py`:

```python
def process_payment(amount):
    fee = amount * 0.029 + 0.30
    return amount - fee
```

You run:
```bash
git blame payments.py
```

Output:
```
f3a91c2 (Alice  2024-03-01 09:12:00 -0800 1) def process_payment(amount):
b7d44e1 (Bob    2024-06-14 16:45:11 -0700 2)     fee = amount * 0.029 + 0.30
f3a91c2 (Alice  2024-03-01 09:12:00 -0800 3)     return amount - fee
```

Bob changed the fee logic on June 14. You inspect it:
```bash
git show b7d44e1
```

Commit message: *"Update payment fee to match Stripe's new pricing"*. Now you understand the intent.

---

## 4. Restricting to a Line Range

You don't always want the whole file. Use `-L`:

```bash
git blame -L 10,20 payments.py       # lines 10 through 20
git blame -L 10,+15 payments.py      # 15 lines starting at line 10
git blame -L 50 payments.py          # line 50 to end
git blame -L :process_payment payments.py   # from function definition
```

The `:function_name` syntax is powerful — it finds the function and blames until the next function.

---

## 5. Ignoring Whitespace and Moves

Real-world code gets reformatted constantly. Blame would falsely attribute lines to the reformatter. Use:

```bash
git blame -w file.py        # ignore whitespace changes
git blame -M file.py        # detect moved/copied lines within the file
git blame -C file.py        # detect lines copied from other files
git blame -C -C file.py     # also detect from files modified in the same commit
git blame -C -C -C file.py  # detect from any commit (slow, thorough)
```

**Real example:** Someone runs `black` on the codebase. Without `-w`, blame shows their reformat commit for every line. With `-w`, blame shows the *actual* authors.

---

## 6. Ignoring Specific Commits (`.git-blame-ignore-revs`)

For massive refactors/formatting commits, you can tell Git to ignore them permanently.

**Step 1:** Create `.git-blame-ignore-revs`:
```
# Migration to black formatting
a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0
# Renamed all foo_* to bar_*
1234567890abcdef1234567890abcdef12345678
```

**Step 2:** Configure Git:
```bash
git config blame.ignoreRevsFile .git-blame-ignore-revs
```

Now `git blame` automatically skips those commits.

---

## 7. Machine-Readable Output (`--porcelain`)

For scripts and tooling:

```bash
git blame --porcelain file.py
```

Outputs structured data:
```
a1b2c3d4e5f6... 1 1 3
author Alice
author-mail <alice@example.com>
author-time 1705329825
author-tz -0500
committer Alice
committer-mail <alice@example.com>
committer-time 1705329825
committer-tz -0500
summary Add tax calculator
filename calculator.py
	def calculate_tax(income):
```

Useful for IDE integrations and automation.

Also useful:
```bash
git blame --line-porcelain file.py   # full info for every line
git blame --incremental file.py      # diff-friendly
```

---

## 8. Custom Formatting with `--date` and format flags

```bash
git blame --date=short file.py       # 2024-01-15
git blame --date=relative file.py    # 2 weeks ago
git blame --date=iso file.py         # full ISO timestamp

git blame -s file.py                 # suppress author name (just hash)
git blame -e file.py                 # show email instead of name
git blame -f file.py                 # show filename in output
git blame -n file.py                 # show original line number
git blame -l file.py                 # show long hash
```

---

## 9. Real-World Workflow: The "Why" Investigation

You find a mysterious magic number in `auth.js`:

```javascript
const SESSION_TIMEOUT = 873;
```

```bash
git blame -L :SESSION_TIMEOUT auth.js
```

```
c4d5e6f (Carol 2023-11-02 11:30:00 -0500 12) const SESSION_TIMEOUT = 873;
```

```bash
git show c4d5e6f
```

Commit message: *"Set session timeout to match legacy system: 14m33s"*

Now you know: it's not arbitrary — it mirrors a legacy system.

---

## 10. Following a Line Across Renames and History

Blame tells you the *last* commit, but sometimes you want the full lineage.

```bash
git log -L :SESSION_TIMEOUT:auth.js
```

This shows every change to that function/line over time.

Also:
```bash
git blame -L 12,12 --reverse auth.js   # find when line was first introduced
```
`--reverse` walks history forward from the root, showing the *earliest* commit touching each line.

---

## 11. Blaming a Specific Commit

By default, blame uses `HEAD`. Blame a historical state:

```bash
git blame v2.0.0 -- payments.py
git blame abc1234 -- payments.py
git blame HEAD~5 -- payments.py
```

This shows who last touched each line *as of that commit*.

---

## 12. Blaming a Range in a Branch

```bash
git blame main -- file.py
git blame origin/main -- file.py
```

---

## 13. Practical Example: Audit Trail for Compliance

You're asked: "Who approved changes to the encryption key rotation logic?"

```bash
git blame -L :rotate_keys crypto.py
```

```
9a8b7c6 (Dave  2024-04-12 08:00:00 -0400 45) def rotate_keys():
3e2f1d0 (Eve   2024-05-20 15:22:00 -0400 46)     new_key = generate_key()
9a8b7c6 (Dave  2024-04-12 08:00:00 -0400 47)     store_key(new_key)
```

Eve modified line 46. You check the PR:
```bash
git log -1 3e2f1d0
```

Message: *"Rotate keys on every request — approved by security team (ticket SEC-442)"*. Full audit trail in seconds.

---

## 14. Understanding Edge Cases

**Case A: Merged commits**
Blame attributes lines to the commit *that introduced them*, not the merge. If a line was cherry-picked, blame shows the original commit.

**Case B: Reverted lines**
If a line was changed then reverted, blame shows the revert commit — but the *content* matches the original. Use:
```bash
git blame --ignore-rev <revert-commit> file.py
```

**Case C: Uncommitted changes**
```
00000000 (Not Committed Yet 2024-... 1) new line
```
The line exists only in your working directory.

**Case D: Binary files**
Blame doesn't work — Git says `"Cannot blame binary file"`.

---

## 15. Combining Blame with Other Tools

**Find the commit, then dig deeper:**
```bash
git blame -L 42,42 app.py
git show <hash>
git log --follow -- app.py   # follow renames
```

**Blame + grep for who touched what keyword:**
```bash
git blame file.py | grep "TODO"
```
Shows every TODO and who wrote it.

**Blame a deleted line (from a diff):**
```bash
git log -S "old_code_string" --oneline
```
`-S` (pickaxe) finds commits that added/removed a string.

---

## 16. Common Flags Cheat Sheet

| Flag | Purpose |
|------|---------|
| `-L <start>,<end>` | Blame a line range |
| `-L :funcname` | Blame a function |
| `-w` | Ignore whitespace |
| `-M` | Detect moved lines (within file) |
| `-C` | Detect copied lines (across files) |
| `-C -C -C` | Aggressive copy detection |
| `--reverse` | Show original (earliest) commit per line |
| `-s` | Suppress author (hash only) |
| `-e` | Show email |
| `-f` | Show filename |
| `-n` | Show original line number |
| `-l` | Long hash |
| `--porcelain` | Machine-readable |
| `--date=short\|relative\|iso` | Date format |
| `--ignore-rev <hash>` | Skip a commit |
| `--ignore-revs-file <file>` | Skip many commits |

---

## 17. Full End-to-End Walkthrough

Scenario: A bug appears in `utils.py` line 88.

```bash
# 1. Who touched it?
git blame -L 88,88 utils.py
# → 7f3e2a1 (Priya 2024-08-01)

# 2. What was the change?
git show 7f3e2a1

# 3. Was it part of a larger change?
git log -1 --stat 7f3e2a1

# 4. What did the line look like before?
git show 7f3e2a1^:utils.py | sed -n '88p'

# 5. Was the file renamed at some point?
git log --follow --oneline -- utils.py

# 6. Was formatting hiding the real author?
git blame -w -M -C -L 88,88 utils.py

# 7. Full history of that line:
git log -L 88,88:utils.py
```

You now have a complete forensic picture.

---

## 18. Mental Model to Remember

Think of `git blame` as:
- **A lens** — you point it at a line and see the last change.
- **A filter** — flags (`-w`, `-M`, `-C`, `--ignore-rev`) remove noise.
- **A gateway** — its output (a hash) is the entry point for `git show`, `git log`, and deeper investigation.

Blame alone rarely gives the full story. **Blame + `git show` + `git log -L`** is the real trio.

---

## 19. Practice Exercises

Try these in any repo (or create a scratch one):

1. Blame a file, then `git show` the top 3 commits.
2. Use `-w` on a file that was reformatted — compare output.
3. Create a `.git-blame-ignore-revs` with a formatting commit and confirm it's skipped.
4. Use `-L :function_name` on a real function.
5. Use `--reverse` on a line to find its true origin.
6. Use `git log -L <line>:<file>` to trace full evolution.
7. Use `--porcelain` and pipe to `grep author`.

---

## 20. Bonus: GUI Alternatives

- `git blame` in **VS Code** (GitLens) — inline blame, hover for commit
- **GitHub** — click "Blame" on any file
- **tig blame file.py** — terminal UI
- **git gui blame** — built-in GUI

They all wrap the same underlying data.

---

If you'd like, I can go even deeper into specific areas: `--porcelain` scripting, `.git-blame-ignore-revs` CI setup, or building your own blame visualizer. Just say the word.