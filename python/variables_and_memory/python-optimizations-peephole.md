## بهینه‌سازی‌های سوراخ‌سوزنی پایتون

بهینه‌سازی‌های سوراخ‌سوزنی به دستهٔ خاصی از راهبردهای بهینه‌سازی اشاره دارند که پایتون در هر یک از مراحل کامپایل به کار می‌گیرد.

#### عبارات ثابت

بیایید ببینیم پایتون عبارات ثابت را برای اهداف بهینه‌سازی چگونه کاهش می‌دهد:


```python
def my_func():
    a = 24 * 60
    b = (1, 2) * 5
    c = 'abc' * 3
    d = 'ab' * 11
    e = 'the quick brown fox' * 10
    f = [1, 2] * 5

```


```python
my_func.__code__.co_consts
```




    (None,
     24,
     60,
     1,
     2,
     5,
     'abc',
     3,
     'ab',
     11,
     'the quick brown fox',
     10,
     1440,
     (1, 2),
     (1, 2, 1, 2, 1, 2, 1, 2, 1, 2),
     'abcabcabc')



همان‌طور که در مثال بالا می‌بینید، `24 * 60` از پیش محاسبه و به‌عنوان یک ثابت ذخیره شده است (`1440`).

به‌طور مشابه، `(1, 2) * 5` به‌عنوان `(1, 2, 1, 2, 1, 2, 1, 2, 1, 2)` ذخیره شده و `'abc' * 3` به‌عنوان `abcabcabc` ذخیره شده است.

از طرف دیگر، توجه کنید که `'the quick brown fox' * 10` از پیش محاسبه **نشده** است (بسیار طولانی است).

به‌طور مشابه `[1, 2] * 5` نیز از پیش محاسبه نشده است، زیرا لیست *تغییرپذیر* است و بنابراین یک *ثابت* نیست.

#### آزمون‌های عضویت

در آزمون عضویت، بهینه‌سازی‌ها همان‌طور که در زیر می‌بینید اعمال می‌شوند:


```python
def my_func():
    if e in [1, 2, 3]:
        pass
```


```python
my_func.__code__.co_consts
```




    (None, 1, 2, 3, (1, 2, 3))



همان‌طور که می‌بینید، لیست تغییرپذیر `[1, 2, 3]` به یک تاپل تغییرناپذیر تبدیل شده است.

انجام این کار در اینجا اشکالی ندارد، زیرا ما عضویت لیست را **در آن نقطه از زمان** آزمون می‌کنیم، بنابراین تبدیل آن به یک تاپل ایمن است و کارآمدتر از آزمون عضویت یک لیست است.

به همین ترتیب، عضویت مجموعه به عضویت مجموعهٔ منجمد تبدیل خواهد شد:


```python
def my_func():
    if e in {1, 2, 3}:
        pass
```


```python
my_func.__code__.co_consts
```




    (None, 1, 2, 3, frozenset({1, 2, 3}))



به‌طور کلی، هنگام نوشتن کد، اگر می‌توانید از آزمون عضویت **مجموعه** استفاده کنید، آن را به لیست یا تاپل ترجیح دهید — این کار بسیار کارآمدتر است.

بیایید یک بنچمارک کوچک سریع (و خام) از این موضوع انجام دهیم:


```python
import string
import time 

char_list = list(string.ascii_letters)
char_tuple = tuple(string.ascii_letters)
char_set = set(string.ascii_letters)

print(char_list)
print()
print(char_tuple)
print()
print(char_set)
```

    ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z']
    
    ('a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z')
    
    {'l', 'p', 'x', 'R', 'j', 'S', 's', 'T', 'W', 'Y', 'Z', 'P', 'g', 'O', 'b', 'u', 'H', 'G', 'v', 'e', 'M', 'n', 'w', 't', 'Q', 'E', 'N', 'X', 'C', 'i', 'A', 'B', 'F', 'V', 'a', 'm', 'r', 'f', 'h', 'U', 'D', 'c', 'y', 'z', 'J', 'd', 'o', 'I', 'L', 'K', 'k', 'q'}



```python
def membership_test(n, container):
    for i in range(n):
        if 'p' in container:
            pass
```


```python
start = time.perf_counter()
membership_test(10000000, char_list)
end = time.perf_counter()
print('list membership: ', end-start)
```

    list membership:  2.6035404184015434



```python
start = time.perf_counter()
membership_test(10000000, char_tuple)
end = time.perf_counter()
print('tuple membership: ', end-start)
```

    tuple membership:  2.602491734651276



```python
start = time.perf_counter()
membership_test(10000000, char_set)
end = time.perf_counter()
print('set membership: ', end-start)
```

    set membership:  0.3743007599607324


همان‌طور که می‌بینید، آزمون‌های عضویت مجموعه بسیار سریع‌تر اجرا می‌شوند — که تعجب‌آور نیست، زیرا آن‌ها اساساً اشیائی شبیه دیکشنری هستند، بنابراین از نگاشت‌های درهم برای جستجوی یک عنصر جهت تعیین عضویت استفاده می‌شود.