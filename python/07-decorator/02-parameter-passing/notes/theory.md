# `08-decorators-passing-arguments.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, `*args`, `**kwargs`, LEGB, nested functions, higher-order functions, closures, decorator fundamentals
> **Core idea:** A decorator must correctly receive and forward the arguments intended for the decorated function.

---

# 1. What is it?

When a decorator wraps a function, the **wrapper becomes the function that the caller actually invokes**.

Therefore, if the original function accepts arguments, the wrapper must be able to accept and forward those arguments.

Example:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Now it can decorate functions with different signatures:

```python
@decorator
def add(a, b):
    return a + b
```

```python
@decorator
def greet(name):
    return f"Hello {name}"
```

```python
@decorator
def create_user(name, age, city="Hyderabad"):
    return name, age, city
```

The decorator doesn't need to know each function's exact parameters.

---

# 2. Why does it exist?

Consider:

```python
@decorator
def add(a, b):
    return a + b
```

After decoration, conceptually:

```python
add = decorator(add)
```

So:

```python
add(10, 20)
```

actually calls:

```text
wrapper(10, 20)
```

If the wrapper is:

```python
def wrapper():
    return func()
```

it cannot accept:

```python
add(10, 20)
```

because `wrapper()` expects zero arguments.

Therefore we need:

```python
def wrapper(*args, **kwargs):
```

This allows the decorator to act as a **transparent layer** around the original function.

---

# 3. How does it work internally?

Consider:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

And:

```python
@decorator
def add(a, b):
    return a + b
```

Python conceptually performs:

```python
def add(a, b):
    return a + b

add = decorator(add)
```

Now:

```python
add(10, 20)
```

Execution becomes:

```text
add(10, 20)
      ↓
wrapper(10, 20)
      ↓
args = (10, 20)
kwargs = {}
      ↓
func(*args, **kwargs)
      ↓
original add(10, 20)
      ↓
30
```

The wrapper acts as an **argument forwarding layer**.

---

# 4. Core concepts

## 4.1 `*args`

`*args` captures positional arguments.

Example:

```python
def wrapper(*args):
    print(args)
```

Call:

```python
wrapper(10, 20, 30)
```

Inside:

```python
args == (10, 20, 30)
```

So:

```text
wrapper(10, 20, 30)
          ↓
args = (10, 20, 30)
```

---

## 4.2 `**kwargs`

`**kwargs` captures keyword arguments.

```python
def wrapper(**kwargs):
    print(kwargs)
```

Call:

```python
wrapper(name="Chinnu", age=22)
```

Inside:

```python
kwargs == {
    "name": "Chinnu",
    "age": 22
}
```

So:

```text
wrapper(name="Chinnu", age=22)
                    ↓
kwargs = {
    "name": "Chinnu",
    "age": 22
}
```

---

# 5. Packing and Unpacking

This is extremely important for decorators.

## Packing

When calling:

```python
wrapper(10, 20, 30)
```

Python packs the positional arguments into:

```python
args = (10, 20, 30)
```

When calling:

```python
wrapper(name="Chinnu", age=22)
```

Python packs them into:

```python
kwargs = {
    "name": "Chinnu",
    "age": 22
}
```

---

## Unpacking

Now:

```python
func(*args, **kwargs)
```

does the opposite.

If:

```python
args = (10, 20)
```

then:

```python
func(*args)
```

is equivalent to:

```python
func(10, 20)
```

If:

```python
kwargs = {
    "name": "Chinnu",
    "age": 22
}
```

then:

```python
func(**kwargs)
```

is equivalent to:

```python
func(name="Chinnu", age=22)
```

Therefore:

```text
Caller
  ↓
arguments
  ↓
wrapper
  ↓
pack into *args/**kwargs
  ↓
unpack
  ↓
original function
```

---

# 6. Execution Flow

Consider:

```python
from functools import wraps


def logger(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("Before")

        result = func(*args, **kwargs)

        print("After")

        return result

    return wrapper
```

Decorated function:

```python
@logger
def add(a, b):
    return a + b
```

Call:

```python
add(10, 20)
```

Execution:

```text
                    add(10, 20)
                         │
                         ▼
                      wrapper
                         │
              ┌──────────┴──────────┐
              │                     │
          args=(10,20)          kwargs={}
              │
              ▼
       func(*args, **kwargs)
              │
              ▼
          func(10,20)
              │
              ▼
             30
```

---

# 7. Positional Arguments

Example:

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("Calling function")
        return func(*args, **kwargs)

    return wrapper
```

Function:

```python
@logger
def add(a, b):
    return a + b
```

Call:

```python
add(10, 20)
```

Inside wrapper:

```python
args = (10, 20)
kwargs = {}
```

Then:

```python
func(*args, **kwargs)
```

becomes:

```python
func(10, 20)
```

---

# 8. Keyword Arguments

Call:

```python
add(a=10, b=20)
```

Inside wrapper:

```python
args = ()
kwargs = {
    "a": 10,
    "b": 20
}
```

Then:

```python
func(*args, **kwargs)
```

becomes:

```python
func(a=10, b=20)
```

---

# 9. Mixed Arguments

Call:

```python
add(10, b=20)
```

Inside wrapper:

```python
args = (10,)
```

and:

```python
kwargs = {
    "b": 20
}
```

Then:

```python
func(*args, **kwargs)
```

becomes:

```python
func(10, b=20)
```

The original function receives exactly what the caller intended.

---

# 10. Why Both `*args` and `**kwargs`?

If we only use:

```python
def wrapper(*args):
```

we can handle positional arguments.

But:

```python
add(a=10, b=20)
```

cannot be handled correctly.

If we only use:

```python
def wrapper(**kwargs):
```

we cannot handle:

```python
add(10, 20)
```

Therefore:

```python
def wrapper(*args, **kwargs):
```

provides a general forwarding mechanism.

---

# 11. Decorator With a Specific Signature

We could write:

```python
def decorator(func):

    def wrapper(a, b):
        return func(a, b)

    return wrapper
```

This works for:

```python
@decorator
def add(a, b):
    return a + b
```

But now the decorator is tightly coupled to:

```text
a, b
```

It cannot easily decorate:

```python
def greet(name):
    ...
```

or:

```python
def create_user(name, age, city):
    ...
```

Generic decorators therefore commonly use:

```python
*args, **kwargs
```

---

# 12. Transparent Decorator

A good general-purpose decorator should ideally preserve:

```text
Arguments
    ↓
Return value
    ↓
Exceptions
    ↓
Metadata
```

Conceptually:

```text
Original Function
       │
       │ arguments
       ▼
    Wrapper
       │
       │ same arguments
       ▼
Original Function
       │
       │ result
       ▼
    Wrapper
       │
       │ same result
       ▼
    Caller
```

This is often called a **transparent wrapper**.

---

# 13. Preserving the Return Value

Consider:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper
```

Problem:

```python
@decorator
def add(a, b):
    return a + b
```

Then:

```python
result = add(10, 20)
```

`result` becomes:

```text
None
```

because the wrapper didn't return the result.

Correct:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Now:

```python
result = add(10, 20)
```

gives:

```text
30
```

---

# 14. Passing Arguments and Closure

Look carefully:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

`wrapper()` accesses:

```python
func
```

from the enclosing scope.

So the decorator contains both:

```text
Argument forwarding
        +
Closure
```

The wrapper remembers:

```text
func
```

while receiving:

```text
args
kwargs
```

This is a direct connection to your previous closure topic.

---

# 15. Decorator With `*args/**kwargs` and `@wraps`

Production-style basic pattern:

```python
from functools import wraps


def logger(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print(f"Calling {func.__name__}")

        result = func(*args, **kwargs)

        print(f"Finished {func.__name__}")

        return result

    return wrapper
```

This gives us:

```text
                Decorator
                    │
       ┌────────────┴────────────┐
       │                         │
   Closure                 Argument forwarding
       │                         │
       ▼                         ▼
    func                  *args, **kwargs
       │                         │
       └────────────┬────────────┘
                    ▼
             Original Function
```

---

# 16. Arguments Can Be Modified

A decorator can inspect or modify arguments before passing them to the original function.

Example:

```python
def uppercase_name(func):

    def wrapper(name):
        name = name.upper()
        return func(name)

    return wrapper
```

Usage:

```python
@uppercase_name
def greet(name):
    return f"Hello {name}"
```

Call:

```python
greet("chinnu")
```

Execution:

```text
"chinnu"
    ↓
wrapper
    ↓
"CHINNU"
    ↓
original greet()
```

Result:

```text
Hello CHINNU
```

So decorators can act as an **argument transformation layer**.

---

# 17. Arguments Can Be Validated

Example:

```python
def validate_positive(func):

    def wrapper(number):
        if number <= 0:
            raise ValueError("Number must be positive")

        return func(number)

    return wrapper
```

Usage:

```python
@validate_positive
def square(number):
    return number * number
```

Now:

```python
square(5)
```

works.

But:

```python
square(-5)
```

is rejected before the original function executes.

---

# 18. Arguments Can Be Logged

Example:

```python
from functools import wraps


def log_arguments(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("args:", args)
        print("kwargs:", kwargs)

        return func(*args, **kwargs)

    return wrapper
```

Usage:

```python
@log_arguments
def create_user(name, age, city):
    return {
        "name": name,
        "age": age,
        "city": city
    }
```

Call:

```python
create_user("Chinnu", 22, city="Hyderabad")
```

The decorator can observe the call without changing the business function.

---

# 19. Decorator Can Inspect Arguments

Because:

```python
args
kwargs
```

are available inside the wrapper, a decorator can implement logic based on them.

For example:

```python
def admin_only(func):

    @wraps(func)
    def wrapper(user, *args, **kwargs):

        if user != "admin":
            raise PermissionError("Access denied")

        return func(user, *args, **kwargs)

    return wrapper
```

This is a simplified illustration of authorization behavior.

In real applications, authentication and authorization are generally more sophisticated.

---

# 20. Exception Behavior

A transparent decorator should normally allow exceptions to propagate unless it intentionally handles them.

Example:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

If:

```python
@decorator
def divide(a, b):
    return a / b
```

and:

```python
divide(10, 0)
```

the `ZeroDivisionError` propagates.

That's usually desirable.

A decorator should not silently hide errors.

---

# 21. Decorator Handling Exceptions

A decorator may intentionally handle errors.

```python
from functools import wraps


def safe_call(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        try:
            return func(*args, **kwargs)
        except ValueError as error:
            print(f"Validation failed: {error}")
            return None

    return wrapper
```

The key production principle:

> Only catch exceptions when the decorator has a clear responsibility for handling them.

Avoid:

```python
except Exception:
    pass
```

because it can hide serious failures.

---

# 22. `*args/**kwargs` Are Not Magic

They don't somehow "know" the original function's signature.

They simply collect arguments.

For:

```python
def wrapper(*args, **kwargs):
```

Python creates:

```text
args   → tuple
kwargs → dictionary
```

Then:

```python
func(*args, **kwargs)
```

unpacks them.

The original function still performs its own argument validation.

---

# 23. Signature Preservation

There is an important distinction between:

```python
@wraps(func)
```

and actually preserving the function's runtime signature.

`functools.wraps` copies useful metadata such as:

* `__name__`
* `__doc__`
* `__module__`
* `__annotations__`
* `__wrapped__`

It does **not** literally change:

```python
def wrapper(*args, **kwargs)
```

into the original function's source-level signature.

For many tools, `__wrapped__` allows introspection libraries to recover the original function.

---

# 24. `__wrapped__`

When using:

```python
@wraps(func)
```

Python sets:

```python
wrapper.__wrapped__ = func
```

Conceptually:

```text
wrapper
  │
  └── __wrapped__
          │
          ▼
       original
```

This is useful for:

* Introspection
* Testing
* Debugging
* Frameworks
* Documentation tools

It is another reason `@wraps` is more important than simply preserving the displayed function name.

---

# 25. Production Perspective

Generic decorators commonly follow this structure:

```python
from functools import wraps


def decorator(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        # Before behavior

        result = func(*args, **kwargs)

        # After behavior

        return result

    return wrapper
```

This pattern is worth recognizing immediately.

When you see:

```python
@something
def function(...):
```

and the decorator implementation contains:

```python
def wrapper(*args, **kwargs):
```

you should immediately think:

```text
The wrapper receives the caller's arguments
        ↓
packs them
        ↓
forwards them
        ↓
to the original function
```

---

# 26. Important Relationships

Your learning progression now looks like:

```text
Functions
    ↓
Functions are objects
    ↓
Functions can be passed as arguments
    ↓
Higher-Order Functions
    ↓
Nested Functions
    ↓
Closures
    ↓
Decorators
    ↓
Wrapper
    ↓
*args / **kwargs
    ↓
Generic argument forwarding
```

The concepts are building on each other.

---

# 27. Common Mistakes

## Mistake 1 — Wrapper doesn't accept arguments

Wrong:

```python
def wrapper():
    return func()
```

for:

```python
def add(a, b):
    ...
```

Correct:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

## Mistake 2 — Forgetting to forward arguments

Wrong:

```python
def wrapper(*args, **kwargs):
    return func()
```

The arguments were received but discarded.

Correct:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

## Mistake 3 — Forgetting the return value

Wrong:

```python
def wrapper(*args, **kwargs):
    func(*args, **kwargs)
```

Correct:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

## Mistake 4 — Mixing up packing and unpacking

```python
def wrapper(*args, **kwargs):
```

means:

> Pack arguments.

While:

```python
func(*args, **kwargs)
```

means:

> Unpack arguments.

---

## Mistake 5 — Using `*args` when keyword arguments are possible

This:

```python
def wrapper(*args):
```

cannot generically handle:

```python
function(name="Chinnu")
```

Use:

```python
def wrapper(*args, **kwargs):
```

for a general-purpose decorator.

---

# 28. Interview Questions

### Basic

1. Why does a decorator need `*args` and `**kwargs`?
2. What does `*args` capture?
3. What does `**kwargs` capture?
4. What is the difference between packing and unpacking?
5. How do you forward arguments from a wrapper to the original function?

### Execution

6. What happens when `add(10, 20)` is called after decoration?
7. Where are `10` and `20` stored inside the wrapper?
8. What happens when the caller uses keyword arguments?
9. What happens when positional and keyword arguments are mixed?
10. Why does `func(*args, **kwargs)` work for many different function signatures?

### Advanced

11. Why shouldn't a generic decorator define `wrapper(a, b)`?
12. What happens if a wrapper doesn't return the original function's result?
13. What is a transparent wrapper?
14. Why is `functools.wraps` used?
15. What is `__wrapped__`?
16. Can a decorator modify arguments?
17. Can a decorator modify the return value?
18. Can a decorator handle exceptions?
19. How does argument forwarding relate to closures?
20. How would you write a decorator that logs all arguments?

---

# 29. Senior-Level Mental Model

Don't think:

> "`*args` and `**kwargs` are just syntax required for decorators."

Think:

> **The wrapper becomes the new callable interface. If we want the wrapper to transparently represent arbitrary original functions, it needs a mechanism to receive and forward arbitrary positional and keyword arguments.**

The core transformation is:

```text
Caller
  │
  │ function arguments
  ▼
Wrapper
  │
  ├── *args
  │      ↓
  │   tuple
  │
  └── **kwargs
         ↓
      dictionary
  │
  ▼
Unpack
  │
  ▼
Original Function
  │
  ▼
Return Value
  │
  ▼
Wrapper
  │
  ▼
Caller
```

The senior mental model:

```text
             DECORATED FUNCTION
                     │
                     ▼
                  Wrapper
                     │
        ┌────────────┴────────────┐
        │                         │
   Receive args              Receive kwargs
        │                         │
        └────────────┬────────────┘
                     ▼
              Forward unchanged
                     │
                     ▼
             Original Function
                     │
                     ▼
                 Result
                     │
                     ▼
               Return result
```

And the complete decorator foundation is:

```text
                DECORATOR
                    │
                    ▼
          receives original function
                    │
                    ▼
             creates wrapper
                    │
             ┌──────┴──────┐
             │             │
          closure      *args/**kwargs
             │             │
             │             ▼
             │        argument forwarding
             │             │
             └──────► original function
                           │
                           ▼
                       result
                           │
                           ▼
                       caller
```

> **Key takeaway:**
> **A decorator replaces the caller's direct path to the original function with a wrapper. `*args` and `**kwargs` allow that wrapper to receive and forward arbitrary positional and keyword arguments, making the decorator reusable across functions with different signatures.**

---


