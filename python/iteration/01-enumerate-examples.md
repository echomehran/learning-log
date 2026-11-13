# Python `enumerate()`

`enumerate()` is a built-in Python function that allows us to iterate over an iterable while keeping track of the index of each item.

## Basic example

Suppose we have a list of names:

    names = ["Alice", "Bob", "Charlie"]

    for index, name in enumerate(names):
        print(index, name)

Output:

    0 Alice
    1 Bob
    2 Charlie

Each iteration gives us two values:

    index
    value

Conceptually, `enumerate()` produces pairs like:

    (0, "Alice")
    (1, "Bob")
    (2, "Charlie")

The loop then unpacks each pair into `index` and `name`.

---

## Starting the index from a different number

By default, `enumerate()` starts counting from `0`.

We can change this using the `start` argument:

    names = ["Alice", "Bob", "Charlie"]

    for index, name in enumerate(names, start=1):
        print(f"{index}. {name}")

Output:

    1. Alice
    2. Bob
    3. Charlie

This is useful when displaying human-readable numbering.

---

## Why use `enumerate()`?

Without `enumerate()`, we might manually maintain a counter:

    names = ["Alice", "Bob", "Charlie"]

    index = 0

    for name in names:
        print(index, name)
        index += 1

This works, but it introduces unnecessary state that we have to maintain ourselves.

With `enumerate()`:

    names = ["Alice", "Bob", "Charlie"]

    for index, name in enumerate(names):
        print(index, name)

The code is shorter and directly expresses what we want:

> Iterate over the values while also giving me their indexes.

---

## What does `enumerate()` return?

A useful mental model is:

    enumerate(iterable, start=0)

It returns an `enumerate` object.

For example:

    names = ["Alice", "Bob", "Charlie"]

    result = enumerate(names)

    print(result)

The result is not a list. It is an `enumerate` object that produces `(index, value)` pairs as we iterate over it.

We can convert it to a list if we want to see all of the pairs:

    names = ["Alice", "Bob", "Charlie"]

    result = list(enumerate(names))

    print(result)

Output:

    [(0, 'Alice'), (1, 'Bob'), (2, 'Charlie')]

With `start=1`:

    result = list(enumerate(names, start=1))

    print(result)

Output:

    [(1, 'Alice'), (2, 'Bob'), (3, 'Charlie')]

---

## `enumerate()` works with more than lists

`enumerate()` works with any iterable.

For example, with a string:

    word = "Python"

    for index, character in enumerate(word):
        print(index, character)

Output:

    0 P
    1 y
    2 t
    3 h
    4 o
    5 n

It can also be used with tuples, sets, generators, and other iterable objects.

---

## A common pattern

A very common Python pattern is:

    for index, value in enumerate(items):
        ...

Use this when you need both:

- the current item's position
- the current item's value

If you only need the values, there is no reason to use `enumerate()`:

    for name in names:
        print(name)

If you need both the position and the value:

    for index, name in enumerate(names):
        print(index, name)

---

## Important idea

`enumerate()` does not modify the original iterable.

For example:

    names = ["Alice", "Bob", "Charlie"]

    for index, name in enumerate(names):
        print(index, name)

    print(names)

The original list is unchanged:

    ['Alice', 'Bob', 'Charlie']

`enumerate()` simply provides another way to iterate over the values while keeping track of their positions.

---

## Key takeaway

The basic form is:

    enumerate(iterable, start=0)

It allows us to write:

    for index, value in enumerate(iterable):
        ...

Instead of manually maintaining a counter.

The main idea to remember is:

    enumerate() → (index, value)

Use it when you need both the position and the item during iteration.