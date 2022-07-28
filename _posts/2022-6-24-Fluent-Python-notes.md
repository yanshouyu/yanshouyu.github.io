---
layout: post
title: "Book notes: Fluent Python, 2nd edition"
---

It's been a long time since I last updates the blog. A lot of things have happened. Time to carry on.

Start taking notes from halfway in chapter 2. Notes will be in a loose format.

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
