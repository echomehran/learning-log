`git cherry-pick` is the "surgical" counterpart to merge and rebase. Where merge joins entire histories and rebase replays a sequence, **cherry-pick copies the change introduced by one specific commit and applies it as a brand-new commit on your current branch** . It is the canonical tool for backporting a single hotfix to a release branch without dragging along everything else from `main`.

### 🍒 Core Mechanism and the Golden Rule

The command requires a **clean working tree** (no uncommitted changes) . When it succeeds, it creates a new commit with a **new SHA** on your current branch—the original commit remains untouched in its original branch.

The classic use case is a hotfix backport: a bug is fixed on `main` (commit `9f2c1ab`), and that exact fix must reach `release/2.4`. You switch to the release branch and run `git cherry-pick 9f2c1ab`. Use `-x` to record the original SHA in the new commit message—future maintainers will thank you .

**Golden rule:** Cherry-picking the same change into two branches that later merge produces duplicate-change conflicts. Treat it as a targeted tool, not a primary workflow .

### 🛠️ Handling Conflicts (The State Machine)

When a change does not apply cleanly, cherry-pick enters a paused state:
1. `HEAD` stays at the last successful commit.
2. `CHERRY_PICK_HEAD` is set to the problematic commit.
3. Conflicted files contain the standard `<<<<<<<` / `=======` / `>>>>>>>` markers .

Your control flow from here:

| Command | What it does |
| :--- | :--- |
| Resolve + `git add` + `git cherry-pick --continue` | Finalize the conflicted commit |
| `git cherry-pick --skip` | Drop this commit and continue with the next in the sequence |
| `git cherry-pick --abort` | Cancel entirely, returning to the pre-cherry-pick state |
| `git cherry-pick --quit` | Stop the sequence but **keep** the changes made so far (no rollback) |

### 🚩 Complete Flag Reference

Every option from `git-cherry-pick(1)`, grouped for clarity.

**Commit Selection & Ranges**

| Flag | Description |
| :--- | :--- |
| `<commit>...` | One or more commits. Ranges like `master~4..master~2` feed all commits into a single walk  |
| `-n`, `--no-commit` | Apply changes to working tree and index **without committing**. Essential for squashing multiple picks into one  |

**Merge Commit Handling**

| Flag | Description |
| :--- | :--- |
| `-m <parent>`, `--mainline <parent>` | **Required** to cherry-pick a merge commit. Parent `1` is the branch merged *into* (mainline); parent `2` is the branch merged *from*  |

**Message & Metadata**

| Flag | Description |
| :--- | :--- |
| `-e`, `--edit` | Edit the commit message before committing  |
| `-x` | Append `(cherry picked from commit <sha>)` to the message. **Do not use** for private-branch picks—it's noise for the recipient. **Do use** for public backports  |
| `--cleanup=<mode>` | Message cleanup mode (`strip`, `whitespace`, `verbatim`, `scissors`)  |
| `-s`, `--signoff` | Add a `Signed-off-by` trailer  |

**Signing**

| Flag | Description |
| :--- | :--- |
| `-S[<keyid>]`, `--gpg-sign[=<keyid>]` | GPG-sign the resulting commit. `keyid` must stick to the flag with no space  |
| `--no-gpg-sign` | Override any `commit.gpgSign` config for this operation  |

**Behavior Modifiers**

| Flag | Description |
| :--- | :--- |
| `--ff` | If `HEAD` equals the picked commit's parent, fast-forward instead of creating a new commit  |
| `--allow-empty` | Preserve commits that were initially empty (by default, fails)  |
| `--allow-empty-message` | Allow commits with empty messages  |
| `-r` | **No-op**. Historically disabled the old default `-x`; now `-x` is off by default anyway  |

**Sequence Control**

| Flag | Description |
| :--- | :--- |
| `--continue` | Resume after conflict resolution |
| `--skip` | Drop the current commit, proceed to next |
| `--abort` | Roll back everything to pre-cherry-pick |
| `--quit` | Forget the sequence but **keep** changes made so far |

### 🔍 Advanced Techniques

**1. Squashing multiple picks with `-n`**
To apply three commits from another branch as a single new commit on yours:
```bash
git cherry-pick -n abc1234 def5678 ghi9012
git commit -m "Combined backport: fix A, B, C"
```
The `-n` flag applies each change to index without committing, letting you inspect and craft one final commit .

**2. Cherry-picking merge commits (`-m`)**
A merge commit has two parents. Parent `1` is the branch you merged *into* (e.g., `main`); parent `2` is the branch merged *from* (e.g., `feature`). To apply the *effect* of the feature onto your release branch, use `-m 1`:
```bash
git cherry-pick -m 1 <merge-commit-sha>
```
This replays the diff relative to the mainline parent .

**3. Scripted bulk picks from a file list**
```bash
git rev-list --reverse master -- README | git cherry-pick -n --stdin
```
Applies all commits touching `README` to the index, then you inspect and make one commit .

**4. `--ff` for linear histories**
If your branch is a direct ancestor of the target commit and history is linear, `--ff` fast-forwards instead of creating a new commit. Rarely needed but exists for completeness .

### ⚠️ Pitfalls and Discipline

- **Duplicate SHAs by design**: Cherry-pick creates *new* commits. The same logical change now exists as two different commits. If the branches eventually merge, Git may see a conflict between "identical changes" .
- **`-x` is a backporting signal**: Use it whenever picking between public branches. Omit it for private or temporary picks .
- **Never cherry-pick onto a dirty tree**: Commit or stash first, or the command refuses to start .
- **`--abort` vs `--quit`**: `--abort` is your undo; `--quit` is "stop, but keep what I have." Choose deliberately.

### 📌 One-Line Summary

`git cherry-pick` copies the *change* (not the identity) of specific commits onto your branch. The essential flags are **`-x`** (provenance for backports) and **`-n`** (squash multiple picks). The golden rule: use it surgically for hotfixes and backports, never as a substitute for merging entire branches.