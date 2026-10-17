# Git Tags: The Complete Deep Dive

This guide merges conceptual understanding, practical examples, and the exhaustive flag reference into one coherent document. Read it top-to-bottom for a full education, or jump to the reference tables when you need a specific flag.

---

## Part 1 — The Core Concept: What Is a Tag, Really?

A Git tag is a **fixed reference** pointing to a specific object (almost always a commit). Unlike branches, which move forward as you commit, tags are anchored. There are two kinds, and the difference is fundamental — not cosmetic.

### Lightweight Tag
Just a file in `.git/refs/tags/<name>` containing a commit SHA. It carries **no** tagger name, no date, no message, no signature. It is a bare pointer.

### Annotated Tag
A **full Git object** stored in the object database (`.git/objects/`). It contains:
- the tagger's name and email,
- a timestamp,
- an independent tag message,
- optionally a GPG signature,
- a pointer to the target object.

Because it's a real object, it has its own SHA, it's checksummed, and it can be signed and verified. **The official `git-tag(1)` manual recommends annotated tags for releases** precisely because they preserve a complete audit trail.

> **Rule of thumb:** annotated (`-a`, `-s`, or implicit via `-m`/`-F`) for anything shared or released; lightweight only for private, temporary markers.

---

## Part 2 — Practical Workflow with Examples

### 2.1 Creating Tags

**Annotated tag on HEAD (best practice for releases):**
```bash
git tag -a v1.0.0 -m "Release version 1.0.0: Initial stable release"
```

**Annotated tag on a specific commit:**
```bash
git tag -a v1.0.1 -m "Hotfix: security patch" abc1234
```

**Lightweight tag (temporary/private only):**
```bash
git tag v1.0.0-beta
```

**Implicit annotated tag** — note that `-m` (or `-F`) *by itself* creates an annotated tag even without `-a`:
```bash
git tag v1.2.0 -m "This is annotated because -m was given"
```

**Signed tag (for high-security releases):**
```bash
git tag -s v2.0.0 -m "Signed release"
git tag -v v2.0.0          # verify later
```

**Message from a file (release notes):**
```bash
git tag -a v1.1.0 -F RELEASE_NOTES.md
git tag -a v1.1.0 -F -      # read from stdin
```

**Force-move a tag (only safe if NOT yet pushed):**
```bash
git tag -f v1.0.0 abc1234
```

### 2.2 Viewing and Inspecting

```bash
git tag                              # list all tags
git tag -l "v1.*"                    # filter by pattern
git tag -n                           # list with 1 line of annotation
git tag -n5                          # list with 5 lines of annotation
git show v1.0.0                      # full details (tagger, date, message, commit)
git describe --tags --abbrev=0       # latest tag reachable from HEAD
```

`git show` on an **annotated** tag prints tagger, date, message, then the commit. On a **lightweight** tag it just prints the commit — the absence of metadata is the tell.

### 2.3 Comparing and Diffing

```bash
git diff v1.0.0 v1.1.0                       # code diff between releases
git log v1.0.0..v1.1.0 --oneline             # commits between releases
git log --oneline --decorate --graph --all   # visualize tags on the graph
```

### 2.4 Checking Out a Tag (Detached HEAD)

```bash
git checkout v1.0.0
# You are now in "detached HEAD" — you can build and inspect,
# but cannot commit onto a branch without creating one:
git checkout -b hotfix/v1.0.1 v1.0.0
```

### 2.5 Pushing Tags to a Remote

**`git push` does NOT push tags by default.** This is the single most common surprise.

```bash
git push origin v1.0.0            # push one specific tag
git push origin --follow-tags     # push annotated tags reachable from pushed commits
git push origin --tags            # push ALL local tags (dangerous — includes temp/lightweight)
```

Prefer `--follow-tags` in CI/CD: it pushes only annotated tags that belong to the commits being pushed.

### 2.6 Deleting Tags

**Local:**
```bash
git tag -d v1.0.0-beta
```

**Remote:**
```bash
git push origin --delete v1.0.0-beta      # modern syntax
git push origin :refs/tags/v1.0.0-beta    # legacy: push an empty ref
```

> If a branch with the same name exists, you **must** use `refs/tags/` to disambiguate, otherwise Git errors with "matches more than one".

### 2.7 Fetching Tags

```bash
git fetch --tags                 # fetch ALL tags (not just those reachable from fetched commits)
git fetch origin tag v1.0.0      # fetch one specific tag
```

By default, `git fetch` only brings tags pointing at commits already fetched. Use `--tags` to guarantee completeness.

---

## Part 3 — The One Immutable Rule: Re-tagging

The official manual has a dedicated section, **"On Re-tagging."** This is the most important piece of engineering discipline in tag management.

**Once a tag is pushed to a shared remote, treat it as immutable.**

| Situation | What to do |
| :--- | :--- |
| **Not yet pushed** | `git tag -f <name>` freely. No problem. |
| **Already pushed — sensible** | Admit the mistake, ship `v1.0.1`. Avoids the disaster where two people's `v1.0.0` point at different code. |
| **Already pushed — insane** | Force-push with `-f`, then tell **every** collaborator to `git tag -d <name>` and re-fetch. Breaks the "tags are eternal" trust model. Strongly discouraged. |

---

## Part 4 — Complete Flag Reference

Every option from `git-tag(1)`, grouped by function.

### 4.1 Creating and Annotating

| Flag | Description |
| :--- | :--- |
| `-a`, `--annotate` | Create an unsigned annotated tag object. |
| `-m <msg>`, `--message=<msg>` | Use `<msg>` as the tag message. Multiple `-m` concatenate as separate paragraphs. **Implies `-a`** if no `-a`/`-s`/`-u` given. |
| `-F <file>`, `--file=<file>` | Read the message from `<file>`; `-` means stdin. **Implies `-a`.** |
| `-e`, `--edit` | Further edit the message supplied via `-F` or `-m`. |
| `-s`, `--sign` | Create a GPG-signed annotated tag. |
| `-u <key-id>`, `--local-user=<key-id>` | Sign with the given GPG key. Implies `-s`. |
| `--cleanup=<mode>` | Message cleanup: `verbatim` (no change), `whitespace` (strip leading/trailing blanks), `strip` (default; also remove comment lines). |
| `--trailer <token>[(=||:)<value>]` | Append a trailer (like `Signed-off-by`) to the message; extractable via `%(trailers)` in `--format`. |
| `-f`, `--force` | Replace an existing tag of the same name. Safe only before pushing. |
| `--create-reflog`, `--no-create-reflog` | Create / suppress a reflog for the tag (tracks tag movements). |

### 4.2 Listing and Filtering

| Flag | Description |
| :--- | :--- |
| `-l <pattern>`, `--list[=<pattern>]` | List tags; supports wildcards. If a pattern is given, `-l` is **mandatory**. |
| `-n[<num>]` | When listing, show `<num>` lines of annotation (default 1). |
| `--contains [<commit>]` | List only tags that contain the given commit (default `HEAD`). |
| `--no-contains [<commit>]` | List only tags that do **not** contain the commit. |
| `--merged [<commit>]` | List tags whose target is reachable from `<commit>`. |
| `--no-merged [<commit>]` | List tags whose target is **not** reachable. |
| `--points-at <object>` | List only tags pointing at `<object>`. |
| `--sort=<key>` | Sort by `<key>` (e.g. `-version:refname` for descending semantic-version order). Repeatable; last is primary. Default: `tag.sort` config, else lexicographic. |
| `--format=<format>` | Custom output using `%(fieldname)` placeholders (same syntax as `git for-each-ref`). |
| `--color[=<when>]` | `always` / `never` / `auto`. |
| `-i`, `--ignore-case` | Case-insensitive sorting and filtering. |
| `--omit-empty` | Do not print a newline for empty formatted results. |
| `--column[=<options>]`, `--no-column` | Column layout for lists. Only effective when annotation lines are not shown. |

### 4.3 Deleting and Verifying

| Flag | Description |
| :--- | :--- |
| `-d`, `--delete` | Delete one or more local tags. |
| `-v`, `--verify` | Verify the GPG signature of a tag. |

### 4.4 Positional Arguments

| Argument | Description |
| :--- | :--- |
| `<tagname>` | Tag to create, delete, or describe. |
| `<commit>`, `<object>` | Target object; defaults to `HEAD`. |

---

## Part 5 — Advanced Techniques and Edge Cases

**1. Detached HEAD from a tag**
`git checkout v1.0.0` leaves you in detached HEAD. Build and inspect freely; to fix a bug on an old release, branch off: `git checkout -b hotfix/v1.0.1 v1.0.0`.

**2. Signed tags and supply-chain security**
`git tag -s` plus `git tag -v` proves both authenticity (the right person signed it) and integrity (the tag content was not tampered with). This is the standard for open-source releases.

**3. Push vs fetch — the asymmetry**
- `git push` skips tags by default. Use explicit names or `--follow-tags`.
- `git fetch` brings only tags reachable from fetched commits. Use `--tags` for completeness.
- Never blanket `git push --tags` in CI — it will publish temporary local tags.

**4. Semantic-version sorting (CI/CD essential)**
```bash
git tag -l --sort=-version:refname "v*" | head -n1     # latest version
```
The leading `-` reverses order, so the first line is the newest release. Underlies most "what's the latest tag" scripts.

**5. Remote deletion with name collision**
`git push origin --delete v1.0.0` fails if a branch `v1.0.0` also exists. Use `git push origin :refs/tags/v1.0.0` to force the tag interpretation.

**6. Finding which tags a commit belongs to**
```bash
git tag --contains abc1234
git tag --points-at abc1234
```

**7. Scriptable output**
```bash
git tag -l --format='%(refname:short) %(taggername) %(taggerdate:iso)'
```
Every field of `for-each-ref` is available — invaluable for release dashboards.

---

## Part 6 — Best Practices Summary

1. **Use annotated tags for all shared work.** `-a` or `-s`; lightweight only for private/temporary markers.
2. **Follow SemVer (`vMAJOR.MINOR.PATCH`).** Enables `--sort=version:refname` and predictable dependency tooling.
3. **Never move a pushed tag.** If broken, ship a new patch. Immutability is the whole point.
4. **Push explicitly.** `git push origin <tag>` or `--follow-tags`; avoid blanket `--tags` in CI.
5. **Sign release tags** in security-sensitive projects; automate `git tag -v` in deployment gates.
6. **Automate "latest tag"** with `git describe --tags --abbrev=0` or `--sort=-version:refname`.

---

## One-Line Summary

Git tags = **annotated tags + immutable after push**. Every flag in this document serves one of three goals: building complete annotated tags (`-a/-m/-F/-s/-u/--cleanup/--trailer`), retrieving them efficiently (`-l/-n/--contains/--merged/--points-at/--sort/--format`), or correcting local mistakes before they become permanent (`-f`, `-d`). The official stance on post-push correction: **don't — cut a new version.**