# `05-class-as-decorator.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, first-class functions, closures, decorators, `*args/**kwargs`, decorator factories
> **Core idea:** A class can be used as a decorator because **instances of classes can be made callable using `__call__()`**.

---

# 1. What is it?

Normally, decorators are implemented using functions:

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("Before")
        result = func(*args, **kwargs)
        print("After")
        return result

    return wrapper
```

But Python also allows a **class to act as a decorator**.

Example:

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("Before")

        result = self.func(*args, **kwargs)

        print("After")

        return result
```

Usage:

```python
@Logger
def greet(name):
    print(f"Hello {name}")
```

This works because:

```python
@Logger
```

is equivalent to:

```python
greet = Logger(greet)
```

Now `greet` refers to a **Logger object**.

But how can we do:

```python
greet("Chinnu")
```

on an object?

Because `Logger` defines:

```python
def __call__(self, ...):
```

Therefore:

> **A class can be used as a decorator when its instances are callable, typically through `__call__()`.**

---

# 2. Why does it exist?

Function decorators are excellent when the decorator behavior is simple.

But sometimes the decorator needs to maintain **state**.

For example:

```text
Number of calls
Configuration
Cached values
Retry count
Metrics
Statistics
Timing information
Rate-limit state
```

A class is naturally suited for maintaining state.

Example:

```python
class CallCounter:

    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)
```

Usage:

```python
@CallCounter
def process():
    print("Processing")
```

Now:

```python
process()
process()
process()
```

The object maintains:

```text
count = 3
```

This is one of the strongest reasons to use a class as a decorator.

---

# 3. How Does It Work Internally?

Consider:

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("Before")

        result = self.func(*args, **kwargs)

        print("After")

        return result
```

Then:

```python
@Logger
def greet():
    print("Hello")
```

Python transforms it into:

```python
def greet():
    print("Hello")

greet = Logger(greet)
```

So:

```text
Original function
      ↓
Logger(function)
      ↓
Logger object
      ↓
greet points to Logger object
```

When we execute:

```python
greet()
```

Python sees that `greet` is an object.

Because the object has:

```python
__call__()
```

Python effectively performs:

```python
greet.__call__()
```

Therefore:

```text
greet()
   ↓
Logger instance
   ↓
__call__()
   ↓
self.func()
   ↓
Original function
```

---

# 4. Core Concepts

There are four important concepts.

```text
Class
  +
Object
  +
__call__()
  +
Decorator Protocol
  =
Class-Based Decorator
```

### 1. `__init__`

Receives the function being decorated.

```python
def __init__(self, func):
    self.func = func
```

### 2. `self.func`

Stores the original function.

```text
self.func
    ↓
Original Function
```

### 3. `__call__`

Makes the instance behave like a function.

```python
def __call__(self, *args, **kwargs):
```

### 4. Decorator Syntax

```python
@Logger
def greet():
    ...
```

becomes:

```python
greet = Logger(greet)
```

---

# 5. Execution Flow

Consider:

```python
class Logger:

    def __init__(self, func):
        print("Creating decorator object")
        self.func = func

    def __call__(self):
        print("Before")
        self.func()
        print("After")
```

Then:

```python
@Logger
def greet():
    print("Hello")
```

## Decoration Time

Python executes:

```python
greet = Logger(greet)
```

So:

```text
Original greet function
        ↓
Logger(greet)
        ↓
Logger object created
        ↓
greet points to Logger object
```

The `__init__()` method runs **at decoration time**.

---

## Call Time

Later:

```python
greet()
```

Since `greet` is now a Logger object:

```text
greet()
   ↓
Logger.__call__()
   ↓
self.func()
   ↓
Original greet()
```

---

# 6. Decoration Time vs Call Time

This distinction is extremely important.

## Decoration Time

```python
@Logger
def greet():
    ...
```

becomes:

```python
greet = Logger(greet)
```

This invokes:

```python
__init__()
```

---

## Call Time

When:

```python
greet()
```

is executed:

```python
__call__()
```

runs.

Therefore:

```text
Decoration Time
      ↓
__init__()

Call Time
      ↓
__call__()
```

Senior engineers must keep these two phases separate.

---

# 7. Complete Example

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):

        print("Function execution started")

        result = self.func(*args, **kwargs)

        print("Function execution completed")

        return result
```

Usage:

```python
@Logger
def add(a, b):
    return a + b
```

Call:

```python
result = add(10, 20)
```

Flow:

```text
add(10, 20)
      ↓
Logger object
      ↓
__call__(10, 20)
      ↓
self.func(10, 20)
      ↓
Original add()
      ↓
30
      ↓
return 30
```

---

# 8. Why `__call__()` Is Important

Normally:

```python
obj.method()
```

calls a method.

But if a class defines:

```python
def __call__(self):
```

we can do:

```python
obj()
```

Example:

```python
class Greeter:

    def __call__(self):
        print("Hello")
```

Create object:

```python
g = Greeter()
```

Now:

```python
g()
```

Output:

```text
Hello
```

Python treats:

```python
g()
```

conceptually as:

```python
g.__call__()
```

Therefore:

> `__call__()` makes an object callable.

This concept is bigger than decorators.

---

# 9. Class as Decorator vs Function as Decorator

## Function Decorator

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("Before")
        return func(*args, **kwargs)

    return wrapper
```

Structure:

```text
Function
   ↓
wrapper function
   ↓
Original function
```

---

## Class Decorator

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("Before")
        return self.func(*args, **kwargs)
```

Structure:

```text
Function
   ↓
Logger object
   ↓
__call__()
   ↓
Original function
```

---

# 10. The Big Difference

A function decorator usually stores state through:

```text
Closure
```

A class decorator usually stores state through:

```text
Instance Attributes
```

### Function-based

```python
def counter(func):

    count = 0

    def wrapper(*args, **kwargs):
        nonlocal count
        count += 1
        return func(*args, **kwargs)

    return wrapper
```

State:

```text
Closure
 ↓
count
```

### Class-based

```python
class Counter:

    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        self.count += 1
        return self.func(*args, **kwargs)
```

State:

```text
Object
 ↓
self.count
```

This is an important connection to your previous topic on closures.

---

# 11. Stateful Class Decorator

Example:

```python
class CallCounter:

    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):

        self.count += 1

        print(f"Call number: {self.count}")

        return self.func(*args, **kwargs)
```

Usage:

```python
@CallCounter
def process():
    print("Processing")
```

Calls:

```python
process()
process()
process()
```

Output:

```text
Call number: 1
Processing

Call number: 2
Processing

Call number: 3
Processing
```

The object maintains state.

---

# 12. Multiple Decorated Functions

Consider:

```python
@CallCounter
def function_a():
    pass


@CallCounter
def function_b():
    pass
```

Python creates two separate objects:

```text
function_a
    ↓
CallCounter object A
    ↓
count = 0


function_b
    ↓
CallCounter object B
    ↓
count = 0
```

After:

```python
function_a()
function_a()
function_b()
```

state becomes:

```text
Object A
count = 2

Object B
count = 1
```

This is instance-level state.

---

# 13. Passing Arguments Through the Decorator

A production-quality class decorator should normally support arbitrary function arguments.

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):

        print(f"Calling {self.func.__name__}")

        result = self.func(*args, **kwargs)

        print(f"Finished {self.func.__name__}")

        return result
```

Usage:

```python
@Logger
def create_user(name, age):
    return {
        "name": name,
        "age": age
    }
```

Call:

```python
user = create_user("Chinnu", 22)
```

Arguments flow:

```text
"Chinnu", 22
      ↓
__call__(*args, **kwargs)
      ↓
self.func(*args, **kwargs)
      ↓
create_user()
```

---

# 14. Returning Values

Just like function-based decorators, class-based decorators must preserve return values.

Correct:

```python
class Logger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        result = self.func(*args, **kwargs)
        return result
```

If we write:

```python
self.func(*args, **kwargs)
```

without:

```python
return
```

the caller may receive:

```text
None
```

---

# 15. Exception Handling

Class decorators can also intercept exceptions.

```python
class ErrorLogger:

    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):

        try:
            return self.func(*args, **kwargs)

        except Exception as exc:
            print(f"Error: {exc}")
            raise
```

Important:

```python
raise
```

re-raises the original exception.

This is useful when you want to:

```text
Log exception
      ↓
Preserve original failure
```

rather than silently hiding the problem.

---

# 16. Class Decorator With Configuration

Now we reach an important complication.

This:

```python
@Logger
def process():
    ...
```

means:

```python
process = Logger(process)
```

But:

```python
@Logger("INFO")
def process():
    ...
```

means:

```python
process = Logger("INFO")(process)
```

Now `Logger("INFO")` is expected to return something callable that can receive the function.

A class can support this pattern too.

Example:

```python
class Logger:

    def __init__(self, level):
        self.level = level

    def __call__(self, func):

        def wrapper(*args, **kwargs):

            print(f"[{self.level}] Calling {func.__name__}")

            return func(*args, **kwargs)

        return wrapper
```

Usage:

```python
@Logger("INFO")
def process():
    print("Processing")
```

Transformation:

```python
process = Logger("INFO")(process)
```

Here:

```text
Logger("INFO")
      ↓
Creates object
      ↓
object(process)
      ↓
__call__(process)
      ↓
returns wrapper
```

Notice the difference from the previous class decorator.

---

# 17. Two Different Class Decorator Patterns

This distinction is important.

## Pattern 1 — Class Directly Decorates Function

```python
@Logger
def process():
    ...
```

Equivalent to:

```python
process = Logger(process)
```

Therefore:

```python
__init__(func)
```

receives the function.

And:

```python
__call__(*args, **kwargs)
```

handles runtime calls.

Structure:

```text
Logger(function)
      ↓
Object
      ↓
object(...)
      ↓
__call__(*args, **kwargs)
```

---

## Pattern 2 — Configured Class Decorator

```python
@Logger("INFO")
def process():
    ...
```

Equivalent to:

```python
process = Logger("INFO")(process)
```

Therefore:

```python
__init__("INFO")
```

receives configuration.

Then:

```python
__call__(process)
```

receives the function.

Structure:

```text
Logger("INFO")
      ↓
Configured Object
      ↓
object(function)
      ↓
__call__(function)
      ↓
Wrapper
```

This is an important bridge to your previous topic on decorator factories.

---

# 18. Function Factory vs Class Decorator

You have now learned two ways to create configurable behavior.

### Function-based

```python
def retry(attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):
            ...

        return wrapper

    return decorator
```

### Class-based

```python
class Retry:

    def __init__(self, attempts):
        self.attempts = attempts

    def __call__(self, func):

        def wrapper(*args, **kwargs):
            ...

        return wrapper
```

Both can achieve similar behavior.

The class gives you explicit object state:

```text
self.attempts
self.count
self.cache
self.metrics
```

---

# 19. Production Perspective

Class-based decorators are particularly useful when decorator state becomes complex.

Examples:

```text
Request metrics
Retry tracking
Caching state
Rate limiting
Circuit breaker
Performance counters
Statistics
Configuration
Resource management
```

For example:

```python
class Metrics:

    def __init__(self, func):
        self.func = func
        self.calls = 0
        self.failures = 0

    def __call__(self, *args, **kwargs):

        self.calls += 1

        try:
            return self.func(*args, **kwargs)

        except Exception:
            self.failures += 1
            raise
```

The decorator object itself becomes a small stateful component.

---

# 20. Common Mistakes

## Mistake 1 — Forgetting `__call__`

This:

```python
class Logger:

    def __init__(self, func):
        self.func = func
```

is not enough.

Using:

```python
@Logger
def process():
    ...
```

will create a Logger object.

But:

```python
process()
```

will fail because the object isn't callable.

You need:

```python
def __call__(self, ...):
```

---

# 21. Mistake 2 — Forgetting `return`

Wrong:

```python
def __call__(self, *args, **kwargs):
    self.func(*args, **kwargs)
```

Correct:

```python
def __call__(self, *args, **kwargs):
    return self.func(*args, **kwargs)
```

---

# 22. Mistake 3 — Not Forwarding Arguments

Wrong:

```python
def __call__(self):
    return self.func()
```

This only works for functions requiring no arguments.

Better:

```python
def __call__(self, *args, **kwargs):
    return self.func(*args, **kwargs)
```

---

# 23. Mistake 4 — Forgetting Metadata

A class decorator replaces the original function with an object.

Therefore:

```python
process.__name__
```

may not behave like a normal function.

For example:

```python
@Logger
def process():
    pass
```

After decoration:

```text
process
   ↓
Logger instance
```

The object itself does not automatically have all the metadata of the original function.

This is one reason function-based decorators with `functools.wraps` are often simpler.

---

# 24. Metadata Preservation

One approach is to use:

```python
from functools import update_wrapper
```

Example:

```python
class Logger:

    def __init__(self, func):

        self.func = func

        update_wrapper(self, func)

    def __call__(self, *args, **kwargs):

        print("Calling function")

        return self.func(*args, **kwargs)
```

Now the decorator object receives useful metadata from the original function.

This becomes important for:

* Introspection
* Debugging
* Documentation
* Frameworks
* Testing

---

# 25. Class Decorator and `__wrapped__`

`update_wrapper()` also helps establish wrapper metadata such as:

```python
__name__
__doc__
__module__
__annotations__
__wrapped__
```

This allows tools and developers to understand the original function behind the decorator.

---

# 26. Important Relationships

Your learning journey now looks like:

```text
Functions
    ↓
First-Class Functions
    ↓
Higher-Order Functions
    ↓
Nested Functions
    ↓
Closures
    ↓
Function Decorators
    ↓
Decorator Arguments
    ↓
Decorator Factories
    ↓
Multiple Decorators
    ↓
Class as Decorator
```

The major conceptual transition is:

```text
Function-based state
       ↓
Closure
```

versus:

```text
Object-based state
       ↓
Instance Attributes
```

---

# 27. Function Decorator vs Class Decorator

| Feature           | Function Decorator | Class Decorator          |
| ----------------- | ------------------ | ------------------------ |
| Wrapper           | Nested function    | `__call__()`             |
| State             | Closure            | Instance attributes      |
| Configuration     | Decorator factory  | Constructor              |
| Readability       | Usually simpler    | Better for complex state |
| Metadata          | `@wraps`           | `update_wrapper()`       |
| Stateful behavior | Possible           | Natural                  |
| Object lifecycle  | Less explicit      | Explicit                 |
| Complexity        | Lower              | Higher                   |

---

# 28. Interview Questions

### Basic

1. Can a class be used as a decorator?
2. Why does a class work as a decorator?
3. What is `__call__()`?
4. What does `@Logger` become internally?
5. What happens when a decorated function is called?

### Internal

6. Explain:

```python
@Logger
def process():
    ...
```

7. Why does `process()` invoke `Logger.__call__()`?
8. What does `self.func` represent?
9. When does `__init__()` execute?
10. When does `__call__()` execute?

### Comparison

11. What is the difference between a function decorator and class decorator?
12. How does closure-based state compare with object-based state?
13. When would you choose a class decorator over a function decorator?

### Advanced

14. How would you implement a configurable class decorator?
15. What is the difference between:

```python
@Logger
```

and:

```python
@Logger("INFO")
```

16. Why can metadata be lost with a class decorator?
17. How does `update_wrapper()` help?
18. How would you implement a call counter using a class decorator?
19. How would you make a class decorator thread-safe if it maintains shared mutable state?
20. What are the performance implications of a class-based decorator?

---

# 29. Senior-Level Mental Model

Don't think:

> "A class can somehow be written above a function."

Think in terms of the **callable protocol**.

Python doesn't require:

```text
"this must be a function"
```

for:

```python
something()
```

It requires the object to be callable.

A class instance becomes callable through:

```python
__call__()
```

Therefore:

```python
@Logger
def process():
    ...
```

becomes:

```python
process = Logger(process)
```

Now:

```text
process
   ↓
Logger instance
   ↓
instance()
   ↓
__call__()
   ↓
self.func()
   ↓
Original function
```

---

# 30. Final Senior Mental Model

There are **three identities** you must keep separate.

### Before decoration

```text
process
   ↓
Function object
```

### After decoration

```text
process
   ↓
Decorator object
   ↓
self.func → Original function
```

### During execution

```text
process()
   ↓
Decorator object's __call__()
   ↓
self.func(...)
   ↓
Original function
```

Visualize it:

```text
                 process()
                     │
                     ▼
          ┌────────────────────┐
          │   Logger Object    │
          │                    │
          │   self.func ───────┼──────┐
          │                    │      │
          │   __call__()       │      ▼
          └────────────────────┘  Original
                                  Function
```

> **Key takeaway:**
>
> A class can act as a decorator because its instance can be made callable through `__call__()`.
>
> ```python
> @Logger
> def process():
>     ...
> ```
>
> becomes:
>
> ```python
> process = Logger(process)
> ```
>
> After decoration, `process` refers to a `Logger` instance rather than directly to the original function. Calling `process()` invokes `Logger.__call__()`, which can execute additional behavior and then delegate to `self.func()`.
>
> **Function decorators commonly use closures for state; class decorators commonly use instance attributes for state.**

---

