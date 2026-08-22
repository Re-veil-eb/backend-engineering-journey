# Python `*args` and `**kwargs`

> **Learning level:** Intermediate → Senior
> **Purpose:** Understand how Python handles a variable number of positional and keyword arguments, what `*` and `**` actually do, and why they are essential for flexible functions and decorators.

---

# 1. What are `*args` and `**kwargs`?

`*args` and `**kwargs` allow a function to accept a variable number of arguments.

Example:

```python
def display(*args):
    print(args)
```

Calling:

```python
display(10, 20, 30)
```

produces:

```text
(10, 20, 30)
```

Similarly:

```python
def display(**kwargs):
    print(kwargs)
```

Calling:

```python
display(name="Chinnu", age=22)
```

produces:

```text
{
    "name": "Chinnu",
    "age": 22
}
```

The important point is:

> `*args` collects positional arguments, while `**kwargs` collects keyword arguments.

---

# 2. What Does the `*` Actually Mean?

The `*` has two related but different roles.

## During function definition

```python
def test(*args):
    ...
```

Here `*` means:

> Collect additional positional arguments into a tuple.

## During function call

```python
test(*values)
```

Here `*` means:

> Unpack an iterable into positional arguments.

So the same symbol participates in both:

```text
Packing     ← *args
Unpacking   ← *values
```

This distinction is extremely important.

---

# 3. What Does `**` Mean?

The same concept applies to `**`.

## During function definition

```python
def test(**kwargs):
    ...
```

It means:

> Collect additional keyword arguments into a dictionary.

## During function call

```python
test(**data)
```

It means:

> Unpack a mapping/dictionary into keyword arguments.

Therefore:

```text
**kwargs      → packing
**dictionary  → unpacking
```

---

# 4. `*args` — Positional Argument Collection

Consider:

```python
def add(*args):
    print(args)
```

Call:

```python
add(10, 20, 30)
```

Python collects:

```text
args = (10, 20, 30)
```

The type is:

```python
tuple
```

Therefore:

```python
def add(*args):
    print(type(args))
```

outputs:

```text
<class 'tuple'>
```

---

# 5. Why Is It Called `args`?

`args` is only a naming convention.

Python does not require the name `args`.

These are equivalent:

```python
def test(*args):
    ...
```

```python
def test(*values):
    ...
```

```python
def test(*numbers):
    ...
```

The important part is:

```text
*
```

not:

```text
args
```

Similarly, `kwargs` is a convention.

---

# 6. `**kwargs` — Keyword Argument Collection

Consider:

```python
def display(**kwargs):
    print(kwargs)
```

Calling:

```python
display(name="Chinnu", age=22)
```

produces:

```python
{
    "name": "Chinnu",
    "age": 22
}
```

The type is:

```python
dict
```

Therefore:

```python
def display(**kwargs):
    print(type(kwargs))
```

produces:

```text
<class 'dict'>
```

---

# 7. Positional vs Keyword Arguments

Consider:

```python
def user(*args, **kwargs):
    print(args)
    print(kwargs)
```

Calling:

```python
user("Chinnu", 22, city="Hyderabad", role="Engineer")
```

produces conceptually:

```text
args
↓
("Chinnu", 22)

kwargs
↓
{
    "city": "Hyderabad",
    "role": "Engineer"
}
```

So:

```text
Positional arguments
        ↓
      *args
        ↓
      tuple


Keyword arguments
        ↓
     **kwargs
        ↓
      dict
```

---

# 8. Using Normal Parameters With `*args`

You can combine regular parameters and `*args`.

```python
def test(first, *args):
    print(first)
    print(args)
```

Calling:

```python
test(10, 20, 30, 40)
```

results in:

```text
first = 10

args = (20, 30, 40)
```

The first positional argument is consumed by `first`.

The remaining positional arguments are collected into `args`.

---

# 9. Using Normal Parameters With `**kwargs`

Example:

```python
def test(name, **kwargs):
    print(name)
    print(kwargs)
```

Calling:

```python
test("Chinnu", age=22, role="Engineer")
```

results in:

```text
name = "Chinnu"

kwargs = {
    "age": 22,
    "role": "Engineer"
}
```

---

# 10. Using `*args` and `**kwargs` Together

A common generic function is:

```python
def test(*args, **kwargs):
    print(args)
    print(kwargs)
```

Calling:

```python
test(10, 20, name="Chinnu", age=22)
```

results in:

```text
args
↓
(10, 20)

kwargs
↓
{
    "name": "Chinnu",
    "age": 22
}
```

This allows the function to accept an unknown number of positional and keyword arguments.

---

# 11. `*args` Is a Tuple

Inside the function:

```python
def test(*args):
    ...
```

`args` is a tuple.

Therefore you can:

```python
args[0]
len(args)
for value in args:
    ...
```

But tuples are immutable.

So:

```python
args[0] = 100
```

raises an error.

---

# 12. `**kwargs` Is a Dictionary

Inside:

```python
def test(**kwargs):
    ...
```

`kwargs` is a dictionary.

Therefore:

```python
kwargs["name"]
kwargs.get("age")
kwargs.keys()
kwargs.values()
kwargs.items()
```

can be used.

You can also modify it:

```python
kwargs["role"] = "Developer"
```

because dictionaries are mutable.

---

# 13. Packing

Packing means collecting multiple values into one object.

Example:

```python
def test(*args):
    ...
```

Calling:

```python
test(10, 20, 30)
```

packs:

```text
10
20
30
 ↓
(10, 20, 30)
```

Therefore:

```text
*args = positional argument packing
```

Similarly:

```text
**kwargs = keyword argument packing
```

---

# 14. Unpacking

Now consider:

```python
numbers = [10, 20, 30]
```

Calling:

```python
test(*numbers)
```

unpacks the list.

Conceptually:

```text
numbers
   ↓
[10, 20, 30]
   ↓
*numbers
   ↓
test(10, 20, 30)
```

So `*` converts an iterable into positional arguments during a call.

---

# 15. Dictionary Unpacking

Consider:

```python
data = {
    "name": "Chinnu",
    "age": 22
}
```

Calling:

```python
test(**data)
```

conceptually becomes:

```python
test(name="Chinnu", age=22)
```

So:

```text
**dictionary
      ↓
keyword arguments
```

---

# 16. Packing vs Unpacking

This distinction should become automatic.

### Packing

```python
def test(*args):
    ...
```

Multiple positional arguments:

```text
10, 20, 30
```

become:

```text
(10, 20, 30)
```

### Unpacking

```python
test(*values)
```

A collection:

```text
[10, 20, 30]
```

becomes:

```text
10, 20, 30
```

Therefore:

```text
             *
        ┌────┴────┐
        ↓         ↓
     Packing   Unpacking
     *args     *values
```

---

# 17. Keyword-Only Parameters

Python also allows parameters that must be passed using keywords.

Example:

```python
def create_user(name, *, age, city):
    ...
```

Here:

```text
name → positional or keyword
age  → keyword-only
city → keyword-only
```

This is valid:

```python
create_user("Chinnu", age=22, city="Hyderabad")
```

But this is invalid:

```python
create_user("Chinnu", 22, "Hyderabad")
```

The `*` acts as a separator indicating:

> Parameters after this point must be passed by keyword.

---

# 18. Positional-Only Parameters

Python also supports positional-only parameters using `/`.

Example:

```python
def test(a, b, /, c, d):
    ...
```

Here:

```text
a, b → positional-only
c, d → positional or keyword
```

This gives Python function signatures more control over how arguments can be supplied.

---

# 19. Complete Parameter Structure

Python allows a function signature to conceptually contain:

```text
positional-only
        ↓
       /
        ↓
positional-or-keyword
        ↓
       *
        ↓
keyword-only
        ↓
*args / **kwargs
```

Example:

```python
def func(a, b, /, c, d, *args, e, f, **kwargs):
    ...
```

This is a more advanced function signature.

The important senior-level idea is:

> Python gives developers fine-grained control over how callers provide arguments.

---

# 20. Why `*args` and `**kwargs` Are Important

They are useful when the number or shape of arguments is not fixed.

Examples:

* Generic utility functions
* Wrapper functions
* Decorators
* Middleware
* Framework APIs
* Callback functions
* Adapters
* Proxy functions
* Forwarding arguments
* Flexible APIs

One of their most important uses is **decorators**.

---

# 21. `*args` and `**kwargs` in Decorators

Consider:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Why do we need them?

Because the decorator should ideally work with many different function signatures.

For example:

```python
@decorator
def add(a, b):
    return a + b
```

and:

```python
@decorator
def greet(name):
    return f"Hello {name}"
```

and:

```python
@decorator
def create_user(name, age, city=None):
    ...
```

The wrapper does not need to know every parameter.

It simply forwards them:

```text
caller
  ↓
wrapper(*args, **kwargs)
  ↓
func(*args, **kwargs)
```

This is one of the most important reasons you learned `*args` and `**kwargs` before decorators.

---

# 22. Argument Forwarding

Consider:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

There are two operations happening.

### Receiving

```python
wrapper(*args, **kwargs)
```

collects the incoming arguments.

### Forwarding

```python
func(*args, **kwargs)
```

unpacks them and sends them to the original function.

Conceptually:

```text
Caller
  │
  │ positional + keyword arguments
  ▼
wrapper
  │
  ├── *args  → tuple
  └── **kwargs → dict
  │
  ▼
unpack
  │
  ▼
original function
```

This pattern is called **argument forwarding**.

---

# 23. Why Not Write Every Parameter Manually?

Suppose the original function is:

```python
def add(a, b):
    return a + b
```

We could write:

```python
def wrapper(a, b):
    return func(a, b)
```

But this wrapper only works naturally for that particular signature.

If the original function becomes:

```python
def add(a, b, c):
    ...
```

the wrapper needs to change.

Using:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

makes the wrapper generic.

---

# 24. `*args` Does Not Mean "Any Number of Anything"

`*args` specifically collects **positional arguments**.

For:

```python
test(10, 20, name="Chinnu")
```

we get:

```text
args
↓
(10, 20)
```

and:

```text
kwargs
↓
{"name": "Chinnu"}
```

The two categories remain separate.

---

# 25. `**kwargs` Does Not Mean "Any Data"

`**kwargs` specifically collects **keyword arguments**.

For:

```python
test(name="Chinnu", age=22)
```

the values become:

```python
{
    "name": "Chinnu",
    "age": 22
}
```

The keys are the parameter names supplied by the caller.

---

# 26. Argument Binding

When a function is called, Python has to determine:

> Which argument belongs to which parameter?

For:

```python
def user(name, age):
    ...
```

and:

```python
user("Chinnu", 22)
```

Python binds:

```text
name → "Chinnu"
age  → 22
```

With:

```python
user(age=22, name="Chinnu")
```

the names determine the binding.

With `*args` and `**kwargs`, Python can collect arguments that don't correspond to explicitly declared parameters.

---

# 27. Internal Mental Model

Consider:

```python
def process(a, b=10, *args, **kwargs):
    ...
```

Call:

```python
process(1, 2, 3, 4, name="Chinnu")
```

Conceptually:

```text
a
↓
1

b
↓
2

args
↓
(3, 4)

kwargs
↓
{
    "name": "Chinnu"
}
```

The arguments are first bound according to the function signature.

Remaining positional arguments go into `args`.

Remaining keyword arguments go into `kwargs`.

---

# 28. `*` and `**` Are Operators for Unpacking/Packing

Don't mentally associate `*args` only with "special syntax".

Instead understand the underlying operations:

```text
*
├── collect positional arguments
└── unpack iterable

**
├── collect keyword arguments
└── unpack mapping
```

This mental model is much more useful.

---

# 29. Common Mistakes

## Mistake 1 — Thinking `args` is special

It isn't.

```python
def test(*args):
    ...
```

and:

```python
def test(*values):
    ...
```

work the same way.

`*` is the important part.

---

## Mistake 2 — Thinking `kwargs` is special

Again, it isn't.

```python
def test(**kwargs):
    ...
```

could be:

```python
def test(**options):
    ...
```

The `**` matters.

---

## Mistake 3 — Confusing packing and unpacking

```python
def test(*args):
```

means packing.

```python
test(*values)
```

means unpacking.

---

## Mistake 4 — Forgetting that `args` is a tuple

```python
def test(*args):
    ...
```

`args` is not a list.

It is a tuple.

---

## Mistake 5 — Forgetting that `kwargs` is a dictionary

```python
def test(**kwargs):
    ...
```

`kwargs` is a dictionary.

---

# 30. Production Perspective

In production code, `*args` and `**kwargs` should not automatically be used everywhere.

They are powerful, but overly generic APIs can make code harder to understand.

For example:

```python
def process(*args, **kwargs):
    ...
```

may hide what the function actually expects.

A senior engineer asks:

> Do I really need a flexible signature here?

Use explicit parameters when the API is known and stable.

Use `*args` and `**kwargs` when flexibility or argument forwarding is actually required.

They are particularly appropriate for:

* Decorators
* Wrappers
* Framework hooks
* Middleware
* Generic utilities
* Proxy functions
* APIs that intentionally support variable arguments

---

# 31. `*args` and `**kwargs` + Decorators

This is the key connection to your previous and upcoming learning.

A generic decorator:

```python
from functools import wraps

def decorator(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("Before")

        result = func(*args, **kwargs)

        print("After")

        return result

    return wrapper
```

The wrapper doesn't need to know whether the original function is:

```python
def add(a, b):
```

or:

```python
def greet(name):
```

or:

```python
def create_user(name, age, city=None):
```

It can receive and forward all of them.

That is the real production value of `*args` and `**kwargs`.

---

# 32. Senior-Level Mental Model

Don't remember:

> `*args` means multiple arguments.

Instead remember:

> **`*args` is a mechanism for collecting or unpacking positional arguments.**

Don't remember:

> `**kwargs` means multiple keyword arguments.

Remember:

> **`**kwargs` is a mechanism for collecting or unpacking keyword arguments.**

And remember the complete flow:

```text
                 ARGUMENTS
                     │
          ┌──────────┴──────────┐
          │                     │
      positional             keyword
          │                     │
          ▼                     ▼
        *args                **kwargs
          │                     │
          ▼                     ▼
        tuple                 dict
```

During forwarding:

```text
tuple + dict
    │
    ▼
*args + **kwargs
    │
    ▼
original function
```

---

# 33. Connection to Your Learning Path

The concepts now connect like this:

```text
Functions
   │
   ▼
Parameters & Arguments
   │
   ├── positional arguments
   │       ↓
   │     *args
   │
   └── keyword arguments
           ↓
         **kwargs
           │
           ▼
     Argument Forwarding
           │
           ▼
        Decorators
```

Later:

```text
Functions
    ↓
*args / **kwargs
    ↓
Nested Functions
    ↓
Scope / LEGB
    ↓
Closures
    ↓
Decorators
    ↓
Decorator Arguments
    ↓
Decorator Factories
    ↓
Stacked Decorators
    ↓
Class Decorators
```

---

# 34. Senior Interview Questions

Before moving forward, you should be able to answer:

1. What does `*args` actually do?
2. What type is `args`?
3. What does `**kwargs` actually do?
4. What type is `kwargs`?
5. What is the difference between packing and unpacking?
6. What does `*` mean in a function definition?
7. What does `*` mean in a function call?
8. What does `**` mean in a function definition?
9. What does `**` mean in a function call?
10. Why are `args` and `kwargs` only naming conventions?
11. Can `*args` and `**kwargs` be used together?
12. How does Python bind normal parameters before collecting remaining arguments?
13. What is argument forwarding?
14. Why are `*args` and `**kwargs` commonly used in decorators?
15. Why shouldn't every production function use `*args` and `**kwargs`?
16. What is the difference between keyword-only parameters and `**kwargs`?
17. What is the purpose of `/` in a function signature?
18. Explain this:

```python
def func(a, b, /, c, *args, d, **kwargs):
    ...
```

19. Explain the difference between:

```python
func(values)
```

and:

```python
func(*values)
```

20. Explain the difference between:

```python
func(data)
```

and:

```python
func(**data)
```

---

# 35. Final Revision

| Concept                   | Mental Model                              |
| ------------------------- | ----------------------------------------- |
| `*args`                   | Collect positional arguments              |
| `**kwargs`                | Collect keyword arguments                 |
| `*values`                 | Unpack iterable into positional arguments |
| `**data`                  | Unpack mapping into keyword arguments     |
| `args`                    | Tuple                                     |
| `kwargs`                  | Dictionary                                |
| Packing                   | Multiple arguments → one collection       |
| Unpacking                 | One collection → multiple arguments       |
| Argument forwarding       | Receive arguments and pass them onward    |
| Keyword-only parameter    | Must be supplied by name                  |
| Positional-only parameter | Must be supplied by position              |

---

# 36. Final Mental Model

```text
                  FUNCTION CALL
                       │
             ┌─────────┴─────────┐
             │                   │
       Positional             Keyword
       arguments              arguments
             │                   │
             ▼                   ▼
          *args                **kwargs
             │                   │
             ▼                   ▼
          tuple                dict
             │                   │
             └─────────┬─────────┘
                       │
                       ▼
               Argument Forwarding
                       │
                       ▼
               Generic Wrapper
                       │
                       ▼
                  DECORATORS
```

> **Key takeaway:**
> `*args` and `**kwargs` are not simply shortcuts for "many arguments." They are mechanisms for **collecting and unpacking arguments**, which makes functions, wrappers, and decorators flexible and composable.
