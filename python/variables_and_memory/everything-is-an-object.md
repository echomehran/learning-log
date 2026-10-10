## همه‌چیز یک شیء است


```python
a = 10
```
نکته: **a** یک شیء از نوع **int** است، یعنی **a** یک نمونه از کلاس **int** است.


```python
print(type(a))
```

    <class 'int'>


اگر **int** یک کلاس باشد، باید بتوانیم آن را با استفاده از روش استاندارد نمونه‌سازی کلاس تعریف کنیم:


```python
b = int(10)
```


```python
print(b)
print(type(b))
```

    10
    <class 'int'>


حتی می‌توانیم مستندات کلاس را درخواست کنیم:


```python
help(int)
```

    Help on class int in module builtins:
    
    class int(object)
     |  int(x=0) -> integer
     |  int(x, base=10) -> integer
     |  
     |  Convert a number or string to an integer, or return 0 if no arguments
     |  are given.  If x is a number, return x.__int__().  For floating point
     |  numbers, this truncates towards zero.
     |  
     |  If x is not a number or if base is given, then x must be a string,
     |  bytes, or bytearray instance representing an integer literal in the
     |  given base.  The literal can be preceded by '+' or '-' and be surrounded
     |  by whitespace.  The base defaults to 10.  Valid bases are 0 and 2-36.
     |  Base 0 means to interpret the base from the string as an integer literal.
     |  >>> int('0b100', base=0)
     |  4
     |  
     |  Methods defined here:
     |  
     |  __abs__(self, /)
     |      abs(self)
     |  
     |  __add__(self, value, /)
     |      Return self+value.
     |  
     |  __and__(self, value, /)
     |      Return self&value.
     |  
     |  __bool__(self, /)
     |      self != 0
     |  
     |  __ceil__(...)
     |      Ceiling of an Integral returns itself.
     |  
     |  __divmod__(self, value, /)
     |      Return divmod(self, value).
     |  
     |  __eq__(self, value, /)
     |      Return self==value.
     |  
     |  __float__(self, /)
     |      float(self)
     |  
     |  __floor__(...)
     |      Flooring an Integral returns itself.
     |  
     |  __floordiv__(self, value, /)
     |      Return self//value.
     |  
     |  __format__(...)
     |      default object formatter
     |  
     |  __ge__(self, value, /)
     |      Return self>=value.
     |  
     |  __getattribute__(self, name, /)
     |      Return getattr(self, name).
     |  
     |  __getnewargs__(...)
     |  
     |  __gt__(self, value, /)
     |      Return self>value.
     |  
     |  __hash__(self, /)
     |      Return hash(self).
     |  
     |  __index__(self, /)
     |      Return self converted to an integer, if self is suitable for use as an index into a list.
     |  
     |  __int__(self, /)
     |      int(self)
     |  
     |  __invert__(self, /)
     |      ~self
     |  
     |  __le__(self, value, /)
     |      Return self<=value.
     |  
     |  __lshift__(self, value, /)
     |      Return self<<value.
     |  
     |  __lt__(self, value, /)
     |      Return self<value.
     |  
     |  __mod__(self, value, /)
     |      Return self%value.
     |  
     |  __mul__(self, value, /)
     |      Return self*value.
     |  
     |  __ne__(self, value, /)
     |      Return self!=value.
     |  
     |  __neg__(self, /)
     |      -self
     |  
     |  __new__(*args, **kwargs) from builtins.type
     |      Create and return a new object.  See help(type) for accurate signature.
     |  
     |  __or__(self, value, /)
     |      Return self|value.
     |  
     |  __pos__(self, /)
     |      +self
     |  
     |  __pow__(self, value, mod=None, /)
     |      Return pow(self, value, mod).
     |  
     |  __radd__(self, value, /)
     |      Return value+self.
     |  
     |  __rand__(self, value, /)
     |      Return value&self.
     |  
     |  __rdivmod__(self, value, /)
     |      Return divmod(value, self).
     |  
     |  __repr__(self, /)
     |      Return repr(self).
     |  
     |  __rfloordiv__(self, value, /)
     |      Return value//self.
     |  
     |  __rlshift__(self, value, /)
     |      Return value<<self.
     |  
     |  __rmod__(self, value, /)
     |      Return value%self.
     |  
     |  __rmul__(self, value, /)
     |      Return value*self.
     |  
     |  __ror__(self, value, /)
     |      Return value|self.
     |  
     |  __round__(...)
     |      Rounding an Integral returns itself.
     |      Rounding with an ndigits argument also returns an integer.
     |  
     |  __rpow__(self, value, mod=None, /)
     |      Return pow(value, self, mod).
     |  
     |  __rrshift__(self, value, /)
     |      Return value>>self.
     |  
     |  __rshift__(self, value, /)
     |      Return self>>value.
     |  
     |  __rsub__(self, value, /)
     |      Return value-self.
     |  
     |  __rtruediv__(self, value, /)
     |      Return value/self.
     |  
     |  __rxor__(self, value, /)
     |      Return value^self.
     |  
     |  __sizeof__(...)
     |      Returns size in memory, in bytes
     |  
     |  __str__(self, /)
     |      Return str(self).
     |  
     |  __sub__(self, value, /)
     |      Return self-value.
     |  
     |  __truediv__(self, value, /)
     |      Return self/value.
     |  
     |  __trunc__(...)
     |      Truncating an Integral returns itself.
     |  
     |  __xor__(self, value, /)
     |      Return self^value.
     |  
     |  bit_length(...)
     |      int.bit_length() -> int
     |      
     |      Number of bits necessary to represent self in binary.
     |      >>> bin(37)
     |      '0b100101'
     |      >>> (37).bit_length()
     |      6
     |  
     |  conjugate(...)
     |      Returns self, the complex conjugate of any int.
     |  
     |  from_bytes(...) from builtins.type
     |      int.from_bytes(bytes, byteorder, *, signed=False) -> int
     |      
     |      Return the integer represented by the given array of bytes.
     |      
     |      The bytes argument must be a bytes-like object (e.g. bytes or bytearray).
     |      
     |      The byteorder argument determines the byte order used to represent the
     |      integer.  If byteorder is 'big', the most significant byte is at the
     |      beginning of the byte array.  If byteorder is 'little', the most
     |      significant byte is at the end of the byte array.  To request the native
     |      byte order of the host system, use `sys.byteorder' as the byte order value.
     |      
     |      The signed keyword-only argument indicates whether two's complement is
     |      used to represent the integer.
     |  
     |  to_bytes(...)
     |      int.to_bytes(length, byteorder, *, signed=False) -> bytes
     |      
     |      Return an array of bytes representing an integer.
     |      
     |      The integer is represented using length bytes.  An OverflowError is
     |      raised if the integer is not representable with the given number of
     |      bytes.
     |      
     |      The byteorder argument determines the byte order used to represent the
     |      integer.  If byteorder is 'big', the most significant byte is at the
     |      beginning of the byte array.  If byteorder is 'little', the most
     |      significant byte is at the end of the byte array.  To request the native
     |      byte order of the host system, use `sys.byteorder' as the byte order value.
     |      
     |      The signed keyword-only argument determines whether two's complement is
     |      used to represent the integer.  If signed is False and a negative integer
     |      is given, an OverflowError is raised.
     |  
     |  ----------------------------------------------------------------------
     |  Data descriptors defined here:
     |  
     |  denominator
     |      the denominator of a rational number in lowest terms
     |  
     |  imag
     |      the imaginary part of a complex number
     |  
     |  numerator
     |      the numerator of a rational number in lowest terms
     |  
     |  real
     |      the real part of a complex number
    


همان‌طور که از مستندات می‌بینیم، حتی می‌توانیم یک **int** را با استفاده از سازندهٔ سربارگذاری‌شده ایجاد کنیم:


```python
b = int('10', base=2)
```


```python
print(b)
print(type(b))
```

    2
    <class 'int'>


### توابع نیز شیء هستند
---


```python
def square(a):
    return a ** 2
```


```python
type(square)
```




    function



در واقع، حتی می‌توانیم آن‌ها را به یک متغیر نسبت دهیم:


```python
f = square
```


```python
type(f)
```




    function




```python
f is square
```




    True




```python
f(2)
```




    4




```python
type(f(2))
```




    int



یک تابع می‌تواند یک تابع بازگرداند


```python
def cube(a):
    return a ** 3
```


```python
def select_function(fn_id):
    if fn_id == 1:
        return square
    else:
        return cube
```


```python
f = select_function(1)
print(hex(id(f)))
print(hex(id(square)))
print(hex(id(cube)))
print(type(f))
print('f is square: ', f is square)
print('f is cube: ', f is cube)
print(f)
print(f(2))
```

    0x21257457b70
    0x21257457b70
    0x21255fab8c8
    <class 'function'>
    f is square:  True
    f is cube:  False
    <function square at 0x0000021257457B70>
    4



```python
f = select_function(2)
print(hex(id(f)))
print(hex(id(square)))
print(hex(id(cube)))
print(type(f))
print('f is square: ', f is square)
print('f is cube: ', f is cube)
print(f)
print(f(2))
```

    0x21255fab8c8
    0x21257457b70
    0x21255fab8c8
    <class 'function'>
    f is square:  False
    f is cube:  True
    <function cube at 0x0000021255FAB8C8>
    8


حتی می‌توانستیم آن را به این شکل فراخوانی کنیم:


```python
select_function(1)(5)
```




    25



یک تابع می‌تواند به‌عنوان آرگومان به تابع دیگری پاس داده شود

(این مثال کاملاً بی‌فایده است، اما نکته را به‌خوبی نشان می‌دهد)


```python
def exec_function(fn, n):
    return fn(n)
```


```python
result = exec_function(cube, 2)
print(result)
```

    8


ما در طول این دوره **بسیار** بیشتر به توابع به‌عنوان آرگومان بازخواهیم گشت!