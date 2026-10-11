## متغیرها ارجاع‌هایی به حافظه هستند

می‌توانیم آدرس حافظه‌ای که یک متغیر به آن *ارجاع* می‌دهد را با استفاده از تابع `id()` پیدا کنیم.

تابع `id()` آدرس حافظهٔ آرگومان خود را به‌صورت یک عدد صحیح در مبنای 10 بازمی‌گرداند.

می‌توانیم از تابع `hex()` برای تبدیل عدد مبنای 10 به مبنای 16 استفاده کنیم.


```python
my_var = 10
print('my_var = {0}'.format(my_var))
print('memory address of my_var (decimal): {0}'.format(id(my_var)))
print('memory address of my_var (hex): {0}'.format(hex(id(my_var))))
```

    my_var = 10
    memory address of my_var (decimal): 1968827120
    memory address of my_var (hex): 0x7559eaf0



```python
greeting = 'Hello'
print('greeting = {0}'.format(greeting))
print('memory address of my_var (decimal): {0}'.format(id(greeting)))
print('memory address of my_var (hex): {0}'.format(hex(id(greeting))))
```

    greeting = Hello
    memory address of my_var (decimal): 1688681719264
    memory address of my_var (hex): 0x1892d4625e0


توجه کنید که آدرس حافظهٔ `my_var` با آدرس `greeting` **متفاوت** است.

به‌طور دقیق، `my_var` با 10 «برابر» نیست.

در عوض `my_var` یک **ارجاع** به یک شیء (*عدد صحیح*) (*حاوی مقدار 10*) است که در آدرس حافظهٔ `id(my_var)` قرار دارد.

به‌طور مشابه برای متغیر `greeting`.