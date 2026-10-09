## جمع‌آوری زباله (Garbage Collection)

```python
import ctypes
import gc
```

از همان تابعی استفاده می‌کنیم که در درس شمارش ارجاع‌ها (`Reference Counting`) به کار بردیم تا تعداد ارجاع‌های یک شیء مشخص را محاسبه کنیم. برای جلوگیری از ایجاد یک ارجاع اضافی، از آدرس حافظهٔ شیء استفاده می‌کنیم.

```python
def ref_count(address):
    return ctypes.c_long.from_address(address).value
```

تابعی ایجاد می‌کنیم که اشیای موجود در سیستم جمع‌آوری زباله (`GC`) را جست‌وجو کند، شناسهٔ مشخصی را پیدا کند و مشخص کند که آیا شیء موردنظر پیدا شده است یا خیر.

```python
def object_by_id(object_id):
    for obj in gc.get_objects():
        if id(obj) == object_id:
            return "Object exists"
    return "Not found"
```

در مرحلهٔ بعد، دو کلاس تعریف می‌کنیم که از آن‌ها برای ایجاد یک ارجاع چرخه‌ای (`Circular Reference`) استفاده خواهیم کرد.

سازندهٔ کلاس `A` یک نمونه از کلاس `B` ایجاد می‌کند و خودِ نمونهٔ `A` را به سازندهٔ کلاس `B` می‌دهد. سپس کلاس `B` این ارجاع را در یکی از متغیرهای نمونهٔ خود ذخیره می‌کند.

```python
class A:
    def __init__(self):
        self.b = B(self)
        print('A: self: {0}, b:{1}'.format(hex(id(self)), hex(id(self.b))))
```

```python
class B:
    def __init__(self, a):
        self.a = a
        print('B: self: {0}, a: {1}'.format(hex(id(self)), hex(id(self.a))))
```

جمع‌آوری زباله (`GC`) را غیرفعال می‌کنیم تا بتوانیم ببینیم وقتی `GC` اجرا نمی‌شود و زمانی که آن را به‌صورت دستی اجرا می‌کنیم، شمارش ارجاع‌ها چگونه تغییر می‌کند.

```python
gc.disable()
```

اکنون یک نمونه از کلاس `A` ایجاد می‌کنیم. این کار به نوبهٔ خود باعث ایجاد یک نمونه از کلاس `B` می‌شود که ارجاعی به نمونهٔ `A` فراخواننده را ذخیره می‌کند.

```python
my_var = A()
```

```text
B: self: 0x1fc1eae44e0, a: 0x1fc1eae4908
A: self: 0x1fc1eae4908, b:0x1fc1eae44e0
```

همان‌طور که مشاهده می‌کنیم، سازنده‌های کلاس‌های `A` و `B` اجرا شده‌اند. همچنین، از روی آدرس‌های حافظه می‌توانیم متوجه شویم که یک ارجاع چرخه‌ای داریم.

در واقع، متغیر `my_var` نیز به همان نمونه از کلاس `A` ارجاع می‌دهد.

```python
print(hex(id(my_var)))
```

```text
0x1fc1eae4908
```

روش دیگری برای مشاهدهٔ این موضوع وجود دارد:

```python
print('a: \t{0}'.format(hex(id(my_var))))
print('a.b: \t{0}'.format(hex(id(my_var.b))))
print('b.a: \t{0}'.format(hex(id(my_var.b.a))))
```

```text
a: 	0x1fc1eae4908
a.b: 	0x1fc1eae44e0
b.a: 	0x1fc1eae4908
```

```python
a_id = id(my_var)
b_id = id(my_var.b)
```

اکنون می‌توانیم تعداد ارجاع‌های موجود به `a` و `b` را مشاهده کنیم.

```python
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 2
refcount(b) = 1
a: Object exists
b: Object exists
```

همان‌طور که مشاهده می‌کنیم، نمونهٔ `A` دو ارجاع دارد: یکی از طرف `my_var` و دیگری از طرف متغیر نمونه‌ای `b` در نمونهٔ کلاس `B`.

نمونهٔ `B` نیز یک ارجاع دارد که از متغیر نمونه‌ای `a` در نمونهٔ کلاس `A` می‌آید.

اکنون بیایید ارجاعی را که `my_var` به نمونهٔ `A` نگه داشته است، حذف کنیم.

```python
my_var = None
```

```python
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 1
refcount(b) = 1
a: Object exists
b: Object exists
```

همان‌طور که مشاهده می‌کنیم، تعداد ارجاع‌های هر دو شیء اکنون برابر با `1` است؛ یعنی فقط یک ارجاع چرخه‌ای باقی مانده است. شمارش ارجاع‌ها به‌تنهایی نتوانسته نمونه‌های `A` و `B` را از بین ببرد و این دو شیء همچنان در حافظه وجود دارند.

اگر جمع‌آوری زباله انجام نشود، این وضعیت به نشت حافظه (`Memory Leak`) منجر خواهد شد.

اکنون بیایید `GC` را به‌صورت دستی اجرا کنیم و دوباره بررسی کنیم که آیا اشیا هنوز وجود دارند یا خیر.

```python
gc.collect()
print('refcount(a) = {0}'.format(ref_count(a_id)))
print('refcount(b) = {0}'.format(ref_count(b_id)))
print('a: {0}'.format(object_by_id(a_id)))
print('b: {0}'.format(object_by_id(b_id)))
```

```text
refcount(a) = 0
refcount(b) = 0
a: Not found
b: Not found
```
