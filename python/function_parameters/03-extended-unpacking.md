# Extended Unpacking

Standard unpacking binds a fixed number of names to an iterable of exactly the same length. **Extended unpacking** relaxes that constraint — a `*rest` name captures whatever isn't claimed by the other names, letting you split an iterable into "the first few" and "everything else," or "the first and last" and "the middle," without slicing.

---

## From Slicing to Starred Unpacking

The classic way to split a list into its first element and "everything else" is with slicing:

```python
l = [1, 2, 3, 4, 5, 6]

a = l[0]
b = l[1:]

print(a)   # 1
print(b)   # [2, 3, 4, 5, 6]
```

Unpacking simplifies this slightly:

```python
a, b = l[0], l[1:]
```

But the `*` operator does the same thing without any indexing:

```python
a, *b = l

print(a)   # 1
print(b)   # [2, 3, 4, 5, 6]
```

This is the central idea: **`*name` collects all the elements not bound to other names**, into a **list**.

### One `*` per pattern level

```python
a, *b, *c = [1, 2, 3, 4]
# SyntaxError: two starred expressions in assignment
```

Two stars in one pattern would leave the split ambiguous — there'd be no rule for deciding how many elements go to `b` versus `c`. One star per pattern level is the rule. (Nesting introduces multiple levels — see below.)

---

## Any Iterable Works

Extended unpacking operates on **iteration order**, not on indexing. So it works on anything iterable:

### Tuples

```python
a, *b = -10, 5, 2, 100

print(a)   # -10
print(b)   # [5, 2, 100]
```

Note that even though the right-hand side is a tuple, `b` is a **list**. The starred name always produces a list, regardless of the source type.

### Strings

```python
a, *b = 'python'

print(a)   # 'p'
print(b)   # ['y', 't', 'h', 'o', 'n']   ← characters, in a LIST
```

When unpacking a string, `*b` captures **characters** — not a substring. If you need the substring back:

```python
b = ''.join(b)   # 'ython'
```

This matters in practice: `a, *b = s` is not equivalent to `a = s[0]; b = s[1:]`. The types differ (`str` vs. `list`), and the semantic intent differs (unpacking vs. slicing).

---

## First, Second, Last, and the Middle

Slicing is clumsy for extracting head, second, tail, and middle:

```python
s = 'python'
a, b, c, d = s[0], s[1], s[2:-1], s[-1]

print(a)   # p
print(b)   # y
print(c)   # tho
print(d)   # n
```

Extended unpacking makes it direct:

```python
a, b, *c, d = s

print(a)   # p
print(b)   # y
print(c)   # ['t', 'h', 'o']
print(d)   # n
```

The `*c` sits in the middle. Python fills the names around it by working from both ends:

- **Names before `*c`** are filled left-to-right from the start (`a`, `b`).
- **Names after `*c`** are filled right-to-left from the end (`d`).
- **`*c`** captures the remainder.

### The binding algorithm

```text
    s = 'p'  'y'  't'  'h'  'o'  'n'
         │    │    │    │    │    │
         ▼    ▼    ▼    ▼    ▼    ▼
    a,   b,   *c,              d
    
    ┌───┐ ┌───┐ ┌─────────────┐ ┌───┐
    │ p │ │ y │ │ ['t','h','o']│ │ n │
    └───┘ └───┘ └─────────────┘ └───┘
      ↑     ↑          ↑          ↑
      │     │          │          └── from the end
      │     │          └───────────── whatever remains
      │     └──────────────────────── from the start
      └────────────────────────────── from the start
```

This works regardless of where the star sits in the pattern — front, middle, or end — as long as there's only one star per level.

### Reassembling the captured characters

Since `c` is a list of characters, not a string:

```python
c = ''.join(c)   # 'tho'
```

A one-line conversion when you need the substring back.

---

## Unpacking on the Right-Hand Side

The `*` also works when **constructing** collections — unpacking an iterable into a new container.

### Building lists

```python
l1 = [1, 2, 3]
l2 = [4, 5, 6]

l = [*l1, *l2]
print(l)   # [1, 2, 3, 4, 5, 6]
```

The two iterables are spliced into a single list. This is not just a convenience for lists — any iterable can be unpacked into any collection type that supports it:

```python
l1 = [1, 2, 3]
s = 'ABC'

l = [*l1, *s]
print(l)   # [1, 2, 3, 'A', 'B', 'C']
```

The string's characters are spliced in just like list elements — because both are iterable.

---

## Sets and Dicts: With Caveats

Unpacking works with unordered containers too, but with important limitations.

### Sets: unpacking works, order doesn't

```python
s = {10, -99, 3, 'd'}

for c in s:
    print(c)
# 10
# 3
# d
# -99   ← not the order you might expect
```

The insertion order is not preserved — sets are backed by hash tables, and iteration order is determined by the hash layout, not by how you wrote the literal.

Unpacking a set works mechanically:

```python
a, b, *c = s
print(a)   # 10 (some element)
print(b)   # 3  (another)
print(c)   # ['d', -99] (the rest)
```

…but assigning "first" and "last" is meaningless when there's no defined order. **Don't rely on which name gets which element.**

### The one genuinely useful set idiom

```python
s = {10, -99, 3, 'd'}
*a, = s
print(a)   # [10, 3, 'd', -99]
```

The trailing comma is essential — it makes `a,` a **pattern** (unpacking one starred name into a list), rather than an assignment of `*a` (which isn't valid on the left-hand side of `=` by itself).

This unpacks the set into a **list**, which *does* have a defined order. It's the idiomatic way to convert a set to a list — useful when you need to pass set elements somewhere that requires a list, or when combining sets via unpacking (below).

### Merging sets

Sets don't support `+`:

```python
s1 = {1, 2, 3}
s2 = {3, 4, 5}

s1 + s2
# TypeError: unsupported operand type(s) for +: 'set' and 'set'
```

The method form works:

```python
s1.union(s2)
# {1, 2, 3, 4, 5}
```

For four sets:

```python
s1 = {1, 2, 3}
s2 = {3, 4, 5}
s3 = {5, 6, 7}
s4 = {7, 8, 9}

s1.union(s2, s3, s4)          # ✅ method takes multiple args
s1.union(s2).union(s3).union(s4)   # ✅ but chaining is uglier
```

Or with unpacking:

```python
{*s1, *s2, *s3, *s4}
# {1, 2, 3, 4, 5, 6, 7, 8, 9}
```

Each set is **unpacked directly into a new set literal** — a form that scales gracefully and reads uniformly with list and dict merging. Both approaches produce the same result; the unpacking form is often preferred for consistency.

### Dicts: `*` unpacks keys, `**` unpacks items

For dictionaries, `*` unpacks **keys only**:

```python
d1 = {'key1': 1, 'key2': 2}
d2 = {'key2': 3, 'key3': 3}

[*d1, *d2]
# ['key1', 'key2', 'key2', 'key3']   ← keys, not pairs
```

To unpack **key-value pairs**, use `**`:

```python
{**d1, **d2}
# {'key1': 1, 'key2': 3, 'key3': 3}
```

### The overwrite rule

Notice what happened to `key2`: it appears in both `d1` and `d2`, but the result kept only `d2`'s value (3). This is **left-to-right application with later-wins**:

```python
{**d1, **d2}   # key2 = 3   (d2's value, applied second)
{**d2, **d1}   # key2 = 2   (d1's value, applied second)
```

Order matters. Reversing the unpacking order reverses which value survives.

### Mixing with literals

Unpacking can be interleaved with literal key-value pairs inside a dict literal:

```python
{'a': 1, 'b': 2, **d1, **d2, 'c': 3}
# {'a': 1, 'b': 2, 'key1': 1, 'key2': 3, 'key3': 3, 'c': 3}
```

The same overwrite rule applies everywhere:

```python
{'key1': 100, **d1, **d2, 'key3': 200}
# {'key1': 1, 'key2': 3, 'key3': 200}
```

Tracing carefully:

```text
    Step 1:  'key1': 100            →  key1 = 100
    Step 2:  **d1                   →  key1 = 1 (overwrites), key2 = 2
    Step 3:  **d2                   →  key2 = 3 (overwrites), key3 = 3
    Step 4:  'key3': 200            →  key3 = 200 (overwrites)
    
    Result:  {'key1': 1, 'key2': 3, 'key3': 200}
```

The general rule: **entries are applied left to right; later entries overwrite earlier ones with the same key.** This is identical to `dict.update()` semantics.

### The config idiom

This behavior makes `{**defaults, **overrides}` the idiomatic way to merge configuration dicts:

```python
defaults = {'host': 'localhost', 'port': 8000}
overrides = {'port': 9000}
config = {**defaults, **overrides}
# {'host': 'localhost', 'port': 9000}
```

Later keys override earlier ones — exactly what "apply overrides" means.

---

## Nested Unpacking

Patterns on the left-hand side can be **nested**, mirroring the structure of the right-hand side:

```python
a, b, (c, d) = [1, 2, ['X', 'Y']]

print(a)   # 1
print(b)   # 2
print(c)   # X
print(d)   # Y
```

The third element of the outer list is a list, so `(c, d)` unpacks it into two names.

### Strings work as inner iterables too

```python
a, b, (c, d) = [1, 2, 'XY']

print(a)   # 1
print(b)   # 2
print(c)   # X
print(d)   # Y
```

The string `'XY'` is iterable, so its characters are unpacked into `c` and `d`. Nothing special is required — any iterable can be the target of any unpacking pattern.

### Nesting with `*`

Combine nesting with starred patterns:

```python
a, b, (c, d, *e) = [1, 2, 'python']

print(a)   # 1
print(b)   # 2
print(c)   # p
print(d)   # y
print(e)   # ['t', 'h', 'o', 'n']
```

The outer pattern is `a, b, <nested>`; the inner pattern is `c, d, *e`, applied to `'python'`.

### Multiple stars — apparently?

Earlier we said `*` can appear only once per pattern. But this works:

```python
a, *b, (c, d, *e) = [1, 2, 3, 'python']

print(a)   # 1
print(b)   # [2, 3]
print(c)   # p
print(d)   # y
print(e)   # ['t', 'h', 'o', 'n']
```

Two stars, one pattern — legal. Why?

Because they are at **different nesting levels**:

- `*b` is at the **outer** level (matching against `[1, 2, 3, 'python']`).
- `*e` is at the **inner** level (matching against `'python'`).

The rule is **one `*` per level**, not one `*` per expression.

### Step-by-step decomposition

The nested form can be decomposed into two unpackings:

```python
a, *b, tmp = [1, 2, 3, 'python']
# a = 1, b = [2, 3], tmp = 'python'

c, d, *e = tmp
# c = 'p', d = 'y', e = ['t', 'h', 'o', 'n']
```

The nested form is exactly shorthand for these two operations — nothing more mysterious is happening.

### Compare with the slicing equivalent

Without unpacking:

```python
l = [1, 2, 3, 'python']

a, b, c, d, e = l[0], l[1:-1], l[-1][0], l[-1][1], list(l[-1][2:])
```

Five index expressions, each depending on the specific structure. With unpacking:

```python
a, *b, (c, d, *e) = l
```

Same result, expressed as a pattern that mirrors the data's shape.

The pattern generalizes to arbitrary depth and structure:

```python
l = [1, 2, 3, 4, 'unladen swallow']
# a=1, b=[2,3,4], c='u', d='n', e=['l','a','d','e','n',' ','s','w','a','l','l','o','w']

l = [1, 2, 3, 4, ['a', 'b', 'c', 'd']]
# a=1, b=[2,3,4], c='a', d='b', e=['c','d']
```

Each level of nesting mirrors one level of structure. What the slicing form expresses as a sequence of index expressions, the unpacking form expresses as a single structural pattern.

---

## Rules and Reference

### Where `*` can appear

| Context | `*x` means |
|---|---|
| Assignment (left side) | collect remaining elements into a **list** |
| Function definition | collect extra positionals into a **tuple** |
| Function call | unpack an iterable into separate arguments |
| List literal `[*a, *b]` | merge iterables into a list |
| Set literal `{*a, *b}` | merge iterables into a set |
| Dict literal (`**a, **b`) | merge key-value pairs into a dict |

### Constraints

- **One `*` per pattern level** — not per expression. Nested patterns can each have their own `*`.
- **Starred name produces a list** on the left side of an assignment, regardless of the source type.
- **Order sensitivity with `**`** — later occurrences overwrite earlier ones.
- **Set order is not guaranteed** — `a, *b = some_set` works, but which elements end up in `a` and `b` is arbitrary.

### Quick examples

```python
# Basic "head and tail"
a, *b = iterable                     # a: first, b: rest (list)

# Head, second, middle, last
a, b, *c, d = iterable               # a: first, b: second, c: middle, d: last

# Nested
a, *b, (c, *d) = iterable            # one * per level — legal

# Unpack into collections
[*iter1, *iter2]                     # list
{*set1, *set2}                       # set
{**d1, **d2}                         # dict (later keys win)

# Set → list (idiomatic)
*a, = some_set                       # trailing comma makes it a pattern
```

---

## Summary

| Feature | Syntax | Result | Notes |
|---|---|---|---|
| Capture the rest | `a, *b = it` | `b` is a `list` | Works on any iterable |
| Trailing names | `a, *b, c = it` | `c` from the end | `*` can sit anywhere in the pattern |
| One star per level | `a, *b, *c = it` | `SyntaxError` | Two stars on one level ambiguous |
| Merge into a list | `[*a, *b]` | `list` | Splice iterables |
| Merge into a set | `{*a, *b}` | `set` | Order not preserved |
| Merge into a dict | `{**a, **b}` | `dict` | Later keys overwrite earlier |
| Mix with literals | `{'k': 0, **d}` | `dict` | Literals can appear anywhere |
| Set → list | `*a, = s` | `list` | Trailing comma essential |
| Nested pattern | `a, (b, *c) = it` | — | One `*` per level |

Extended unpacking generalizes basic unpacking from "exactly N names for N elements" to "N names plus a catch-all remainder." This single relaxation unlocks a wide range of patterns — head/tail splitting, first/second/last extraction, list merging, dict merging, and structurally recursive unpacking of nested containers — all expressed declaratively, without the manual indexing and slicing that would otherwise be required.

The unifying idea: **the left-hand pattern mirrors the shape of the right-hand value**. A nested pattern matches nested structure. A starred name captures the flexible part. The one-per-level rule keeps the binding unambiguous, even when the pattern is arbitrarily deep.