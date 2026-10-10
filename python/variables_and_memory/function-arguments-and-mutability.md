## آرگومان‌های تابع و تغییرپذیری

تابعی را در نظر بگیرید که یک آرگومان از نوع *رشته* دریافت می‌کند و آرگومان را به‌نوعی تغییر می‌دهد:

```python
def process(s):
    print('initial s # = {0}'.format(hex(id(s))))
    s = s + ' world'
    print('s after change # = {0}'.format(hex(id(s))))
```

```python
my_var = 'hello'
print('my_var # = {0}'.format(hex(id(my_var))))
```

    my_var # = 0x1e7e96fc420

توجه کنید که وقتی *s* دریافت می‌شود، به همان شیئی ارجاع می‌دهد که *my_var* به آن ارجاع می‌دهد.

پس از اینکه *s* را «تغییر» می‌دهیم، *s* به یک آدرس حافظهٔ جدید اشاره می‌کند:

```python
process(my_var)
```

    initial s # = 0x1e7e96fc420
    s after change # = 0x1e7e97153b0

و متغیر خودمان *my_var* هنوز به آدرس حافظهٔ اصلی اشاره می‌کند:

```python
print('my_var # = {0}'.format(hex(id(my_var))))
```

    my_var # = 0x1e7e96fc420

بیایید ببینیم این موضوع با اشیاء تغییرپذیر چگونه کار می‌کند:

```python
def modify_list(items):
    print('initial items # = {0}'.format(hex(id(items))))
    if len(items) > 0:
        items[0] = items[0] ** 2
    items.pop()
    items.append(5)
    print('final items # = {0}'.format(hex(id(items))))
```

```python
my_list = [2, 3, 4]
print('my_list # = {0}'.format(hex(id(my_list))))
```

    my_list # = 0x1e7e972d308

```python
modify_list(my_list)
```

    initial items # = 0x1e7e972d308
    final items # = 0x1e7e972d308

```python
print(my_list)
print('my_list # = {0}'.format(hex(id(my_list))))
```

    [4, 3, 5]
    my_list # = 0x1e7e972d308

همان‌طور که می‌بینید، در سراسر کد، آدرس حافظه‌ای که *my_list* و *items* به آن ارجاع می‌دهند همیشه **یکسان** است (ارجاع مشترک) — ما صرفاً محتویات (**حالت داخلی**) شیء را در آن آدرس حافظه تغییر می‌دهیم.

حال، حتی با اشیاء ظرفی تغییرناپذیر هم باید مراقب باشیم، مثلاً یک تاپل حاوی یک لیست (تاپل تغییرناپذیر است، اما عنصر لیست داخل تاپل **تغییرپذیر** است)

```python
def modify_tuple(t):
    print('initial t # = {0}'.format(hex(id(t))))
    t[0].append(100)
    print('final t # = {0}'.format(hex(id(t))))
```

```python
my_tuple = ([1, 2], 'a')
```

```python
hex(id(my_tuple))
```

    '0x1e7e9614288'

```python
modify_tuple(my_tuple)
```

    initial t # = 0x1e7e9614288
    final t # = 0x1e7e9614288

```python
my_tuple
```

    ([1, 2, 100], 'a')

همان‌طور که می‌بینید، عنصر اول تاپل تغییر داده شد.