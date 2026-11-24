# Mayank Johri's Python Learning Notes

**Updated:** 23 November 2026  
**Purpose:** Preserve my actual attempts, mistakes, corrections, and engineering lessons for future revision.  
**Status:** Phase 1, Topics 1 and 2 reviewed; next topic is Tuples and Immutability.

> These notes distinguish what I predicted from what Python actually does. Some snippets are reconstructed from the conversation, and wording is lightly cleaned up without changing the substance of my answers.

## Topic 1: Variables, references, mutability

### 1. Assignment aliases a list; `.copy()` creates another list

```python
a = [1, 2, 3]
b = a
c = a.copy()
b.append(4)
c.append(5)
a = [10, 20]
print(a, b, c)
```

**My answer:** `a=[10,20]`, `b=[1,2,3,4]`, `c=[1,2,3,5]`. I briefly wondered whether changing `a` would make `b` become `[10,20]`.

**Correct:** Exactly those three lists. `b = a` binds both names to the same list; `a = [...]` later rebinds only `a`. `c` is a separate outer list.

**Remember:** Reassignment changes a name's binding; mutation changes the object. A shallow copy does not recursively copy nested mutable objects.

### 2. Integers: reassignment is not mutation

```python
x = 100
y = x
y = 200
print(x, y)  # 100 200
```

**My answer:** `x=100`, `y=200` because integers are immutable. **Correct.** More precisely, `y` is rebound to another integer object.

### 3. `+=` versus `+` on lists

```python
a = [10, 20]
b = a
a += [30]
print(a, b)
a = a + [40]
print(a, b)
```

**My answer:** First outputs both `[10,20,30]`; second `a=[10,20,30,40]`, but I guessed `b=[]` and questioned whether `+` works on lists.

**Correct:**
```text
[10, 20, 30] [10, 20, 30]
[10, 20, 30, 40] [10, 20, 30]
```

**Why:** For lists, `+=` extends the existing object; `a + [40]` constructs a new list and `a = ...` rebinds `a`. List `+` is valid when concatenating lists.

### 4. Function arguments can mutate objects without returning them

```python
def modify(items):
    items.append(100)
    items = [500, 600]
    items.append(700)

numbers = [10, 20]
modify(numbers)
print(numbers)
```

**My answer:** `[10,20]`, reasoning that no object was returned and the variable was not in scope. **Incorrect.**

**Correct:** `[10,20,100]`.

**Why:** `items` initially refers to the same list as `numbers`; `append(100)` mutates it. `items = [500,600]` only rebinds the local name. `append(700)` affects that new list, not `numbers`.

**Related answer:** If the rebinding line is removed, I predicted `[10,20,100,700]`, which is **correct**, but my scope explanation was wrong.

**Remember:** A return value is not required for side effects. Scope determines where names are bound, not whether a shared mutable object can change.

### 5. Writing a non-mutating `add_item`

**My attempted solution:**
```python
def add_item(items, value):
    items = items + value
    return items
```

**My reasoning:** `+` creates a new list. **Concept correct, syntax/type usage incorrect** when `value` is an integer.

**Correct:**
```python
def add_item(items, value):
    return items + [value]

original = [10, 20]
result = add_item(original, 30)
assert original == [10, 20]
assert result == [10, 20, 30]
assert original is not result
```

**Why:** `[10,20] + 30` raises `TypeError`; list concatenation needs another list, hence `[value]`.

**My statement about `append`:** I said it would return the original list with the added value. **Refinement:** `items.append(value)` mutates the list and itself returns `None`. A function can separately `return items` to return the same mutated object.

### 6. Returning a new list versus a mutated list

**Case A:**
```python
def update(data):
    data = data + [100]
    return data
original = [10, 20]
result = update(original)
print(original, result, original is result)
```

**My answer:** Both lists `[10,20,100]`; identity `False`. **Partially incorrect.**

**Correct:** `[10,20]`, `[10,20,100]`, `False`.

**Case B:**
```python
def update(data):
    data += [100]
    return data
original = [10, 20]
result = update(original)
print(original, result, original is result)
```

**My answer:** `original=[10,20,100]`, but I thought `result` might be a new list. **Partially incorrect.**

**Correct:** `[10,20,100]`, `[10,20,100]`, `True`.

**Remember:** Returning an object does not copy it. `is` tests identity, while `==` tests value equality.

### 7. Mutation followed by `.copy()`

```python
def process(data):
    data.append(30)
    return data.copy()
original = [10, 20]
result = process(original)
result.append(40)
print(original, result, original is result)
```

**My answer:** `original=[10,20,30,40]`, `result=[10,20,3,40]`, identity `True`. **Incorrect because I missed the `.copy()` call when reading the code.**

**Correct:** `[10,20,30]`, `[10,20,30,40]`, `False`.

**Lesson:** Read each operation carefully before reasoning. Specifically mark calls to `.copy()`, `append()`, `+=`, and assignments.

### 8. Final reference check: solved independently

```python
a = [10, 20]
b = a
c = a.copy()
b.append(30)
c.append(40)
```

**My correct answer:** `a=[10,20,30]`, `b=[10,20,30]`, `c=[10,20,40]`. `a is b` is `True`; `a is c` is `False`.

**Topic assessment:** Good. Revisit nested mutable objects and function side effects later.

---

## Topic 2: Python lists and performance

### 9. `pop(index)` versus `remove(value)`

**Task:** Add 106, remove property ID 103, insert 100 at the beginning.

**My attempted code:**
```python
property_ids.append(106)
property_ids.pop(103)
property_ids.insert(0, 100)
```

**Mistake:** `pop(103)` means remove at *index* 103, not the element with *value* 103; this raises `IndexError` for the short list.

**Correct:**
```python
property_ids = [101, 102, 103, 104, 105]
property_ids.append(106)
property_ids.remove(103)
property_ids.insert(0, 100)
assert property_ids == [100, 101, 102, 104, 105, 106]
```

**Remember:** `pop(index)` removes by position and returns the value; `remove(value)` removes the first matching value.

### 10. List performance and choice of collection

**My correct reasoning:** `append` is generally faster than `insert(0, x)` because front insertion shifts elements. Appending is amortized O(1); front insertion O(n).

**My proposed lookup structure:** A dictionary keyed by property ID, because real services usually retrieve attributes, not just check existence. **Good engineering judgment when attributes are required.**

**Refinement:** A `set` is clearer for membership-only or deduplication use cases; dictionary for key-to-attributes mapping; list for ordered sequences. Hash-based lookups are average O(1), not guaranteed worst-case O(1).

### 11. Removing while iterating skips elements

```python
numbers = [2, 4, 6, 8]
for n in numbers:
    numbers.remove(n)
print(numbers)
```

**My answer:** Empty list. **Incorrect.**

**Correct:** `[4,8]`.

**Why:** After removing `2`, the list becomes `[4,6,8]`, but the iterator advances to index 1, now holding `6`. It removes `6`, leaving `[4,8]`.

**Remember:** Do not change a list's length while iterating directly over it unless the algorithm is explicitly designed to handle shifting indices.

### 12. My safe-removal design: correct but inefficient

**My proposed solution:** First collect even values in a `removal_list`, then loop over that separate list and call `numbers.remove(val)`.

**Valid implementation:**
```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8]
removal_list = []
for n in numbers:
    if n % 2 == 0:
        removal_list.append(n)
for val in removal_list:
    numbers.remove(val)
assert numbers == [1, 3, 5, 7]
```

**Assessment:** Correct, preserves the original list object, and handles duplicates. But repeated `remove` searches and shifts elements, so worst-case time O(n^2), auxiliary space O(n).

**My wording mistake:** I initially said to create a list of only *even* numbers and return it. The desired output should retain only *odd* numbers.

### 13. Efficient in-place filtering using slice assignment

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8]
alias = numbers
numbers[:] = [x for x in numbers if x % 2 != 0]
assert numbers == [1, 3, 5, 7]
assert alias is numbers
```

**My complexity analysis:** O(n) time; O(number of odd values) temporary list space. **Correct.** Let `k` be the number of odd elements: O(k) auxiliary list storage, worst case O(n).

**Important distinction:** `numbers = new_list` rebinds the name and leaves any alias referring to the old list; `numbers[:] = new_list` replaces the existing list's contents and preserves its identity.

**Topic assessment:** Good. The slice-assignment implementation was demonstrated during mentoring; my independent contribution was the correct complexity analysis.

---

## Quick mistake-prevention checklist

Before predicting Python code, ask:

1. **Which names refer to the same object?** Track aliases (`b = a`).
2. **Is this mutation or rebinding?** `append`, `remove`, list `+=`, and slice assignment mutate; `a = ...` rebinds.
3. **Was a copy created?** Look for `.copy()`, slicing, or `+` concatenation.
4. **Does the function mutate its argument?** A `return` is not necessary for a side effect.
5. **Does a method accept an index or a value?** `pop(index)` vs `remove(value)`.
6. **Am I changing the list during iteration?** If yes, consider skipped elements.
7. **What is the actual time and auxiliary space cost?** Include searching, shifting, and temporary collections.
8. **Do I need membership, attributes, or ordering?** Set, dictionary, or list respectively.

## Spaced-revision prompts (try before looking up answers)

1. Predict `a`, `b`, and `a is b` after `b=a; a=a+[1]` versus `b=a; a += [1]`.
2. Write a function that returns a new list with one added element without changing its input.
3. Explain why a function can mutate a caller's list even if it returns `None`.
4. Explain `pop(2)` versus `remove(2)` for `[2, 5, 2, 7]`.
5. Predict what happens when removing elements from a list inside a `for` loop over that same list.
6. Filter a list in O(n) time while preserving its identity for aliases.
7. State when to use a dictionary instead of a set for property IDs.

## Latest handover

**Date:** 23 November 2026  
**Phase:** 1, Python Fundamentals  
**Topic:** Python Lists (completed)  
**Concepts covered:** References, mutation, copying, function arguments, list operations, iteration, slice assignment, basic complexity  
**Problem assigned:** Safe removal of even numbers; optimized slice assignment  
**Problem status:** Solved conceptually; optimized approach analyzed  
**My understanding:** Good, with earlier mistakes recorded above for revision  
**Next topic:** Tuples and Immutability (not started)

**Instruction to future mentor:** Continue with tuples. Revisit the recorded misconceptions occasionally through short retrieval exercises, not a full restart. Let Mayank attempt code before revealing solutions.

---

# Day 2 Addendum: Tuples, Sets, Dictionaries and Comprehensions

**Date:** 24 November 2026  
**Status:** Tuples and Sets completed; dictionary fundamentals covered; dictionary comprehension filtering exercise pending.

## 14. Tuple containing mutable lists

```python
a = ([1, 2], [3, 4])
b = a
c = a[0]
c.append(5)
print(a)
a[0].append(6)
print(b)
a = ([10, 20], [30, 40])
print(b)
print(a)
```

**My predictions (all correct):**
```text
([1, 2, 5], [3, 4])
([1, 2, 5, 6], [3, 4])
([1, 2, 5, 6], [3, 4])
([10, 20], [30, 40])
```

**Why:** Tuple slots cannot be reassigned, but the lists stored in those slots can be mutated. `b` remains bound to the old tuple after `a` is rebound. Thus `a is b` is `False` at the end.

**Bonus:**
```python
a = ([1, 2], [3, 4])
a[0] += [5]
```

**My answer:** It updates the list. **Partially correct:** It mutates the list to `[1, 2, 5]` **and then raises `TypeError`** because augmented assignment attempts to store the result back into the immutable tuple slot. A failed statement can leave partial state changes.

**Final check (correct):** `a[0].append(30)` succeeds, whereas `a[0] = [10,20,30]` raises `TypeError`. My explanation was that the first mutates the nested list and the second attempts assignment. **Refinement:** Assignment does not necessarily create a new object; it attempts to rebind a slot or name.

## 15. Set deduplication must preserve order when required

**Original attempt:**
```python
property_set = set(property_ids)
return list(property_set)
```

**What was right:** Removes duplicates. **What was wrong:** Set iteration order is not guaranteed, so the original first-appearance order is not reliably preserved.

**My corrected solution (independently written):**
```python
def get_unique_properties(property_ids):
    unique_ids = []
    seen = set()
    for property_id in property_ids:
        if property_id not in seen:
            seen.add(property_id)
            unique_ids.append(property_id)
    return unique_ids

property_ids = [101, 102, 103, 101, 104, 102, 105]
print(get_unique_properties(property_ids))
# [101, 102, 103, 104, 105]
```

**My complexity explanation (correct):** Set membership is O(1) on average, so the loop is expected O(n), not O(n²). Both `seen` and `unique_ids` hold up to `k` unique IDs, giving O(k) auxiliary space, worst case O(n). For one million input IDs and 100 unique IDs, `k = 100`.

**Edge case:** Elements must be hashable. A list of lists causes `TypeError` when checking membership in a set.

## 16. Dictionary equality and hashability

```python
data = {}
key1 = (101, 2026)
key2 = (101, 2026)
data[key1] = "Property A"
data[key2] = "Property B"
```

**My answers:** `len(data) == 1`, and `data[key1] == 'Property B'`, because equivalent keys update the value. **Correct.** A tuple containing a list cannot be a dictionary key because the nested list is unhashable. **Correct.**

**Important nuance:** `key1 == key2` is `True`; `key1 is key2` should not be relied on in this literal example because constant reuse is implementation-dependent. Dictionaries use hash and equality, not identity alone.

## 17. Numeric and Boolean keys can be equivalent

```python
data = {}
data[1] = "Integer"
data[True] = "Boolean"
data[1.0] = "Float"
```

**My prediction:** Two keys, maybe an error because I wasn't sure if `bool` was hashable. **Incorrect.**

**Correct:** `data == {1: 'Float'}`, `len(data) == 1`, and `data[True] == 'Float'`.

**Why:** `True == 1 == 1.0` and all three have equal hashes. Boolean values are hashable. Updating an equivalent key changes the value while retaining the original stored key.

**Follow-up (correct independently):**
```python
data = {}
data[0] = "Zero"
data[False] = "False Value"
data["0"] = "String Zero"
# {0: 'False Value', '0': 'String Zero'}
# len(data) == 2
# data[False] == 'False Value'
```

**My reasoning:** Quoted `"0"` is a string and is not equal to integer `0` or boolean `False`. **Correct.** Distinct types can still compare equal, so type alone does not determine key uniqueness.

## 18. Dictionary indexing versus `.get()`

**My answers:** `properties[103]` raises `KeyError`; `properties.get(103)` returns `None`. **Correct.**

**My original design preference:** Always use `.get()` with a default to avoid errors. **Refinement:** This is a requirements-driven choice, not a universal rule. Use a default when missing data is expected and a fallback is valid; fail or validate explicitly when a missing required key indicates bad data. I chose not to spend more time on this judgment discussion.

## 19. Dictionary comprehension syntax: current learning gap

**Task:** Map each property ID to `"High"` if occupancy >= 90, otherwise `"Low"`.

**My attempted expression (formatting normalized):**
```python
dicty = {
    p[id]: x for p in properties x = "High" if p[occupancy] >= 90 else "low"
}
```

**My idea was right:** Iterate properties, use ID as key, classify occupancy. **Syntax was incorrect:**
- Dictionary string keys require `p["id"]` and `p["occupancy"]`.
- The value expression belongs before `for`, not an `x = ...` assignment after it.
- Required spelling was `"Low"`, not `"low"`.

**Correct solution shown during review:**
```python
dicty = {
    p["id"]: "High" if p["occupancy"] >= 90 else "Low"
    for p in properties
}
```

**Critical distinction:**
- **Transform all records with conditional values:** `{key: A if condition else B for item in items}`.
- **Filter out records:** `{key: value for item in items if condition}`.

**Pending challenge, NOT solved yet:**
```python
properties = [
    {"id": 101, "occupancy": 95},
    {"id": 102, "occupancy": 82},
    {"id": 103, "occupancy": 91},
    {"id": 104, "occupancy": 76},
]
```

Write a dictionary comprehension producing `{101: 95, 103: 91}` and explain time complexity and auxiliary space complexity. **Do not reveal the solution before my attempt in the next session.**

## Day 2 lessons to revisit

1. A tuple can hold mutable lists. Mutating the list is different from replacing the tuple slot.
2. `a[0] += [...]` on a tuple-held list can mutate and then fail.
3. `list(set(values))` deduplicates but does not guarantee original order.
4. For order-preserving deduplication, use a `seen` set plus an output list.
5. Dictionary key equivalence is based on hashing and equality, not simply type or identity.
6. `True`, `1`, and `1.0` are equivalent dictionary keys; `"0"` and `0` are not.
7. Comprehension value expressions and filter conditions have different positions.
8. Check exact syntax, string capitalization, and quoted dictionary keys.

## Day 2 master handover

**Date:** 24 November 2026  
**Phase:** 1, Python Fundamentals  
**Topic:** Dictionary Comprehensions (in progress)  
**Concepts covered:** Tuples, nested mutability, sets, ordered deduplication, dictionary hashability and key equality, dictionary comprehension syntax  
**Problem assigned:** Filter properties with occupancy >= 90 into an ID-to-occupancy dictionary  
**Problem status:** Not started (follow-up); classification comprehension attempted and corrected during review  
**My understanding:** Good on tuples, sets, and dictionary fundamentals; needs syntax practice on comprehensions  
**Next topic:** Finish dictionary comprehensions, then Functions, Arguments and Scope  
**Mentor instruction:** Continue the pending exercise, review my own code before showing a solution, and occasionally revisit these documented mistakes.
