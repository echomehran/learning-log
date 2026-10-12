### بولین‌ها: تقدم و اتصال کوتاه

```python
True or True and False
```




    True



این معادل است، به دلیل اینکه ``and`` تقدم بالاتری نسبت به ``or`` دارد، با:

```python
True or (True and False)
```




    True



این با موارد زیر یکسان نیست:

```python
(True or True) and False
```




    False



#### اتصال کوتاه

```python
a = 10
b = 2

if a/b > 2:
    print('a is at least double b')
```

    a is at least double b

```python
a = 10
b = 0

if a/b > 2:
    print('a is at least double b')
```

    ---------------------------------------------------------------------------
    ZeroDivisionError                         Traceback (most recent call last)
    <ipython-input-12-98ac73a1accd> in <module>()
          2 b = 0
          3 
    ----> 4 if a/b > 2:
          5     print('a is at least double b')

    ZeroDivisionError: division by zero

```python
a = 10
b = 0

if b and a/b > 2:
    print('a is at least double b')
```

همچنین می‌تواند برای مقابله با رشته‌های null یا خالی در یک پایگاه داده مفید باشد:

```python
import string
```

```python
help(string)
```

    Help on module string:
    
    NAME
        string - A collection of string constants.
    
    DESCRIPTION
        Public module variables:
        
        whitespace -- a string containing all ASCII whitespace
        ascii_lowercase -- a string containing all ASCII lowercase letters
        ascii_uppercase -- a string containing all ASCII uppercase letters
        ascii_letters -- a string containing all ASCII letters
        digits -- a string containing all ASCII decimal digits
        hexdigits -- a string containing all ASCII hexadecimal digits
        octdigits -- a string containing all ASCII octal digits
        punctuation -- a string containing all ASCII punctuation characters
        printable -- a string containing all ASCII characters considered printable
    
    CLASSES
        builtins.object
            Formatter
            Template
        
        class Formatter(builtins.object)
         |  Methods defined here:
         |  
         |  check_unused_args(self, used_args, args, kwargs)
         |  
         |  convert_field(self, value, conversion)
         |  
         |  format(*args, **kwargs)
         |  
         |  format_field(self, value, format_spec)
         |  
         |  get_field(self, field_name, args, kwargs)
         |      # given a field_name, find the object it references.
         |      #  field_name:   the field being looked up, e.g. "0.name"
         |      #                 or "lookup[3]"
         |      #  used_args:    a set of which args have been used
         |      #  args, kwargs: as passed in to vformat
         |  
         |  get_value(self, key, args, kwargs)
         |  
         |  parse(self, format_string)
         |      # returns an iterable that contains tuples of the form:
         |      # (literal_text, field_name, format_spec, conversion)
         |      # literal_text can be zero length
         |      # field_name can be None, in which case there's no
         |      #  object to format and output
         |      # if field_name is not None, it is looked up, formatted
         |      #  with format_spec and conversion and then used
         |  
         |  vformat(self, format_string, args, kwargs)
         |  
         |  ----------------------------------------------------------------------
         |  Data descriptors defined here:
         |  
         |  __dict__
         |      dictionary for instance variables (if defined)
         |  
         |  __weakref__
         |      list of weak references to the object (if defined)
        
        class Template(builtins.object)
         |  A string class for supporting $-substitutions.
         |  
         |  Methods defined here:
         |  
         |  __init__(self, template)
         |      Initialize self.  See help(type(self)) for accurate signature.
         |  
         |  safe_substitute(*args, **kws)
         |  
         |  substitute(*args, **kws)
         |  
         |  ----------------------------------------------------------------------
         |  Data descriptors defined here:
         |  
         |  __dict__
         |      dictionary for instance variables (if defined)
         |  
         |  __weakref__
         |      list of weak references to the object (if defined)
         |  
         |  ----------------------------------------------------------------------
         |  Data and other attributes defined here:
         |  
         |  delimiter = '$'
         |  
         |  flags = <RegexFlag.IGNORECASE: 2>
         |  
         |  idpattern = '[_a-z][_a-z0-9]*'
         |  
         |  pattern = re.compile('\\n    \\\\$(?:\\n      (?P<escaped>\\\\$)..._a-z][_a-...
    
    FUNCTIONS
        capwords(s, sep=None)
            capwords(s [,sep]) -> string
            
            Split the argument into words using split, capitalize each
            word using capitalize, and join the capitalized words using
            join.  If the optional second argument sep is absent or None,
            runs of whitespace characters are replaced by a single space
            and leading and trailing whitespace are removed, otherwise
            sep is used to split and join the words.
    
    DATA
        __all__ = ['ascii_letters', 'ascii_lowercase', 'ascii_uppercase', 'cap...
        ascii_letters = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
        ascii_lowercase = 'abcdefghijklmnopqrstuvwxyz'
        ascii_uppercase = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
        digits = '0123456789'
        hexdigits = '0123456789abcdefABCDEF'
        octdigits = '01234567'
        printable = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTU...'
        punctuation = '!"#$%&\'()*+,-./:;<=>?@[\\]^_`{|}~'
        whitespace = ' \t\n\r\x0b\x0c'
    
    FILE
        c:\users\fbapt\anaconda3\envs\deepdive\lib\string.py

```python
string.digits
```




    '0123456789'



```python
string.ascii_letters
```




    'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'



```python
name = ''
if name[0] in string.digits:
    print('Name cannot start with a digit!')
```

    ---------------------------------------------------------------------------
    IndexError                                Traceback (most recent call last)
    <ipython-input-19-b6d1fe6f1f39> in <module>()
          1 name = ''
    ----> 2 if name[0] in string.digits:
          3     print('Name cannot start with a digit!')

    IndexError: string index out of range

```python
name = ''
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = None
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = 'Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

```python
name = '1Bob'
if name and name[0] in string.digits:
    print('Name cannot start with a digit!')
```

    Name cannot start with a digit!
