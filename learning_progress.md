# Technical Learning Progress Handover

**Learner:** Mayank Johri  
**Last session:** 25 November 2026 (Day 3)  
**Phase:** 1 - Python Fundamentals  
**Weekly target:** 6-7 hours, usually weekdays around 4:00-5:30 PM IST  
**Career direction:** Senior backend/data platform engineering; ₹45 LPA+ long-term target

## Current position

**Current topic:** Functions: `*args` and `**kwargs`  
**Status:** Introduced, exercise **not started**. Session stopped at user's request.  
**Next action:** Ask Mayank to implement `calculate_metrics()` and answer the two conceptual questions below. Do not reveal the implementation first.

## Completed

| Topic | Assessment | Notes |
|---|---|---|
| Variables, references and mutability | Good | Early confusion about side effects, aliasing and `.copy()` resolved through practice |
| Lists and iteration | Good | `pop` vs `remove`, iteration mutation, slice assignment, complexity |
| Tuples and nested mutability | Good | Correctly reasoned about nested lists; learned tuple `+=` partial mutation/exception |
| Sets and ordered deduplication | Good | Independently implemented set + list; O(n) expected time, O(k) space |
| Dictionary hashability and lookup basics | Good | Learned cross-type numeric key equality; missing-key policy is context-dependent |
| Dictionary comprehensions | Good, syntax to revisit | Final valid/invalid syntax exercise solved; initially omitted zero in `range(5)` |
| Mutable default function arguments | Good | Follow-up with `None` defaults and shared list solved correctly |

## Partially covered / deferred

- **Local/global scope:** Basic local vs global example solved correctly. `UnboundLocalError` from a read before local assignment explained, including unreachable assignment. More scope and mutation exercises deferred at Mayank's request; LEGB and closures **not completed**.
- **Function arguments:** `*args` and `**kwargs` introduced but no challenge attempt yet.

## Pending challenge (resume here)

```python
def calculate_metrics(property_id, *args, **kwargs):
    # Implement this yourself
    pass
```

Requirements:
1. `property_id` is mandatory.
2. `*args` contains numeric values; calculate their sum.
3. `**kwargs` contains additional property attributes.
4. Return a dictionary with keys `property_id`, `total`, and `attributes`.

Example call:
```python
calculate_metrics(101, 100, 200, 300, occupancy=95, region="Delhi")
```

Expected result:
```python
{
    "property_id": 101,
    "total": 600,
    "attributes": {"occupancy": 95, "region": "Delhi"}
}
```

Questions:
1. What happens when calling `calculate_metrics(101)`?
2. What differs between `calculate_metrics(101, 100, 200)` and `calculate_metrics(101, occupancy=100, units=200)`?

**Problem status:** Not started. **Understanding:** Not yet assessed.

## Significant mistakes worth revisiting

- Assumed no return means no mutation can persist; corrected.
- Missed `.copy()` in code; read operations carefully.
- Used `pop(value)` instead of `remove(value)`.
- Assumed list deletion during iteration removes every element.
- Initially used `list(set(...))` despite ordering requirement.
- Didn't know `True`, `1`, and `1.0` are equivalent dictionary keys.
- Mixed conditional value expressions with comprehension filters.
- Initially thought explicit `[]` replaces a function's stored mutable default.
- Initially believed globals aren't visible in functions; learned local assignment classification.

Detailed attempts, corrected examples, and explanations: **`learning_notes.md`**.

## Instructions for the next AI mentor

1. Continue directly with the pending `*args`/`**kwargs` challenge.
2. Let Mayank attempt code before showing the solution.
3. Review correctness, edge cases, readability and complexity as appropriate.
4. Teach one concept at a time, not a long lecture.
5. Do not reopen settled judgment calls or repeat basic scope exercises unless they become relevant to closures/decorators.
6. Occasionally use brief revision exercises from `learning_notes.md`.
7. At the end, update both notes and this handover. Do not mark unfinished work complete.

## Master tracking record

**Date:** 25 November 2026  
**Phase:** 1 - Python Fundamentals  
**Topic:** Functions: `*args` and `**kwargs`  
**Concepts covered:** Mutable default arguments and their persistence; `None` default pattern; basic local/global scope; introduction to `*args` and `**kwargs`  
**Problem assigned:** `calculate_metrics()` implementation and conceptual questions  
**Problem status:** Not started  
**My understanding:** Good on mutable defaults; basic scope understood; `*args`/`**kwargs` not assessed  
**Next topic:** Finish `*args`/`**kwargs`, then closures/decorators when ready
