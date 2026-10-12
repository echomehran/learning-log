### بولین‌ها: مقادیر درستی

همهٔ اشیاء در پایتون یک **مقدار درستی** یا **درستی‌بودن** مرتبط دارند.

در درس قبلی دیدیم که اعداد صحیح یک مقدار درستی ذاتی دارند:

```python
bool(0)
```




    False



```python
bool(1), bool(-1), bool(100)
```




    (True, True, True)



این درستی‌بودن هیچ ارتباطی با این واقعیت ندارد که **bool** زیرکلاسی از **int** است.

در عوض، به این واقعیت مربوط می‌شود که کلاس **int** یک متد `__bool__()` را پیاده‌سازی می‌کند:

```python
help(bool)
```

    Help on class bool in module builtins:
    
    class bool(int)
     |  bool(x) -> bool
     |  
     |  Returns True when the argument x is true, False otherwise.
     |  The builtins True and False are the only two instances of the class bool.
     |  The class bool is a subclass of the class int, and cannot be subclassed.
     |  
     |  Method resolution order:
     |      bool
     |      int
     |      object
     |  
     |  Methods defined here:
     |  
     |  __and__(self, value, /)
     |      Return self&value.
     |  
     |  __new__(*args, **kwargs) from builtins.type
     |      Create and return a new object.  See help(type) for accurate signature.
     |  
     |  __or__(self, value, /)
     |      Return self|value.
     |  
     |  __rand__(self, value, /)
     |      Return value&self.
     |  
     |  __repr__(self, /)
     |      Return repr(self).
     |  
     |  __ror__(self, value, /)
     |      Return value|self.
     |  
     |  __rxor__(self, value, /)
     |      Return value^self.
     |  
     |  __str__(self, /)
     |      Return str(self).
     |  
     |  __xor__(self, value, /)
     |      Return self^value.
     |  
     |  ----------------------------------------------------------------------
     |  Methods inherited from int:
     |  
     |  __abs__(self, /)
     |      abs(self)
     |  
     |  __add__(self, value, /)
     |      Return self+value.
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
     |  __pos__(self, /)
     |      +self
     |  
     |  __pow__(self, value, mod=None, /)
     |      Return pow(self, value, mod).
     |  
     |  __radd__(self, value, /)
     |      Return value+self.
     |  
     |  __rdivmod__(self, value, /)
     |      Return divmod(value, self).
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
     |  __sizeof__(...)
     |      Returns size in memory, in bytes
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
     |  Data descriptors inherited from int:
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

اگر در مستندات به پایین اسکرول کنید باید به بخشی برسید که شبیه این است:

`` 
|  __bool__(self, /)
|      self != 0
``

بنابراین، وقتی می‌نویسیم:

```python
bool(100)
```




    True



پایتون در واقع `100.__bool__()` را فراخوانی می‌کند و آن را بازمی‌گرداند:

```python
(100).__bool__()
```




    True



```python
(0).__bool__()
```




    False



بیشتر اشیاء یا متد `__bool__()` یا متد `__len__()` را پیاده‌سازی می‌کنند. اگر آن‌ها را پیاده‌سازی نکنند، مقدار مرتبط آن‌ها همیشه **True** خواهد بود.

#### انواع عددی

هر مقدار عددی غیرصفر درست است. هر مقدار عددی صفر نادرست است:

```python
from fractions import Fraction
from decimal import Decimal
bool(10), bool(1.5), bool(Fraction(3, 4)), bool(Decimal('10.5'))
```




    (True, True, True, True)



```python
bool(0), bool(0.0), bool(Fraction(0,1)), bool(Decimal('0')), bool(0j)
```




    (False, False, False, False, False)



#### انواع دنباله

یک شیء دنبالهٔ خالی نادرست است، یک شیء غیرخالی درست است:

```python
bool([1, 2, 3]), bool((1, 2, 3)), bool('abc'), bool(1j)
```




    (True, True, True, True)



```python
bool([]), bool(()), bool('')
```




    (False, False, False)



#### انواع نگاشت

به‌طور مشابه، یک نوع نگاشت خالی نادرست خواهد بود، یک نوع غیرخالی درست:

```python
bool({'a': 1}), bool({1, 2, 3})
```




    (True, True)



```python
bool({}), bool(set())
```




    (False, False)



#### شیء None

شیء تک‌نمونهٔ **None** همیشه نادرست است:

```python
bool(None)
```




    False



#### یک کاربرد از مقادیر درستی

هر عبارت شرطی که شامل اشیائی غیر از انواع **bool** باشد، از مقدار درستی مرتبط به‌عنوان نتیجهٔ عبارت شرطی استفاده خواهد کرد.

```python
a = [1, 2, 3]
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

    1

```python
a = []
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

    a is None, or a is empty

```python
a = 'abc'
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

    a

```python
a = ''
if a:
    print(a[0])
else:
    print('a is None, or a is empty')
```

    a is None, or a is empty

می‌توانستیم این را با یک عبارت طولانی‌تر بنویسیم:

```python
a = 'abc'
if a is not None and len(a) > 0:
    print(a[0])
else:
    print('a is None, or a is empty')
```

    a

انجام موارد زیر کد ما را در برخی موارد می‌شکند:

```python
a = 'abc'
if a is not None:
    print(a[0])
```

    a

کار می‌کند، اما:

```python
a = ''
if a is not None:
    print(a[0])
```

    ---------------------------------------------------------------------------
    IndexError                                Traceback (most recent call last)
    <ipython-input-44-47991d9c7397> in <module>()
          1 a = ''
    ----> 2 if a is not None:
          3     print(a[0])

    IndexError: string index out of range

یا حتی:

```python
a = None
if len(a) > 0:
    print(a[0])
```

    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-45-92ec20435e20> in <module>()
          1 a = None
    ----> 2 if len(a) > 0:
          3     print(a[0])

    TypeError: object of type 'NoneType' has no len()

برای کامل بودن باید بنویسیم:

```python
a = None
if a is not None and len(a) > 0:
    print(a[0])
```

همچنین، ترتیب عبارات بولین در اینجا مهم است!


برای مثال:

```python
a = None
if len(a) > 0 and a is not None:
    print(a[0])
```

    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-47-0842480c6625> in <module>()
          1 a = None
    ----> 2 if len(a) > 0 and a is not None:
          3     print(a[0])

    TypeError: object of type 'NoneType' has no len()