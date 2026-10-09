# Python Async Basics

## چرا این موضوع در پروژه Media Bot مهم است؟

پروژه `media-bot` با `aiogram` و SQLAlchemy به صورت asynchronous کار می‌کند.

یعنی برنامه قرار نیست هنگام انجام کارهایی مثل:

- دریافت پیام Telegram
- اجرای query روی PostgreSQL
- دریافت اطلاعات از یک سرویس خارجی
- دانلود یک فایل
- ارسال فایل به Telegram

کل برنامه را متوقف کند و منتظر تمام شدن آن کار بماند.

برای همین در کد پروژه با چیزهایی مثل این مواجه می‌شویم:

```python
async def main(): ...
```

و:

```python
await dispatcher.start_polling(bot)
```

برای فهمیدن این کدها باید چند مفهوم را از هم جدا کنیم.

---

## اصلاحیه: `async def` چیست؟

در Python برای تعریف یک function معمولی می‌نویسیم:

```python
def get_user(): ...
```

اما برای تعریف یک asynchronous function می‌نویسیم:

```python
async def get_user(): ...
```

پس:

```text
def
    function معمولی

async def
    asynchronous function
```

نکته: `async` به Python می‌گوید که این function یک coroutine function است و می‌تواند در محیط asynchronous اجرا شود.

مثلاً در پروژه:

```python
async def main() -> None: ...
```

تابع `main` یک asynchronous function است.

---

## اصلاحیه: Coroutine چیست؟

وقتی یک `async def` را صدا می‌زنیم، function مثل یک function معمولی فوراً اجرا نمی‌شود.

مثلاً:

```python
async def get_user():
    return "Mehran"


result = get_user()
```

اینجا `result` مقدار `"Mehran"` نیست.

بلکه یک coroutine object است که باید توسط event loop اجرا شود.

به زبان ساده:

```text
async def
    ↓
coroutine function

calling the function
    ↓
coroutine object

event loop
    ↓
اجرای coroutine
```

برای همین در کد پروژه با `await` مواجه می‌شویم.

---

## اصلاحیه: `await` چیست؟

نکته: `await` یعنی:

> اجرای این coroutine را ادامه بده، اما اجازه بده event loop در زمان انتظار کارهای asynchronous دیگری را انجام دهد.

مثلاً:

```python
async def main():
    await dispatcher.start_polling(bot)
```

اینجا `start_polling()` یک عملیات asynchronous است.

ما نمی‌خواهیم آن را مثل یک function معمولی صدا بزنیم:

```python
dispatcher.start_polling(bot)
```

بلکه می‌نویسیم:

```python
await dispatcher.start_polling(bot)
```

چون باید coroutine اجرا و مدیریت شود.

---

## چرا `await` برنامه را کامل متوقف نمی‌کند؟

فرض کنیم bot باید از PostgreSQL اطلاعات بگیرد.

اگر برنامه به شکل کاملاً synchronous کار کند:

```text
Python
  ↓
Database query
  ↓
منتظر PostgreSQL
  ↓
دریافت نتیجه
  ↓
ادامه برنامه
```

در زمان انتظار، thread می‌تواند عملاً بیکار باشد.

در مدل asynchronous:

```text
Python
  ↓
Database query
  ↓
منتظر PostgreSQL
  │
  ├── کار دیگری آماده اجراست
  │
  ├── Telegram event دیگری رسیده
  │
  └── operation دیگری آماده اجراست
  ↓
database response
  ↓
ادامه coroutine
```

بنابراین asynchronous programming مخصوصاً برای برنامه‌هایی که مقدار زیادی I/O دارند مفید است.

---

## اصلاحیه: I/O چیست؟

نکته: I/O مخفف:

```text
Input / Output
```

است.

هر کاری که برنامه باید با یک منبع خارجی ارتباط برقرار کند، معمولاً I/O محسوب می‌شود.

مثلاً:

```text
PostgreSQL
Telegram API
HTTP API
File system
Network
```

در `media-bot` این موضوع بسیار مهم است، چون برنامه احتمالاً مقدار زیادی I/O خواهد داشت.

مثلاً:

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

بخش بزرگی از این workflow شامل انتظار برای منابع خارجی است.

---

## اصلاحیه: Event Loop چیست؟

نکته: Event loop را می‌توان به عنوان موتور اجرای asynchronous code در نظر گرفت.



به صورت ساده:

```text
Event Loop
    │
    ├── coroutine A → منتظر database
    │
    ├── coroutine B → آماده اجرا
    │
    ├── coroutine C → منتظر network
    │
    └── coroutine D → آماده اجرا
```

وقتی یک coroutine به یک عملیات I/O می‌رسد و باید منتظر بماند، event loop می‌تواند coroutine دیگری را اجرا کند.

این یکی از پایه‌های asynchronous programming در Python است.

---

## چرا `asyncio.run()` داریم؟

در فایل `main.py` پروژه:

```python
if __name__ == "__main__":
    asyncio.run(main())
```

تابع `main()` یک coroutine function است.

برای اجرای آن باید یک event loop داشته باشیم.

`asyncio.run()` این کار را برای ما انجام می‌دهد.

به صورت مفهومی:

```text
asyncio.run(main())
        ↓
ایجاد event loop
        ↓
اجرای main()
        ↓
مدیریت coroutine
        ↓
پایان event loop
```

پس این دو مفهوم را نباید با هم اشتباه بگیریم:

```python
async def main():
```

تعریف یک asynchronous function است.

در حالی که:

```python
asyncio.run(main())
```

آن coroutine را از نقطه شروع برنامه اجرا می‌کند.

---

## چرا Database هم async است؟

در پروژه از SQLAlchemy با async support استفاده می‌کنیم:

```python
from sqlalchemy.ext.asyncio import AsyncSession
```

و database driver ما:

```text
asyncpg
```

است.

بنابراین queryهای database هم می‌توانند به صورت asynchronous اجرا شوند.

مثلاً:

```python
result = await session.execute(statement)
```

در اینجا:

```text
session.execute(...)
```

یک عملیات asynchronous است.

و:

```python
await
```

به برنامه اجازه می‌دهد در زمان انتظار برای database، event loop بتواند کارهای دیگری را مدیریت کند.

---

## یک اشتباه رایج

این دو را با هم اشتباه نگیریم:

```python
async def get_user(): ...
```

و:

```python
await get_user()
```

اولی function را تعریف می‌کند.

دومی coroutine حاصل از آن function را اجرا/منتظر می‌ماند.

---

## ارتباط این موضوع با Media Bot

در معماری فعلی پروژه:

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

در چند نقطه asynchronous programming داریم.

مثلاً:

```python
async def main():
```

```python
async def get_by_telegram_id(...):
```

```python
result = await session.execute(statement)
```

این‌ها اتفاقی نیستند.

هدف این است که workflowهای I/O-heavy پروژه بتوانند بدون block کردن کل application اجرا شوند.

---

## چیزهایی که فعلاً لازم نیست بدانم

در این مرحله لازم نیست تمام جزئیات `asyncio` را یاد بگیرم.

فعلاً این mental model کافی است:

```text
async def
    ↓
یک asynchronous function تعریف می‌کند

calling async function
    ↓
coroutine می‌سازد

await
    ↓
اجرای coroutine و انتظار asynchronous

event loop
    ↓
مدیریت اجرای coroutineها

I/O
    ↓
جایی که معمولاً asynchronous programming بیشترین فایده را دارد
```

بعداً هنگام پیشرفت پروژه می‌توانیم مفاهیم زیر را جداگانه یاد بگیریم:

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

## ارتباط با فایل‌های پروژه

برای دیدن این مفاهیم در پروژه فعلی:

```text
src/music_bot/main.py
```

شروع asynchronous application را نشان می‌دهد.

```text
src/music_bot/repositories/user.py
```

نشان می‌دهد database operation چگونه asynchronous شده است.

```text
tests/conftest.py
```

نشان می‌دهد چگونه async database engine و `AsyncSession` را در tests استفاده کرده‌ایم.

---

## خلاصه

مهم‌ترین چیزی که فعلاً باید به خاطر داشته باشم:

```text
async def
    ↓
تعریف asynchronous function

await
    ↓
انتظار برای یک عملیات asynchronous

event loop
    ↓
مدیریت اجرای asynchronous کارها

I/O
    ↓
کارهایی مثل database و network که زمان انتظار دارند
```

در `media-bot` از async استفاده می‌کنیم چون بخش بزرگی از برنامه با Telegram، PostgreSQL، network و در آینده دانلود media سروکار خواهد داشت.