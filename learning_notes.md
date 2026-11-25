# Mayank Johri | Technical Learning Notes

Updated: 25 November 2026 (Day 3)

These notes preserve my own attempts, corrections, and recurring mistakes. Use with `learning_progress.md` and the master learning context. Answers marked 'corrected' were explained by the mentor, not necessarily independently solved.

## Day 1: References, mutability and lists (completed)

### Assignment vs mutation
```python
a = [1, 2, 3]
b = a
c = a.copy()
b.append(4)
c.append(5)
a = [10, 20]
# a: [10, 20]; b: [1, 2, 3, 4]; c: [1, 2, 3, 5]
```
My answer: Correct, though initially unsure whether rebinding `a` would also change `b`.
Lesson: `b = a` shares an object; `.copy()` creates a new outer list; reassignment rebinds one name only.

### `+=` vs `+` on lists
```python
a = [10, 20]
b = a
a += [30]       # Mutates shared list
a = a + [40]   # Creates new list and rebinds a
# a: [10, 20, 30, 40]; b: [10, 20, 30]
```
My answer: Correct through first mutation; guessed `b=[]` afterward. Correction: `b` still references original list.

### Function arguments and side effects
```python
def modify(items):
    items.append(100)
    items = [500, 600]
    items.append(700)
numbers = [10, 20]
modify(numbers)
# numbers: [10, 20, 100]
```
My mistake: Thought no return means no effect and confused scope with mutation. Correction: `append` mutates shared object even if function returns nothing; local reassignment doesn't affect caller binding.

### Non-mutating function
My attempt: `items = items + value` (incorrect when value is an integer).
```python
def add_item(items, value):
    return items + [value]
```
`append()` mutates a list and returns `None`; returning the mutated list from a function is a separate operation.

### Return identity
- `data = data + [100]`: caller's original unchanged, returned list new, `original is result` False.
- `data += [100]`: caller's list mutated, return references same list, `original is result` True.
My mistake: Initially thought both changed caller or return created a copy.

### Copy inside function
```python
def process(data):
    data.append(30)
    return data.copy()
original = [10, 20]
result = process(original)
result.append(40)
# original: [10, 20, 30]
# result: [10, 20, 30, 40]
# original is result: False
```
My mistake: Missed `.copy()` when reading and predicted both changed. Later correctly solved a simpler alias/copy exercise.

### Lists and complexity
- `append`: amortized O(1); `insert(0, x)`: O(n) shifts.
- My code used `property_ids.pop(103)` to remove value 103. Wrong: `pop` takes an index. Correct: `property_ids.remove(103)`.
- For entity attributes, I argued dictionaries are more useful than sets. Refinement: dict for key-to-attributes; set for membership-only; list for ordered sequence.

### Mutating while iterating
```python
numbers = [2, 4, 6, 8]
for n in numbers:
    numbers.remove(n)
# [4, 8], not []
```
My mistake: Assumed every element removed. Removing shifts elements left while iterator advances its index.

My safe solution: collect evens into `removal_list`, then remove each from `numbers`. Correct but repeated `remove` may be O(n^2).
Optimized version shown during review:
```python
numbers[:] = [x for x in numbers if x % 2 != 0]
```
I correctly analyzed O(n) time and O(k) temporary list space, k = number of retained odd values. Slice assignment preserves list identity.

## Day 2: Tuples, sets, dictionaries (completed), comprehensions (continued Day 3)

### Tuple with nested mutable lists
```python
a = ([1, 2], [3, 4])
b = a
c = a[0]
c.append(5)
a[0].append(6)
a = ([10, 20], [30, 40])
# b: ([1, 2, 5, 6], [3, 4]); a: ([10, 20], [30, 40])
```
My predictions: Correct. Tuple slots are immutable; referenced nested lists can mutate. `a is b` becomes False after reassignment.

Bonus:
```python
a = ([1, 2], [3, 4])
a[0] += [5]
```
My answer: It updates list. Partially correct: the list changes to `[1,2,5]` **then** a TypeError is raised trying to assign back to tuple slot. A failed operation can leave partial changes.
Final check: Correctly distinguished `a[0].append(30)` (allowed) from `a[0] = [...]` (TypeError).

### Order-preserving deduplication
My first attempt: `list(set(property_ids))`, which removes duplicates but does not guarantee first-appearance order.
My corrected independent implementation:
```python
def get_unique_properties(property_ids):
    unique_ids = []
    seen = set()
    for property_id in property_ids:
        if property_id not in seen:
            seen.add(property_id)
            unique_ids.append(property_id)
    return unique_ids
```
I correctly explained expected O(n) time and O(k) auxiliary space (k unique IDs). Set members must be hashable.

### Dictionary key equality and hashing
- Two equal tuple keys `(101, 2026)` map to one dictionary entry; later value overwrites earlier. I answered correctly.
- Tuple containing a list is unhashable. I answered correctly.
- `key1 is key2` for equal tuple literals is not a portable guarantee; equality, not identity, determines matching keys with compatible hashes.
- My incorrect prediction: `1`, `True`, `1.0` might form two keys or error. Correct: all compare equal and have hash 1, so `{1: 'Float'}` with length 1.
- Follow-up correctly answered: `0` and `False` share one entry, but string `'0'` is distinct: `{0: 'False Value', '0': 'String Zero'}`.

### Dictionary lookup choice
I correctly answered `d[key]` raises KeyError if absent and `d.get(key)` returns None. I initially said always prefer `.get()` to avoid failures. Refinement: missing-key policy is a business/data-contract decision; defaulting required data can silently hide errors. I chose to move on rather than prolong a judgment discussion.

### Dictionary comprehensions
My first attempt had `p[id]` instead of `p['id']`, attempted an `x = ...` assignment inside comprehension, and used `'low'` instead of `'Low'`.
Correct transformation:
```python
classification = {
    p['id']: 'High' if p['occupancy'] >= 90 else 'Low'
    for p in properties
}
```
My next attempt put `if` after the value but before `for` without `else`. This is invalid conditional-expression syntax.
Correct filtering:
```python
filtered = {
    p['id']: p['occupancy']
    for p in properties
    if p['occupancy'] >= 90
}
```
I correctly reasoned O(n) time, O(k) space where k matches.

Final syntax check: Correctly chose options A (filter after `for`) and C (if/else value before `for`) as valid; B (if without else before `for`) invalid. My Option A output omitted `0`; correction: `0 % 2 == 0`, so `{0: 0, 2: 4, 4: 8}`. Option C output correctly predicted `{0:0, 1:0, 2:4, 3:0, 4:8}`. Integer keys do not need quotes.
**Status: Dictionary comprehensions completed; revisit syntax and range(0) occasionally.**

## Day 3: Functions and scope

### Mutable default arguments (completed)
```python
def add_property(property_id, properties=[]):
    properties.append(property_id)
    return properties

a = add_property(101)
b = add_property(102)
c = add_property(103, [])
d = add_property(104)
# a, b, d: [101, 102, 104]; c: [103]
# a is b: True
```
My initial answer: `a` and `b` share `[101,102]`; incorrectly thought `c` and `d` share `[103,104]` because explicit `[]` replaced the default. Correction: default list is created once at function definition; passing `[]` binds a fresh list for that call only, without replacing the stored default.

Safer default pattern:
```python
def process(value, items=None):
    if items is None:
        items = []
    items.append(value)
    return items
```
My correct follow-up:
```python
a = process(10)       # [10]
b = process(20)       # [20]
shared = [100]
c = process(30, shared)
d = process(40, shared)
# c, d, shared: [100, 30, 40]; same object
# a and b: separate objects
```
Refinement: `items = []` creates a new list and binds the local name; it doesn't 'write a new address' into the old object. Supplying an existing list still permits side effects.
**Status: Completed, understanding Good.**

### Local and global scope (basic understanding; deeper scope deferred by choice)
```python
count = 10
def update():
    print(count)
    count = 20
update()
```
My prediction: Exception (correct). My reasoning: Global variable isn't visible in function (incorrect). Correction: globals can be read in functions, but assignment to `count` anywhere in the function makes it local unless declared `global`/`nonlocal`; reading before local assignment raises `UnboundLocalError`.

```python
count = 10
def update():
    if False:
        count = 20
    print(count)
update()
```
I said I didn't know. Correction: still `UnboundLocalError`; local binding classification does not depend on the branch running. `global count` would direct the function to use the global name.

```python
x = 100
def test():
    x = 200
    print(x)
test()    # 200
print(x)  # 100
```
My prediction and explanation: Correct. Local `x` is separate from global `x`.

We intentionally skipped another list mutation vs local reassignment exercise because I felt this scope detail had limited immediate value. **Do not mark LEGB/closures as completed.** Revisit when learning closures and decorators.

### `*args` and `**kwargs` (introduced, challenge NOT attempted)
- `*args` collects extra positional arguments into a tuple.
- `**kwargs` collects extra keyword arguments into a dictionary.
- Explicit signatures are usually preferable when the function contract is fixed.

**Pending exercise, do not reveal solution before my attempt:**
```python
def calculate_metrics(property_id, *args, **kwargs):
    # My implementation goes here
    pass
```
Requirements:
1. Mandatory `property_id`.
2. `args` are numeric and should be summed.
3. `kwargs` are extra property attributes.
4. Return `{'property_id': ..., 'total': ..., 'attributes': {...}}`.
Example: `calculate_metrics(101, 100, 200, 300, occupancy=95, region='Delhi')` should produce total 600 and attributes containing occupancy and region.
Also answer: What happens for `calculate_metrics(101)`? Difference between positional numeric arguments and keyword attributes?

**Status: Not started. Stop point for next session.**

## Mistake-prevention checklist

1. Track object aliases; distinguish mutation from rebinding.
2. Check for `.copy()` and whether a function mutates inputs.
3. Remember list `+=` mutates but list `+` creates a new list.
4. Distinguish `pop(index)` and `remove(value)`.
5. Avoid deleting from a list while directly iterating over it.
6. Use list + set when deduplicating with order preservation.
7. Remember cross-type numeric dictionary keys can compare equal.
8. In comprehensions, filtering `if` follows `for`; conditional values require `if ... else ...` before `for`.
9. Remember `range(n)` includes zero and excludes n.
10. Mutable defaults persist across omitted-argument calls; explicit arguments do not overwrite stored defaults.
11. Reading a local variable before binding raises `UnboundLocalError`, even if assignment is in unreachable code.

## Latest master handover

Date: 25 November 2026
Phase: 1 - Python Fundamentals
Topic: Functions: `*args` and `**kwargs`
Concepts covered: Mutable defaults (completed), local/global scope basics (covered; advanced deferred), introduction to variable positional and keyword arguments
Problem assigned: Implement `calculate_metrics(property_id, *args, **kwargs)` and answer two conceptual questions
Problem status: Not started
My understanding: Good on mutable defaults; basic local/global understood; `*args`/`**kwargs` not yet assessed
Next topic: Complete `*args` and `**kwargs` exercise, then proceed to closures and decorators at a suitable pace
