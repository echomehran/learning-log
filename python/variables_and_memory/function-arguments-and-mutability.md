# Function Arguments and Mutability

## The Core Question

When we pass an argument to a function, does the function get *its own copy* of the value, or does it get a *reference* to the caller's object? And if it modifies that argument, will the caller see the change?

The answer depends on **two distinct things**:

1. **Rebinding vs. mutating** — does the function reassign the name, or mutate the object?
2. **Mutability of the object** — can the object's internal state change at all?

These are often conflated, so let's separate them carefully using `id()` to track object identity.

---

## Case 1: Immutable Object + Rebinding

```python
def process(s):
    print('initial s  # = {0}'.format(hex(id(s))))
    s = s + ' world'
    print('s after change # = {0}'.format(hex(id(s))))


my_var = 'hello'
print('my_var # = {0}'.format(hex(id(my_var))))
# my_var # = 0x1e7e96fc420
```

When `s` is received, it **references the same object** as `my_var` — same memory address.

Now call the function:

```python
process(my_var)
# initial s  # = 0x1e7e96fc420
# s after change # = 0x1e7e97153b0
```

The address changed! Why? Because `s = s + ' world'` does **not** modify the string in place (strings are immutable). It creates a **new string object** and **rebinds the name `s`** to it.

Meanwhile, `my_var` still points to the original object:

```python
print('my_var # = {0}'.format(hex(id(my_var))))
# my_var # = 0x1e7e96fc420   ← unchanged
```

### What happened

```text
Caller:  my_var ────► "hello"  (addr A)

Inside:  s ─────────► "hello"  (addr A)     ← same object
         
         s = s + ' world'
         
         s ─────────► "hello world"  (addr B)   ← NEW object
         my_var ────► "hello"        (addr A)   ← still unchanged
```

**Key insight:** the function never had access to `my_var`. It only had `s`, which is a *local name* bound to the same object. Rebinding `s` has zero effect on `my_var`.

---

## Case 2: Mutable Object + Mutation

```python
def modify_list(items):
    print('initial items # = {0}'.format(hex(id(items))))
    if len(items) > 0:
        items[0] = items[0] ** 2
    items.pop()
    items.append(5)
    print('final items   # = {0}'.format(hex(id(items))))


my_list = [2, 3, 4]
print('my_list # = {0}'.format(hex(id(my_list))))
# my_list # = 0x1e7e972d308
```

Call it:

```python
modify_list(my_list)
# initial items # = 0x1e7e972d308
# final items   # = 0x1e7e972d308   ← SAME address
```

The address never changes — because `items[0] = ...`, `items.pop()`, and `items.append(5)` all **mutate the existing list in place**. No new object is created; the internal state changes.

Now look at `my_list`:

```python
print(my_list)                              # [4, 3, 5]
print('my_list # = {0}'.format(hex(id(my_list))))
# my_list # = 0x1e7e972d308                ← still the same object
```

The caller *does* see the change — because `my_list` and `items` are **two names for the same object**, and that object's contents were modified.

### What happened

```text
Caller:  my_list ────► [2, 3, 4]  (addr A)
                    
Inside:  items ──────► [2, 3, 4]  (addr A)     ← same object

         items[0] = items[0] ** 2   → mutate in place
         items.pop()                → mutate in place
         items.append(5)            → mutate in place

         items ──────► [4, 3, 5]  (addr A)     ← same object, new contents
         my_list ────► [4, 3, 5]  (addr A)     ← caller sees the change
```

**Key insight:** mutation changes the object that *both* names reference. No rebinding occurs, so no new object is created.

---

## Case 3: Immutable Container Holding a Mutable Element

This is where things get subtle. Consider a tuple containing a list:

```python
def modify_tuple(t):
    print('initial t # = {0}'.format(hex(id(t))))
    t[0].append(100)
    print('final t   # = {0}'.format(hex(id(t))))


my_tuple = ([1, 2], 'a')
hex(id(my_tuple))
# '0x1e7e9614288'

modify_tuple(my_tuple)
# initial t # = 0x1e7e9614288
# final t   # = 0x1e7e9614288
```

The tuple's address never changes — as expected, since tuples are immutable. But look at the result:

```python
my_tuple
# ([1, 2, 100], 'a')
```

The first element — the **list** — was modified!

### What happened

```text
my_tuple ────► ( [1, 2], 'a' )   addr A  (tuple is immutable)
                    │
                    └──► [1, 2]  addr B  (list is mutable)

Inside function:
    t ──────────► ( [1, 2], 'a' )   addr A   ← same tuple
                        │
                        └──► [1, 2]  addr B   ← same list

    t[0].append(100)
        ↓
    mutates the list at addr B in place
    (the tuple at addr A is unchanged — its slot still points to addr B)
```

The tuple itself is immutable — you cannot reassign `t[0]`. But **`t[0]` is a reference to a mutable list**, and mutating that list through the reference is perfectly legal. The outer tuple never changes; the object it *points to* does.

**Key insight:** "Immutable" refers to the *container's* slots, not to the objects those slots reference. Immutability is not deep by default.

---

## The Underlying Model

Everything here follows from one rule:

> **Assignment and argument passing always bind *names* to *objects*. They never copy objects.**

So the real distinction isn't "pass by value" vs. "pass by reference" (Python is neither, strictly). It's:

| What the function does | Effect on caller |
|---|---|
| **Rebinds** the parameter name (`s = s + ...`) | None — caller's name still points to the original object |
| **Mutates** the object (`items[0] = ...`) | Caller sees it — same object, changed contents |
| Mutates through a reference inside an immutable container | Caller sees it — the inner object is still mutable |

### Mental model

```text
    Function arguments are bindings, not copies.

    f(x) does:
        local_name ──► (same object as x)

    Rebinding inside f:
        local_name = <new object>       → no visible effect outside
        (because x still points to the old object)

    Mutating inside f:
        local_name[i] = ...             → visible outside
        local_name.append(...)          → visible outside
        (because both names point to the same object)

    Immutable container with mutable contents:
        t[0].append(...)                → visible outside
        (the outer container is immutable, but its element isn't)
```

---

## Practical Implications

### 1. Don't assume a parameter is "safe" to modify

If you write:

```python
def normalize(items):
    items.sort()          # mutates caller's list!
    return items
```

…the caller's list is now sorted, whether they wanted that or not. To avoid surprises:

- **Mutate on purpose** (documented, expected), *or*
- **Copy first**: `items = list(items)` or `items = items.copy()`

### 2. Beware of mutable default arguments

A related classic pitfall (worth knowing now):

```python
def append_to(item, target=[]):    # ← the list is created ONCE
    target.append(item)
    return target

append_to(1)   # [1]
append_to(2)   # [1, 2]  ← not [2]!
```

The default `[]` is evaluated **once**, at function definition time. Subsequent calls reuse the *same* list. The fix: use `None` as the sentinel.

```python
def append_to(item, target=None):
    if target is None:
        target = []
    target.append(item)
    return target
```

### 3. "Immutable" doesn't mean "safe"

A tuple can hold a mutable list. A `frozenset` can hold only hashable — but hashable doesn't guarantee immutable contents either (custom objects can be hashable and mutable, though that's usually a bug). Treat immutability as *shallow* unless you've verified otherwise.

### 4. Identity is the real question

When you want to know "did the function change my object?", the answer is:

- **`id()` before and after** → same address means the same object; mutations are visible.
- **Different address means the name was rebound** — your original is untouched.

---

## Summary

```text
Function arguments are BINDINGS, not copies.

    Passing x to f(x):
        local_name ──► same object as x

    Rebinding inside f:
        local_name = <new object>
        → NO effect on caller

    Mutating inside f:
        local_name[i] = ...
        local_name.append(...)
        → visible to caller (same object)

    Immutable container with mutable element:
        t[0].append(...)
        → visible to caller (inner object is mutable)

Rules of thumb:
    - To avoid touching the caller's object: copy it.
    - To intentionally modify it: mutate in place (document it).
    - Never use mutable defaults like [] or {} in function signatures.
    - "Immutable" is shallow — always check what's inside.
```

Once you internalize that **names are references, and passing is binding**, questions about "pass by value or reference" dissolve into a much simpler, more accurate question: *did the function rebind a name, or mutate an object?* Everything else follows from that.