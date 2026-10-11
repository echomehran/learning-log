## بهینه‌سازی‌های پایتون: درون‌سپاری رشته

پایتون به‌طور خودکار *برخی* رشته‌ها را درون‌سپاری می‌کند.

به‌طور خاص، همهٔ شناسه‌ها (نام متغیرها، نام توابع، نام کلاس‌ها و غیره) درون‌سپاری می‌شوند (اشیاء تک‌نمونه ایجاد می‌شوند).

پایتون همچنین رشته‌های ثابتی را که شبیه شناسه‌ها هستند درون‌سپاری می‌کند.

برای مثال:


```python
a = 'hello'
b = 'hello'
print(id(a))
print(id(b))
```

    1342722069536
    1342722069536


اما موارد زیر را درون‌سپاری نمی‌کند:


```python
a = 'hello, world!'
b = 'hello, world!'
print(id(a))
print(id(b))
```

    1342722047024
    1342722170928


با این حال، چون ثابت‌های زیر شبیه شناسه‌ها هستند، حتی اگر نسبتاً طولانی باشند، پایتون همچنان به‌طور خودکار آن‌ها را درون‌سپاری می‌کند:


```python
a = 'hello_world'
b = 'hello_world'
print(id(a))
print(id(b))
```

    1342722047856
    1342722047856


و حتی طولانی‌تر:


```python
a = '_this_is_a_long_string_that_could_be_used_as_an_identifier'
b = '_this_is_a_long_string_that_could_be_used_as_an_identifier'
print(id(a))
print(id(b))
```

    1342721886784
    1342721886784


حتی اگر رشته با یک رقم شروع شود:


```python
a = '1_hello_world'
b = '1_hello_world'
print(id(a))
print(id(b))
```

    1342722046256
    1342722046256


این مورد درون‌سپاری شد (اشاره‌گر یکسان است)، اما به این یکی نگاه کنید:


```python
a = '1 hello world'
b = '1 hello world'
print(id(a))
print(id(b))
```

    1342722046832
    1342722172592


درون‌سپاری رشته‌ها (تبدیل آن‌ها به اشیاء تک‌نمونه) به این معناست که آزمون برابری رشته می‌تواند با مقایسهٔ آدرس حافظه سریع‌تر انجام شود:


```python
a = 'this_is_a_long_string'
b = 'this_is_a_long_string'
print('a==b:', a == b)
print('a is b:', a is b)
```

    a==b: True
    a is b: True


#### <font color="orange">توجه: به یاد داشته باشید، استفاده از `is` فقط زمانی کار می‌کند که رشته‌ها درون‌سپاری شده باشند!</font>

اینجاست که این تکنیک شکست می‌خورد:


```python
a = 'hello world'
b = 'hello world'
print('a==b:', a==b)
print('a is b:', a is b)
```

    a==b: True
    a is b: False


شما *می‌توانید* رشته‌ها را مجبور به درون‌سپاری کنید (اما فقط در صورتی از آن استفاده کنید که نیاز معتبری به بهینه‌سازی عملکرد داشته باشید):


```python
import sys
```


```python
a = sys.intern('hello world')
b = sys.intern('hello world')
c = 'hello world'
print(id(a))
print(id(b))
print(id(c))
```

    1342722172080
    1342722172080
    1342722174896


توجه کنید که `a` و `b` به همان شیء اشاره می‌کنند، اما `c` **این‌طور نیست**.

بنابراین، چون هم `a` و هم `b` درون‌سپاری شده‌اند، می‌توانیم از `is` برای آزمون برابری دو رشته استفاده کنیم:


```python
print('a==b:', a==b)
print('a is b:', a is b)
```

    a==b: True
    a is b: True


پس آیا درون‌سپاری واقعاً تفاوت سرعت زیادی ایجاد می‌کند؟

بله، اما فقط در صورتی که تعداد *زیادی* مقایسه انجام دهید.

بیایید چند بنچمارک سریع و خام اجرا کنیم:


```python
def compare_using_equals(n):
    a = 'a long string that is not interned' * 200
    b = 'a long string that is not interned' * 200
    for i in range(n):
        if a == b:
            pass
```


```python
def compare_using_interning(n):
    a = sys.intern('a long string that is not interned' * 200)
    b = sys.intern('a long string that is not interned' * 200)
    for i in range(n):
        if a is b:
            pass
```


```python
import time

start = time.perf_counter()
compare_using_equals(10000000)
end = time.perf_counter()

print('equality: ', end-start)

```

    equality:  2.965451618090112



```python
start = time.perf_counter()
compare_using_interning(10000000)
end = time.perf_counter()

print('identity: ', end-start)
```

    identity:  0.28690104431129626


همان‌طور که می‌بینید، تفاوت عملکرد، به‌ویژه برای رشته‌های طولانی و برای مقایسه‌های زیاد، می‌تواند کاملاً چشمگیر باشد!