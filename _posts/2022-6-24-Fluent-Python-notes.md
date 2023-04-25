---
layout: post
title: "Book notes: Fluent Python, 2nd edition. Part 1 - Data Structures"
---

It's been a long time since I last updates the blog. A lot of things have happened. Time to carry on.

Start taking notes from halfway in chapter 2. Notes will be in a loose format.

# Part 1 - Data Structures

## Chapter 2. An Array of Sequences

* Match / case in python3.10
  * Consider use this new feature in some record / json parsing task. 
* `typing.TypeAlias`
  * Also new in Python 3.10. More typing setting helps to clear thoughts.
* `array.array`
  * Only used array in `numpy` before.
* `hash()` to define if an object is immutable.
* `collections.deque`

## Chapter 3. Dictionaries and Sets

* Merge mapping with `|` and `|=`
  * I only used `dict.update` before. This pattern seems more concise.
* Pattern mapping
  * Saves the efforts in nested "if-else"
  * "Partial mapping": items not in mapping pattern will not be compared.
  * Extra items can be unpacked by applying `**` at the end of pattern
* `dict.setdefault`: If `k` in `d`, return `d[k]`; else set `d[k] = default` and return it. A convinient updating method.
* `collections.defaultdict`
  * `__missing__(k)` and `default_factory`. The `default_factory` is called by `__missing__(k)` to generate a `v` and then the `k, v` is added as an item.
  * `__getitem__` (`dd[k]`) can invoke `default_factory` via `__missing__`, but `get` doesn't.
* `collections.UserDict` is what to inherit when customizing mapping type. Its `data` is a `dict`. 
* "Python’s default behavior is to store instance attributes in a special `__dict__` attribute."
* `frozenset` is hashable
* `dict_keys` and `dict_items` objects also support set operators: &, |, -, ^
  * ^: symmetric difference: (a|b) - (a&b)

## Chapter 4. Unicode Text Versus Bytes

Terms
* Code point: a number for a character.
* Encode: string to binary sequences.
* Decode: binary sequences to string.

UTF-8 is the default source encoding for Python 3.

The python library `chardetect` guesses encoding. It also has a command line entrypoint.

For encodings using more than 1 byte, e.g. UTF-16, byte order is an issue. For UTF-16, a BOM (byte-order mark) is placed at the beginning of encoded bytes to tell the byte order. Little-endian byte ordering has the least significant byte first. Big-endian the opposite. Codec *UTF-16LE* is explicitly little-endian.

The "Unicode sandwich": decode as early as possible -> process by business logic in `str` -> encode as late as possible. Python `open()` built-in handles text files in this way, e.g. the `file.read()` returns `str`.

`isatty()`: method to determine if the `_io.TextIOWrapper` instance is an 'interactive' stream.

```python
>>> sys.stderr.encoding
'utf-8'
>>> sys.stdout.isatty()
True
```

Unicode literal: the "name" of unicode char. Specify unicode literal in `'\N{}'`. E.g. `'\N{INFINITY}'`: ∞; `'\N{CIRCLED NUMBER FORTY TWO}'`: ㊷.  
*Combining characters*: marks add to preceding character. E.g. `'A\N{COMBINING ACUTE ACCENT}'`: Á.

```python
>>> chr(99)
'c'
>>> ord('c')
99
>>> chr(0x1F638)
'😸'
>>> import unicodedata
>>> unicodedata.name(chr(0x1F638))
'GRINNING CAT FACE WITH SMILING EYES'
>>> print('\N{ROMAN NUMERAL TWELVE}')
Ⅻ
>>> unicodedata.numeric('\N{ROMAN NUMERAL TWELVE}')
12.0
```


Sort non-ASCII text
- (not recommended) use `locale.strxfrm` as key in sorting. Locale can be returned by `locale.setlocale`. Warning: this is a global setting.
- (recommended) use `pyuca.Collator.sort_key`. UCA stands for *Unicode Collation Algorithm*.

## Chapter 5. Data Class Builders

- `collections.namedtuple`. Can be used dynamicly. Light-weighted.
- `typing.NamedTuple`. Used for class construction. Child class of a tuple.
- `@dataclasses.dataclass`

To construct a dict from a dataclass (`dataclasses.dataclass` decorated object), use `dataclasses.asdict`.

Type hints are not enforced. They have no impact on the runtime behavior:
```
>>> def func(a: str):
...     print(f'{a} here')
... 
>>> func("str")
str here
>>> func(32)
32 here
```

`@dataclass` tips:
- Make good use of class variable - not all vars should belong to the instance. In the example code the class variable is designed to be a global var. (`cls = self.__class__`)
- A pseudotype `typing.ClassVar` can be used to indicate which field is a class variable.
- `__post_init__` method to manipulate / add more variables after the `__init__`.
- Init-only variables: args passed to instance initialization but not designed to be instance fields. Such variables should be declared as the pseudotype `dataclasses.InitVar`. Init-only variables are also passed to `__post_init__`.

> "The main idea of object-oriented programming is to *place the behavior and data together* in the same code unit: a class."

Serialize dataclass into dict:
- `collections.namedtuple`: `._asdict()`
- `typing.NamedTuple`: `._asdict()`
- `@dataclasses.dataclass`: `.asdict()`

`match case` pattern matching class instances:
```python
# simple class patterns on built-in types
match x:
    case float():    # with () indicates a class pattern
        do_something_with(x)

# keyword class pattern
match x:
    # match the x_arg1 and collect value of x_arg2
    case ClassX(x_arg1="blabla", x_arg2=foo):
        do_something_with(foo)

# position class pattern
match x:
    # match 1st arg, collect 3rd arg
    case ClassX("foo", _, bar):
        do_something_with(bar)
```
Class pattern matching relies on class attribute `__match_args__`.

## Chapter 6. Object References, Mutability, and Recycling

By saying *shallow copy* we are referring to just duplicate the **outer-most container**.

Default shallow copy of a list: `alst = list(blst)` or `alst = blst[:]`. The newly created object has different id, but if it contains mutable object, these inner references are the same.

Deep copy by `deepcopy` method in `copy` module.

Mode of parameter passing in Python is *call by sharing* - each parameter gets a copy of each **reference** in the arguments.

Why using mutable types as parameter defaults is a bad idea: every action on the default affects the `__defaults__` instance. A solution is to use `None` as default value for mutable arguments. When the parameter is `None`, initiate a new mutable parameter so that its reference is not shared.

> The `==` operator compares object values; `is` compares references.