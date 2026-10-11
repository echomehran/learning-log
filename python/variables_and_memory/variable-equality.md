## برابری متغیرها

از درس قبلی می‌دانیم که **a** و **b** یک ارجاع **مشترک** خواهند داشت:


```python
a = 10
b = 10

print(hex(id(a)))
print(hex(id(b)))
```

    0x7559eaf0
    0x7559eaf0


وقتی از عملگر **is** استفاده می‌کنیم، **ارجاع‌های** آدرس حافظه را مقایسه می‌کنیم:


```python
print("a is b: ", a is b)
```

    a is b:  True


اما اگر از عملگر **==** استفاده کنیم، **محتویات** را مقایسه می‌کنیم:


```python
print("a == b:", a == b)
```

    a == b: True


با این حال، موارد زیر ارجاع مشترک ندارند:


```python
a = [1, 2, 3]
b = [1, 2, 3]

print(hex(id(a)))
print(hex(id(b)))
```

    0x27006f17288
    0x27006e968c8


اگرچه آن‌ها اشیاء یکسانی نیستند، اما «مقادیر» یکسانی دارند:


```python
print("a is b: ", a is b)
print("a == b", a == b)
```

    a is b:  False
    a == b True


پایتون تلاش می‌کند مقادیر را به بهترین شکل ممکن مقایسه کند، برای مثال:


```python
a = 10
b = 10.0
```

این‌ها ارجاع یکسانی **نیستند**، زیرا یک شیء **int** و دیگری **float** است


```python
print(type(a))
print(type(b))
```

    <class 'int'>
    <class 'float'>



```python
print(hex(id(a)))
print(hex(id(b)))
```

    0x7559eaf0
    0x270064b1870



```python
print('a is b:', a is b)
print('a == b:', a == b)
```

    a is b: False
    a == b: True


بنابراین، حتی اگر *a* یک عدد صحیح 10 باشد و *b* یک عدد اعشاری 10.0 باشد، مقادیر همچنان برابر مقایسه خواهند شد.

در واقع، این مورد نیز همین رفتار را خواهد داشت:


```python
c = 10 + 0j
print(type(c))
```

    <class 'complex'>



```python
print('a is c:', a is c)
print('a == c:', a == c)
```

    a is c: False
    a == c: True


### شیء None
----
`None`

نکته: **None** یک «متغیر» داخلی از نوع *NoneType* است.

اساساً کلمهٔ کلیدی **None** یک ارجاع به یک نمونه شیء از *NoneType* است.

اشیاء NoneType تغییرناپذیر هستند! بنابراین مدیر حافظهٔ پایتون از ارجاع‌های مشترک به شیء None استفاده خواهد کرد.


```python
print(None)
```

    None



```python
hex(id(None))
```




    '0x75576bc0'




```python
type(None)
```




    NoneType




```python
a = None
print(type(a))
print(hex(id(a)))
```

    <class 'NoneType'>
    0x75576bc0



```python
a is None
```




    True




```python
a == None
```




    True




```python
b = None
hex(id(b))
```




    '0x75576bc0'




```python
a is b
```




    True




```python
a == b
```




    True




```python
l = []
```


```python
type(l)
```




    list




```python
l is None
```




    False




```python
l == None
```




    False