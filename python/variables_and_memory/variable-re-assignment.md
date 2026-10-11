## تخصیص مجدد متغیر

توجه کنید که آدرس حافظهٔ **a** هر بار متفاوت است.


```python
a = 10
hex(id(a))
```




    '0x7559eaf0'




```python
a = 15
hex(id(a))
```




    '0x7559eb90'




```python
a = 5
hex(id(a))
```




    '0x7559ea50'




```python
a = a + 1
hex(id(a))
```




    '0x7559ea70'



با این حال، به این نگاه کنید:


```python
a = 10
b = 10
print(hex(id(a)))
print(hex(id(b)))
```

    0x7559eaf0
    0x7559eaf0


آدرس حافظهٔ هر دو **a** و **b** یکسان است!!
