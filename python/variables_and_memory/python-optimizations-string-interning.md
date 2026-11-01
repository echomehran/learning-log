# Python Optimizations: String Interning

Python automatically **interns** *some* strings — meaning it stores a single shared instance of certain string values so that duplicates reference the same object rather than separate copies.

The question is: **which strings?**

The rule is roughly: **identifiers and identifier-like constant strings get interned**.

---

## What Gets Interned

### All identifiers are interned

Every name the parser sees — variable names, function names, class names, attribute names, module names — is interned. This makes lookups in namespaces (`globals()`, `locals()`, attribute lookups) faster, because the dict keys are guaranteed to be the same objects.

### Identifier-like string literals

Constant strings that *look like* identifiers — letters, digits, underscores, not starting with a digit, no spaces — are also interned at compile time.

### Simple identifier-like strings

```python
a = 'hello'
b = 'hello'
print(id(a))   # 1342722069536
print(id(b))   # 1342722069536   ← same object
```

`hello` looks like an identifier → interned.

### Strings with spaces are NOT interned

```python
a = 'hello, world!'
b = 'hello, world!'
print(id(a))   # 1342722047024
print(id(b))   # 1342722170928   ← different objects
```

The comma and space disqualify it from the identifier-like set → not interned.

### Underscores and length don't matter (much)

```python
a = 'hello_world'
b = 'hello_world'
print(id(a), id(b))
# 1342722047856 1342722047856   ← interned (underscore is identifier-like)
```

Even very long identifier-like strings are interned:

```python
a = '_this_is_a_long_string_that_could_be_used_as_an_identifier'
b = '_this_is_a_long_string_that_could_be_used_as_an_identifier'
print(id(a), id(b))
# 1342721886784 1342721886784   ← still interned
```

### Leading digit: OK. Leading space: NOT OK.

```python
a = '1_hello_world'
b = '1_hello_world'
print(id(a), id(b))
# 1342722046256 1342722046256   ← interned (no spaces, identifier-like)
```

But add a space:

```python
a = '1 hello world'
b = '1 hello world'
print(id(a), id(b))
# 1342722046832 1342722172592   ← NOT interned
```

So the practical rule for *automatic* interning is: **looks like a Python identifier, even loosely** — no spaces, no punctuation beyond underscores. (The leading-digit case still gets interned, even though it's not a valid identifier — the trigger is empirical, not strictly "is a valid identifier".)

---

## Why Interning Exists

If two strings are the same object, equality comparison can be reduced to **identity comparison** — comparing a pointer instead of walking character-by-character.

```python
a = 'this_is_a_long_string'
b = 'this_is_a_long_string'
print('a==b:', a == b)   # True    (value comparison)
print('a is b:', a is b) # True    (identity comparison — same object)
```

That's the optimization: when interning guarantees uniqueness, `is` and `==` agree for **equal interned values**.

> ⚠️ **Crucial caveat:** `is` only works as an equality test **for interned strings**. For non-interned strings, `is` is still a valid identity test — it's just that two independently created strings usually **are** distinct objects.

---

## Where the `is` Trick Breaks

```python
a = 'hello world'
b = 'hello world'
print('a==b:', a == b)   # True
print('a is b:', a is b) # False   ← different objects!
```

Both strings contain the same value, but because `'hello world'` has a space, it isn't interned — so `a` and `b` are distinct objects. `is` returns `False` even though `==` returns `True`.

This is exactly why the canonical rule is:

> **Use `==` to compare strings. Use `is` only to test for `None` or other guaranteed singletons.**

The performance benefit of `is` does not justify relying on interning, because **interning is an implementation detail** — CPython interns identifier-like strings, but PyPy, Jython, and future CPython versions are under no obligation to do so.

---

## Forcing Interning with `sys.intern`

If you have a **legitimate** need to compare the same string millions of times, you can force interning:

```python
import sys

a = sys.intern('hello world')
b = sys.intern('hello world')
c = 'hello world'

print(id(a))   # 1342722172080
print(id(b))   # 1342722172080   ← same object
print(id(c))   # 1342722174896   ← different object
```

- `a` and `b` both point to the **same interned object** — even though `'hello world'` wouldn't be interned automatically.
- `c` is a plain (non-interned) string literal — a distinct object.

Therefore, for `a` and `b`, identity works as equality:

```python
print('a==b:', a == b)   # True
print('a is b:', a is b) # True    ← safe HERE, because both were interned
```

But note: `a is c` would be `False` — even though `a == c` is `True`. Mixing interned and non-interned strings breaks the `is` shortcut.

`sys.intern` is intended for **performance-sensitive** code (e.g., parsing where millions of identical short strings appear). For ordinary code, it's unnecessary and adds complexity. The docs are explicit: *"Interning strings is useful to gain a little performance on dictionary lookup... but only if the string is created many times."*

---

## When Does Interning Actually Pay Off?

Interning's benefit is real but **narrow**: it matters only when

1. You do a **very large number** of comparisons, **and**
2. The strings are **long** enough that character-by-character comparison is expensive, **or**
3. The strings are used as **dict keys** repeatedly.

Otherwise, the cost of interning (hashing the string, looking it up in the intern table, possibly creating the interned object) dominates any savings.

### A crude benchmark

```python
def compare_using_equals(n):
    a = 'a long string that is not interned' * 200
    b = 'a long string that is not interned' * 200
    for i in range(n):
        if a == b:
            pass


def compare_using_interning(n):
    a = sys.intern('a long string that is not interned' * 200)
    b = sys.intern('a long string that is not interned' * 200)
    for i in range(n):
        if a is b:
            pass
```

Run each 10,000,000 times:

```python
import time

start = time.perf_counter()
compare_using_equals(10_000_000)
end = time.perf_counter()
print('equality: ', end - start)
# equality:  2.965...

start = time.perf_counter()
compare_using_interning(10_000_000)
end = time.perf_counter()
print('identity: ', end - start)
# identity:  0.287...
```

### Interpreting the result

| Method | Time | Comparison cost |
|---|---|---|
| `==` | ~2.97 s | O(n) in string length |
| `is` | ~0.29 s | O(1) — pointer check |

**Roughly 10× faster** — for these specific conditions: long strings, 10 million comparisons.

But look at what the benchmark *requires* to show a win:
- Strings long enough (here, ~6,200 chars each) that equality is slow.
- Ten million iterations.
- Both sides explicitly interned.

For short strings, or few comparisons, the difference is negligible — and the overhead of `sys.intern` may even make it **slower**.

### Real-world guidance

**Do not** reach for `sys.intern` unless you have measured a hot path. Common legitimate cases:

- **Parsing** — a lexer producing millions of identifier tokens; interning makes subsequent equality checks and dict lookups trivial.
- **Serialization / schema mapping** — matching keys against a fixed set of field names.
- **Symbol tables** — compilers and interpreters.
- **Repeated comparisons** — where profiling shows string comparison dominating.

For everything else, **use `==`**. The 10× speedup on a pathological benchmark is not a reason to make your code depend on identity — it's a reason to understand *why* the standard library itself interns identifiers.

---

## What Actually Gets Interned (Practical Summary)

| String | Interned automatically? |
|---|---|
| `'hello'` | ✅ |
| `'hello_world'` | ✅ |
| `'_a_very_long_identifier_like_string'` | ✅ |
| `'1_hello'` | ✅ (leading digit still triggers interning) |
| `'hello world'` | ❌ (space) |
| `'hello!'` | ❌ (punctuation) |
| `'Hello, World!'` | ❌ |
| User-input strings | ❌ (created at runtime) |
| Strings from `sys.intern(...)` | ✅ (forced) |

The rule of thumb: **compile-time constant strings that look like identifiers**. Runtime-created strings, and constants with spaces/punctuation, are not interned by default.

---

## The Bigger Picture

String interning is one of several sharing optimizations in Python, alongside:

| Optimization | What's shared | Introduced in |
|---|---|---|
| **Small-int interning** | Integers -5 to 256 | Always |
| **Identifier interning** | Variable/function/class names | Always |
| **String literal interning** | Identifier-like constants | Always |
| **`sys.intern`** | Arbitrary strings (opt-in) | Always |
| **Empty tuple singleton** | `()` | Always |

All of these exist for the same reason: **when an object is immutable and common, sharing it is free speed and free memory**. The catch is that sharing changes the **`is`** relationship — which is why `is` is only safe for **guaranteed singletons**, never as a value-equality shortcut on arbitrary data.

---

## Summary

```text
String interning:
    Python caches certain strings as singletons.
    - All identifiers.
    - Constant strings that look like identifiers
      (letters, digits, underscores; no spaces/punctuation).

Consequences:
    a = 'hello'; b = 'hello'    → a is b   → True
    a = 'hello world'; b = 'hello world'
                                → a is b   → False

Comparing with is:
    Works ONLY for interned strings.
    'hello_world'  is 'hello_world'    → True
    'hello world'  is 'hello world'    → False

Forcing interning:
    a = sys.intern('hello world')      → a is sys.intern('hello world') → True

Performance:
    is   → O(1) pointer compare
    ==   → O(n) char-by-char compare
    ~10× speedup for long strings + many comparisons

Caveats:
    - Interning is a CPython implementation detail.
    - Use == for string comparison in general code.
    - sys.intern only when profiling justifies it.
    - Use is only for guaranteed singletons (None, True, False).

Mental model:
    Compile-time identifier-like constants are shared;
    everything else is a distinct object.
    Don't rely on this — but understand it when you see it.
```

String interning is the string analogue of small-integer interning: **a narrow, safe, automatic optimization** whose purpose is fast lookups in symbol tables and repeated comparisons. It's a perfect illustration of the recurring Python theme — **immutability enables sharing; sharing enables identity comparison; identity comparison is fast but only valid under specific conditions**. Knowing when those conditions hold is the difference between using `is` correctly and writing code that works on one interpreter version and breaks on the next.