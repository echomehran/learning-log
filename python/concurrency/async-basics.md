# Python Async Basics

## Why This Topic Matters in the Media Bot Project

The `media-bot` project uses `aiogram` and SQLAlchemy asynchronously.

This means the program is not supposed to freeze the entire application while waiting for tasks such as:

- Receiving Telegram messages
- Running queries on PostgreSQL
- Fetching data from an external service
- Downloading a file
- Uploading a file to Telegram

That's why we encounter things like this in the project code:

```python
async def main(): ...
```

And:

```python
await dispatcher.start_polling(bot)
```

To understand these, we need to separate a few concepts.

---

## Clarification: What Is `async def`?

In Python, to define a regular function, we write:

```python
def get_user(): ...
```

But to define an asynchronous function, we write:

```python
async def get_user(): ...
```

So:

```text
def
    regular function

async def
    asynchronous function
```

Note: `async` tells Python that this function is a coroutine function and can run in an asynchronous environment.

For example, in the project:

```python
async def main() -> None: ...
```

The `main` function is an asynchronous function.

---

## Clarification: What Is a Coroutine?

When we call an `async def` function, it does not execute immediately like a regular function.

For example:

```python
async def get_user():
    return "Mehran"


result = get_user()
```

Here, `result` is not the value `"Mehran"`.

Instead, it is a coroutine object that must be executed by the event loop.

In simple terms:

```text
async def
    ↓
coroutine function

calling the function
    ↓
coroutine object

event loop
    ↓
executes the coroutine
```

That's why we encounter `await` in the project code.

---

## Clarification: What Is `await`?

Note: `await` means:

> Continue executing this coroutine, but allow the event loop to perform other asynchronous work while waiting.

For example:

```python
async def main():
    await dispatcher.start_polling(bot)
```

Here, `start_polling()` is an asynchronous operation.

We don't want to call it like a regular function:

```python
dispatcher.start_polling(bot)
```

Instead, we write:

```python
await dispatcher.start_polling(bot)
```

Because the coroutine must be executed and managed.

---

## Why Doesn't `await` Fully Stop the Program?

Suppose the bot needs to fetch data from PostgreSQL.

If the program runs fully synchronously:

```text
Python
  ↓
Database query
  ↓
Waiting for PostgreSQL
  ↓
Receiving the result
  ↓
Continuing the program
```

During the wait, the thread may effectively be idle.

In the asynchronous model:

```text
Python
  ↓
Database query
  ↓
Waiting for PostgreSQL
  │
  ├── Another task is ready to run
  │
  ├── Another Telegram event has arrived
  │
  └── Another operation is ready to run
  ↓
database response
  ↓
continuing the coroutine
```

Therefore, asynchronous programming is especially useful for programs that involve a lot of I/O.

---

## Clarification: What Is I/O?

Note: I/O stands for:

```text
Input / Output
```

Any task where the program needs to communicate with an external resource is generally considered I/O.

For example:

```text
PostgreSQL
Telegram API
HTTP API
File system
Network
```

In `media-bot`, this is very important because the program will likely involve a lot of I/O.

For example:

```text
Telegram message
      ↓
API request
      ↓
database query
      ↓
external provider
      ↓
download
      ↓
file storage
      ↓
Telegram upload
```

A large part of this workflow involves waiting for external resources.

---

## Clarification: What Is an Event Loop?

Note: The event loop can be thought of as the engine that runs asynchronous code.

Simply put:

```text
Event Loop
    │
    ├── coroutine A → waiting for database
    │
    ├── coroutine B → ready to run
    │
    ├── coroutine C → waiting for network
    │
    └── coroutine D → ready to run
```

When a coroutine reaches an I/O operation and must wait, the event loop can run another coroutine.

This is one of the foundations of asynchronous programming in Python.

---

## Why Do We Have `asyncio.run()`?

In the project's `main.py` file:

```python
if __name__ == "__main__":
    asyncio.run(main())
```

The `main()` function is a coroutine function.

To execute it, we need an event loop.

`asyncio.run()` does this for us.

Conceptually:

```text
asyncio.run(main())
        ↓
create event loop
        ↓
run main()
        ↓
manage coroutines
        ↓
end event loop
```

So we should not confuse these two concepts:

```python
async def main():
```

This defines an asynchronous function.

Whereas:

```python
asyncio.run(main())
```

This runs that coroutine from the program's entry point.

---

## Why Is the Database Also Async?

In the project, we use SQLAlchemy with async support:

```python
from sqlalchemy.ext.asyncio import AsyncSession
```

And our database driver is:

```text
asyncpg
```

Therefore, database queries can also be executed asynchronously.

For example:

```python
result = await session.execute(statement)
```

Here:

```text
session.execute(...)
```

is an asynchronous operation.

And:

```python
await
```

allows the event loop to manage other tasks while waiting for the database.

---

## A Common Mistake

Let's not confuse these two:

```python
async def get_user(): ...
```

And:

```python
await get_user()
```

The first defines a function.

The second executes/awaits the coroutine produced by that function.

---

## How This Relates to Media Bot

In the project's current architecture:

```text
Telegram
    ↓
aiogram
    ↓
Application
    ↓
Repository
    ↓
SQLAlchemy
    ↓
asyncpg
    ↓
PostgreSQL
```

We have asynchronous programming at several points.

For example:

```python
async def main():
```

```python
async def get_by_telegram_id(...):
```

```python
result = await session.execute(statement)
```

These are not accidental.

The goal is for the project's I/O-heavy workflows to run without blocking the entire application.

---

## What I Don't Need to Know Yet

At this stage, I don't need to learn all the details of `asyncio`.

For now, this mental model is sufficient:

```text
async def
    ↓
defines an asynchronous function

calling async function
    ↓
creates a coroutine

await
    ↓
runs the coroutine and waits asynchronously

event loop
    ↓
manages the execution of coroutines

I/O
    ↓
where asynchronous programming is usually most beneficial
```

Later, as the project progresses, we can learn the following concepts separately:

- coroutine
- task
- Future
- event loop
- concurrency
- `asyncio.create_task`
- cancellation
- timeout
- async context manager
- async iterator
- async generator

---

## Relation to Project Files

To see these concepts in the current project:

```text
src/music_bot/main.py
```

shows the start of the asynchronous application.

```text
src/music_bot/repositories/user.py
```

shows how a database operation has been made asynchronous.

```text
tests/conftest.py
```

shows how we've used an async database engine and `AsyncSession` in tests.

---

## Summary

The most important thing to remember for now:

```text
async def
    ↓
defines an asynchronous function

await
    ↓
waits for an asynchronous operation

event loop
    ↓
manages the execution of asynchronous tasks

I/O
    ↓
tasks like database and network that involve waiting
```

In `media-bot`, we use async because a large part of the application deals with Telegram, PostgreSQL, the network, and—in the future—media downloads.