# Everything Is an Object

## Variables Are Objects Too

```python
a = 10
```

**Note:** `a` is an object of type `int` — more precisely, `a` is an **instance** of the `int` class.

```python
print(type(a))
# <class 'int'>
```

If `int` is a class, we should be able to create instances of it using the standard class instantiation syntax:

```python
b = int(10)
print(b)         # 10
print(type(b))   # <class 'int'>
```

So `10` and `int(10)` produce the **same kind of object**. The literal `10` is really just sugar for constructing an `int` instance.

---

## Even Built-in Types Have Documentation

We can ask Python for the documentation of any class:

```python
help(int)
```

Among the docs, several things stand out:

```text
class int(object)
 |  int(x=0) -> integer
 |  int(x, base=10) -> integer
```

- `int` inherits from `object` — the root of all classes.
- It has an **overloaded constructor**: it can be called with zero, one, or two arguments.

The docs also reveal methods like `__add__`, `__bool__`, `__eq__`, and so on — every one of which is an **attribute on the object**, waiting to be called.

### Using the overloaded constructor

Because `int` accepts a `base`, we can parse numbers in arbitrary bases:

```python
b = int('10', base=2)
print(b)         # 2
print(type(b))   # <class 'int'>
```

The string `'10'` is interpreted as a **binary** literal, producing the integer `2`.

---

## Functions Are Objects Too

A function defined with `def` is not merely a syntactic construct — it's a **first-class object**.

```python
def square(a):
    return a ** 2

type(square)
# → function
```

The function has a type (`function`), and — like any object — it can be:

- **Bound to another name** (assigned to a variable)
- **Passed as an argument**
- **Returned from another function**
- **Stored in containers**
- **Inspected** (`id`, `type`, `__name__`, `__doc__`, …)

### Rebinding

```python
f = square
type(f)      # → function
f is square  # → True   (the same object, just two names)
f(2)         # → 4
type(f(2))   # → int    (calling a function returns an object)
```

`f` and `square` are **two names for the same object** — not a copy. `f is square` returning `True` confirms this: identity, not equality.

---

## A Function Can Return a Function

```python
def cube(a):
    return a ** 3

def select_function(fn_id):
    if fn_id == 1:
        return square
    else:
        return cube
```

`select_function` **returns a function object** — not the result of calling one, but the function itself.

```python
f = select_function(1)

print(hex(id(f)))        # 0x21257457b70
print(hex(id(square)))   # 0x21257457b70   ← same object
print(hex(id(cube)))     # 0x21255fab8c8

print(type(f))                   # <class 'function'>
print('f is square: ', f is square)   # True
print('f is cube: ',   f is cube)     # False
print(f)                         # <function square at 0x...>
print(f(2))                      # 4
```

The `id()` values are the smoking gun: `f` and `square` share an address — they are **the very same object**.

Selecting the other branch:

```python
f = select_function(2)

print(hex(id(f)))        # 0x21255fab8c8
print(hex(id(square)))   # 0x21257457b70
print(hex(id(cube)))     # 0x21255fab8c8   ← same object as f

print(type(f))                   # <class 'function'>
print('f is square: ', f is square)   # False
print('f is cube: ',   f is cube)     # True
print(f)                         # <function cube at 0x...>
print(f(2))                      # 8
```

### Chained calls

Because the return value is itself a function, you can immediately call it:

```python
select_function(1)(5)
# → 25
```

`select_function(1)` returns `square`, and `(5)` calls it. Two operations, one expression — a direct consequence of functions being ordinary values.

---

## A Function Can Take a Function as an Argument

```python
def exec_function(fn, n):
    return fn(n)
```

The parameter `fn` is just a name — and names can point to *any* object, including a function.

```python
result = exec_function(cube, 2)
print(result)   # 8
```

This is *the* mechanism that enables:

- **Higher-order functions** — functions that accept or return other functions
- **Callbacks** — passing a function to be invoked later
- **Map/filter/reduce**-style patterns
- **Decorators** — functions that wrap other functions
- **Dependency injection** — parameterizing behavior instead of hard-coding it

We'll return to functions-as-arguments **extensively** throughout this course.

---

## Why "Everything Is an Object" Matters

This isn't just a slogan — it's the reason a huge range of Python idioms work:

| Concept | Enabled by |
|---|---|
| `type(x)` works on anything | Every value is an instance of some class |
| `dir(x)`, `help(x)` work uniformly | Every object exposes attributes/methods |
| Functions can be passed/returned | Functions are first-class objects |
| Classes can be passed/returned | Classes are objects too (`type` is a class) |
| Everything can be stored in a list/dict | Containers hold references to arbitrary objects |
| `f is g` distinguishes same vs. equal | Objects have identity |
| Monkey-patching, decorators, metaprogramming | Classes and functions are mutable objects |

Not everything in Python is an *instance* of a *user-defined* class — but **every value has a type, and every type is itself an object**. The uniformity is what makes Python's object model both simple and powerful.

---

## Summary

```text
Everything is an object.

    - 10              is an int object  (instance of class int)
    - "hello"         is a str object
    - square          is a function object
    - int             is a type object  (an instance of type)
    - None            is an object      (the NoneType singleton)

Every object has:
    - a type            (type(obj))
    - an identity       (id(obj))
    - attributes        (obj.attr, dir(obj))

Because functions are objects:
    - They can be assigned to names      f = square
    - They can be passed as arguments    exec_function(cube, 2)
    - They can be returned                select_function(1)
    - They can be called immediately     select_function(1)(5)

Implications:
    f is square  → True   (same object, two names)
    int('10', 2) → 2      (constructors are just class calls)
```

Once you internalize that **a name is a reference to an object**, and **every object carries a type and an identity**, statements like `f = square`, `f is square`, and `select_function(1)(5)` stop looking like special tricks. They're just the ordinary behavior of objects in a uniformly object-oriented language.