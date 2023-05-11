---
layout: post
title: "Book notes: Fluent Python, 2nd edition. Part 2 - Functions as Objects"
---

230511: Reading in progress.

# Part 2 - Functions as Objects

## Chapter 7. Functions as First-Class Objects

Higher-order function means the function can take function(s) as its argument. For example, the built-in `map` (`map(func, *iterables)`) and `sorted` (`sorted(iterable, /, *, key=None, reverse=False`).

`all([])` returns `True` - it only cares if there is `False` in the argument iterable.

To determine if an object is callable, use the built-in `callable()` which returns a boolean value.

### Keyword-Only and Positional-Only Arguments

Keyword-only arguments are arguments that can only be assigned via keywords explicitly. To specify keyword-only arguments, put a single `*` argument before them. For example, in the built-in higher order function `sorted(iterable, /, *, key=None, reverse=False`, the `key` and `reverse` are keyword-only arguments.

Positional-only arguments are arguments that can only be accessed via position. In function definition they are put before a single `/`. From the same example `sorted(iterable, /, *, key=None, reverse=False`, we know that the `iterable` argument is positional-only and cannot be assigned like `sorted(iterable=[...])`.

### Module `operator`

The `operator` module, as its name implies, *exports a set of functions implemented in C corresponding to the intrinsic operators of Python*. It saves efforts in rewriting anonymous lambda functions for basic operators, e.g. `lambda a, b: a+b` can be replaced by `operator.add`, `lambda it: it[1], it[0]` can be replaced by `operator.itemgetter(1, 0)`. 

Similar to `operator.itemgetter` there is an `operator.attrgetter`. `attrgetter` can get nested objects from `.` connected argument name. Note: don't confuse the key of `dict` from the attribute of an object.

### Module `functools`

`functools.reduce` cumulatively applies a function of 2 arguments from start to end. If argument `initial` is provided, the first operation is between `initial` and the first element. Otherwise the first operation is between the first and second element.

`functools.partial` can be used to create a new callable with certain arguments fixed.

## Chapter 8. Type Hints in Functions

Type hints are ignored at runtime. 

Not only type hints can be used in function arguments, but also in variable declaration.

`mypy` is one of the widely used Python type checkers.

The `typing.TYPE_CHECKING` constant is always `False` at runtime. Type checkers like `mypy` can pretend `typing.TYPE_CHECKING` is True and call `reveal_type` to output debugging type messages like this one:
```
comparable/top_test.py:32: note: Revealed type is "typing.Iterator[Tuple[builtins.int, builtins.str]]"
```
Note that `reveal_type()` is a mypy debugging facility thus does not require to be imported.

[`flake8`](https://pypi.org/project/flake8/) is a easy-to-use coding style guide tool.

*duck typing* and *nominal typing*.


### Types Usable in Annotations

Since Python 3.10, `|` operator can be used as logical OR in type hints. For example, `int | str` can replace the `typing.Union[str, int]`. Also in `isinstance` and `issubclass`, expressions like `isinstance(str | int)` are valid.

Coding tip: avoid creating functions that return `Union` types to save the user from checking the returned type.

#### Generic Collections and Mappings

Since Python 3.9, collection types (`list`, `set`, `collections.abc.Sequence`, etc.) can be used in type hints directly. Redundant generic types in `typing` module (e.g. `typing.List`, `typing.Sequence`) are deprecated.

Similar to generic collections, built-in `dict` and mapping types in `collections` and `collections.abc` can be used in type hints directly since python 3.9. In earlier versions `typing.Dict` is used.

#### Tuple

Type hint `stuff: tuple[int, ...]` means `stuff` is a tuple having >= 1 int elements.

*type alias* (e.g. `Lat_Lon = tuple[float, float]`) can make the code more readable. A preferred way to create type alias from python 3.10 would be using `typing.TypeAlias`, e.g. `Lat_Lon: TypeAlias = tuple[float, float]`.

`typing.NamedTuple` is used where we want to annotate a tuple with named fields. When the tuple contains many fields or is to be reused, `NamedTuple` makes the code clear.

#### Abstract Base Classes

Abstract base classes "give more flexibility to the caller". For example, `collections.abc.Sequence` or `collections.abc.Iterable` (which to use depends on what you need) are better in type-hinting arguments than `list`. Similar case in `collections.abc.Mapping` and `collections.abc.MutableMapping` vs. `dict`.

However, when type-hinting returned values, it is better to be specific as possible.

#### Parameterized Generics and TypeVar

`typing.TypeVar`'s simple usage from its doc:

```
T = TypeVar('T')  # Can be anything
A = TypeVar('A', str, bytes)  # Must be str or bytes
```

Type variable can be used when we want to refer the same parameter type to the result type.

#### Static Protocols

> "a protocol type is defined by specifying one or more methods"

Subclassing a `typing.Protocol` and define supported methods. Then this subclass can be used as `bound` in defining a `TypeVar`.

#### Callable
