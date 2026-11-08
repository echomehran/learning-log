# Python Peephole Optimizations

Peephole optimizations are a specific category of optimization strategies that Python applies **during compilation**. Unlike runtime optimizations, these are baked into the compiled bytecode before a single line executes — the resulting `code object` already reflects the improved form.

You can inspect them directly through `func.__code__.co_consts`, which exposes the constant table the compiler produced.

---

## Constant Folding

### What it does

The compiler evaluates **constant expressions at compile time** and replaces them with their computed result. Let's see what happens:

```python
def my_func():
    a = 24 * 60
    b = (1, 2) * 5
    c = 'abc' * 3
    d = 'ab' * 11
    e = 'the quick brown fox' * 10
    f = [1, 2] * 5
```

Now inspect the constant pool:

```python
my_func.__code__.co_consts
```

```text
(None,
 24, 60,                           ← operands preserved
 1, 2, 5,                          ← operands preserved
 'abc', 3,                         ← operands preserved
 'ab', 11,                         ← operands preserved
 'the quick brown fox', 10,        ← NOT folded (too long)
 1440,                             ← 24 * 60 was folded
 (1, 2),                           ← (1, 2) preserved
 (1, 2, 1, 2, 1, 2, 1, 2, 1, 2),   ← (1, 2) * 5 was folded
 'abcabcabc')                      ← 'abc' * 3 was folded
```

### What got folded — and what didn't

| Expression | Folded? | Why |
|---|---|---|
| `24 * 60` | ✅ → `1440` | Constant integers, cheap arithmetic |
| `(1, 2) * 5` | ✅ → `(1,2,1,2,1,2,1,2,1,2)` | Tuple is immutable, small result |
| `'abc' * 3` | ✅ → `'abcabcabc'` | String is immutable, short result |
| `'ab' * 11` | ⚠️ Partial (operands kept, not obviously folded here) | Result size near the fold threshold |
| `'the quick brown fox' * 10` | ❌ | Result would be **170 chars** — too large to fold |
| `[1, 2] * 5` | ❌ | **List is mutable** — cannot be a compile-time constant |

### The rules behind constant folding

1. **Operands must be immutable.** `int`, `str`, `tuple`, `bytes`, `frozenset` qualify. `list`, `dict`, `set` do not — their contents could be mutated, so a single cached constant would be wrong (all calls would share the same mutable object).
2. **The result must be small.** CPython enforces limits (e.g., string length) so that folding doesn't silently produce huge constants or consume compile-time memory. Folding a 170-character string is not worth the bytecode-table cost; it's cheaper to multiply at runtime.
3. **The operands are still kept** in `co_consts` when the folded result depends on them in other contexts, or because the compiler doesn't do a second pass to remove them.

### Why this matters

- **Bytecode is smaller.** `1440` is one load; `24 * 60` would be three ops.
- **Runtime is faster.** The multiplication cost is paid once, at compile time, not on every call.
- **The optimization is free.** No programmer effort required — it just happens.

### When it can't apply

Mutable containers are the big exception. Consider:

```python
def f():
    x = [1, 2] * 5    # NOT folded
```

If the compiler *did* fold this, every call to `f()` would get the **same list object** — and mutating it in one call would leak into the next. So the compiler refuses. The multiplication runs at runtime, producing a fresh list each call.

This is a good example of why **mutability limits what the compiler is allowed to do**.

---

## Membership Test Rewriting

### Lists → tuples

Consider a membership test using a literal list:

```python
def my_func():
    if e in [1, 2, 3]:
        pass
```

```python
my_func.__code__.co_consts
# (None, 1, 2, 3, (1, 2, 3))
```

The mutable list `[1, 2, 3]` has been **converted into an immutable tuple** `(1, 2, 3)` at compile time.

### Why is that safe?

The list is written as a **literal** inside an expression — it's never bound to a name, never passed around, never mutated. The only operation performed on it is the **membership test**, which only reads. So the compiler can substitute an equivalent immutable object without changing observable behavior.

The tuple is:
- **Smaller** in memory (no over-allocation like lists have).
- **Faster to iterate**, because it has a fixed length and predictable layout.

### Sets → frozensets

The same trick applies to set literals:

```python
def my_func():
    if e in {1, 2, 3}:
        pass
```

```python
my_func.__code__.co_consts
# (None, 1, 2, 3, frozenset({1, 2, 3}))
```

The mutable `set` literal is rewritten as an immutable `frozenset`.

### Why this matters — the big one

Compile-time rewriting makes set-membership tests **dramatically faster** than list or tuple tests, because sets (and frozensets) are backed by **hash tables** — the same mechanism as `dict` lookups.

`x in container`:

| Container | Complexity | Mechanism |
|---|---|---|
| `list` | **O(n)** | Linear scan |
| `tuple` | **O(n)** | Linear scan |
| `set` / `frozenset` | **O(1)** amortized | Hash lookup |

For small containers the difference is invisible; for large containers it's enormous.

### Benchmark

A quick, crude measurement:

```python
import string
import time

char_list  = list(string.ascii_letters)
char_tuple = tuple(string.ascii_letters)
char_set   = set(string.ascii_letters)


def membership_test(n, container):
    for i in range(n):
        if 'p' in container:
            pass
```

List membership:

```python
start = time.perf_counter()
membership_test(10_000_000, char_list)
end = time.perf_counter()
print('list membership: ', end - start)
# list membership: 2.6035...
```

Tuple membership:

```python
start = time.perf_counter()
membership_test(10_000_000, char_tuple)
end = time.perf_counter()
print('tuple membership: ', end - start)
# tuple membership: 2.6024...
```

Set membership:

```python
start = time.perf_counter()
membership_test(10_000_000, char_set)
end = time.perf_counter()
print('set membership: ', end - start)
# set membership: 0.3743...
```

### Interpreting the results

- **List ≈ Tuple.** Both are linear scans — no meaningful difference. Their 2.6 s vs 2.6 s is essentially noise.
- **Set is ~7× faster here**, and the gap **grows with container size** because set lookup is O(1) while list/tuple is O(n). With only 52 elements, set lookup is already ~7× faster; with thousands of elements, the gap becomes orders of magnitude.

> The benchmark is intentionally crude — `time.perf_counter` on a loop includes interpreter overhead — but the asymptotics are the point. The 7× here is a *floor*, not a ceiling.

### Practical takeaway

When writing membership tests **by hand**, prefer:

```python
x in {1, 2, 3, 4, 5}       # ✅ set literal → O(1) lookup
```

over:

```python
x in [1, 2, 3, 4, 5]       # ❌ list literal → O(n) lookup
x in (1, 2, 3, 4, 5)       # ❌ tuple literal → O(n) lookup
```

…whenever the collection is a compile-time literal and order/duplicates don't matter.

The compiler will help you in the **literal** case (by freezing it), but you should still write the set literal *yourself* — it makes intent clear and avoids relying on an optimization for correctness of *performance expectations*.

---

## The Bigger Picture

Peephole optimizations are one layer in Python's optimization stack:

```text
Source code
    ↓
Compilation (CPython's compiler, bytecode emitter)
    ├── Constant folding          ← this lesson
    ├── Membership rewriting      ← this lesson
    ├── Dead code elimination
    ├── Constant propagation
    └── …
    ↓
Bytecode (already partially optimized)
    ↓
Runtime (interpreter loop)
    ├── Caching (small-int interning, etc.)
    ├── Adaptive specialization (Python 3.11+)
    └── …
```

### What the lessons reinforce

| Concept | Where it appeared before | What it means here |
|---|---|---|
| **Immutability unlocks optimization** | `__bool__`, `__len__`; `is` vs `==` | Only immutable objects can be safely cached or folded |
| **`set` vs `list` membership** | Not directly | Sets use hashing → O(1) |
| **`is` vs `==`** | Interned integers | Folding relies on identity safety |
| **Compile-time vs runtime** | — | Folding and freezing happen **before** any code runs |

Every one of these connects back to the same underlying rules: immutability enables sharing and caching; hashing enables O(1) lookup; identity-vs-equality determines when sharing is observable.

---

## Summary

```text
Peephole optimizations — compile-time rewrites applied to constants
and simple expressions before bytecode is emitted.

Constant folding:
    24 * 60                     → 1440
    (1, 2) * 5                  → (1, 2, 1, 2, 1, 2, 1, 2, 1, 2)
    'abc' * 3                   → 'abcabcabc'
    
    NOT folded:
        'long string' * 10      → too large
        [1, 2] * 5              → mutable, cannot be constant

Membership rewriting:
    e in [1, 2, 3]              → e in (1, 2, 3)      (tuple)
    e in {1, 2, 3}              → e in frozenset(...) (frozenset)

Why safe:
    The literals are never bound or mutated — only read.
    Replacing a mutable literal with an immutable equivalent
    is observationally identical.

Why important:
    Sets/frozensets use hash tables → O(1) membership.
    Lists/tuples use linear scan    → O(n) membership.
    For large containers, the difference is enormous.

Write it yourself, don't rely on the optimizer:
    x in {1, 2, 3}              → write the set literal explicitly
    Avoid x in [1, 2, 3]        → unless you need list semantics
```

Once you understand these rewrites, `co_consts` stops looking like a curiosity and starts reading as a **window into the compiler's reasoning** — showing exactly which expressions it considered safe to compute early, and (just as importantly) which it refused to touch because mutability or size made folding incorrect or unprofitable.