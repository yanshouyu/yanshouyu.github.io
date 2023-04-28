---
layout: post
title: "Book notes: Fluent Python, 2nd edition. Part 2 - Functions as Objects"
---


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