# `07-decorators-fundamentals.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, `*args/**kwargs`, LEGB, nested functions, higher-order functions, closures
> **Core idea:** A decorator is a mechanism for **wrapping or transforming a callable without changing its original source code**.

---

# 1. What is it?

A **decorator** is a callable that takes another callable as input and returns a callable, usually by wrapping the original callable with additional behavior.

Basic example:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

We can decorate a function:

```python
@decorator
def greet():
    print("Hello")
```

When we call:

```python
greet()
```

Output:

```text
Before
Hello
After
```

The important thing is that the original `greet()` function was not modified internally.

Instead, its reference was replaced by the returned `wrapper()`.

---

# 2. Why does it exist?

Suppose we have many functions:

```python
def create_user():
    ...


def delete_user():
    ...


def update_user():
    ...
```

We want logging around every function:

```text
Before function
    ↓
Execute function
    ↓
After function
```

Without decorators, we might repeatedly write:

```python
def create_user():
    print("Starting")
    ...
    print("Finished")
```

```python
def delete_user():
    print("Starting")
    ...
    print("Finished")
```

This creates:

* duplicated code
* poor maintainability
* mixed business logic and cross-cutting logic

A decorator allows us to separate them.

```text
Business Logic
      +
Cross-cutting Behavior
      ↓
Decorator
```

Typical uses:

* Logging
* Authentication/authorization
* Timing
* Validation
* Caching
* Retry logic
* Metrics
* Tracing
* Transactions
* Access control

---

# 3. How does it work internally?

This is the **most important part**.

Consider:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

And:

```python
@decorator
def greet():
    print("Hello")
```

Python essentially transforms this:

```python
@decorator
def greet():
    print("Hello")
```

into:

```python
def greet():
    print("Hello")

greet = decorator(greet)
```

This is the fundamental mental model.

---

## What happens?

Initially:

```text
greet
  ↓
original greet function
```

Then:

```python
greet = decorator(greet)
```

The original function is passed to `decorator()`:

```text
original greet
      ↓
  decorator()
      ↓
   wrapper
      ↓
returned
```

Finally:

```text
greet
  ↓
wrapper function
  ↓
original greet
```

So when you execute:

```python
greet()
```

you are actually calling:

```text
wrapper()
   ↓
print("Before")
   ↓
original greet()
   ↓
print("After")
```

---

# 4. Core concepts

## 4.1 Decorator is based on first-class functions

You already learned that functions are objects.

Therefore:

```python
def greet():
    print("Hello")
```

allows:

```python
x = greet
```

It also allows:

```python
decorator(greet)
```

So decorators are possible because Python treats functions as first-class objects.

---

# 4.2 Decorator is a higher-order function

Recall:

> A higher-order function accepts a function or returns a function.

A decorator does both:

```python
def decorator(func):

    def wrapper():
        ...
    
    return wrapper
```

It:

1. receives `func`
2. creates `wrapper`
3. returns `wrapper`

Therefore:

```text
Decorator
   ↓
Higher-Order Function
```

---

# 4.3 Wrapper function

The `wrapper()` is the function that surrounds the original function.

Example:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

Conceptually:

```text
           wrapper()
          /        \
         /          \
    Before          After
        \            /
         \          /
        original function
```

The wrapper controls the execution around the original function.

---

# 4.4 Decorator does not necessarily mean wrapper

Most beginner decorators use a wrapper:

```python
def decorator(func):

    def wrapper():
        ...
```

But technically, a decorator is simply a callable that transforms/returns another callable.

For example:

```python
def decorator(func):
    return another_function
```

The decorator doesn't have to literally define a nested `wrapper()`.

The word **wrapper** describes a common implementation technique.

---

# 4.5 Decorator transformation

This:

```python
@decorator
def greet():
    print("Hello")
```

means:

```python
def greet():
    print("Hello")

greet = decorator(greet)
```

This is called **decoration**.

After decoration, the name `greet` refers to the returned object.

That may be the wrapper.

---

# 5. Execution flow

Consider:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper


@decorator
def greet():
    print("Hello")
```

There are actually **two different execution phases** to understand.

---

## Phase 1 — Decoration time

When Python reaches:

```python
@decorator
def greet():
    print("Hello")
```

the function object is created first.

Then conceptually:

```python
greet = decorator(greet)
```

Flow:

```text
Create original greet()
       ↓
Pass greet to decorator()
       ↓
func → original greet
       ↓
Create wrapper()
       ↓
wrapper closes over func
       ↓
return wrapper
       ↓
greet → wrapper
```

Notice:

> The original `greet()` body has NOT executed yet.

---

## Phase 2 — Call time

Later:

```python
greet()
```

Since:

```text
greet → wrapper
```

Python executes:

```text
wrapper()
   ↓
Before
   ↓
func()
   ↓
original greet()
   ↓
Hello
   ↓
After
```

This distinction between **decoration time** and **call time** is extremely important.

---

# 6. Decoration Time vs Call Time

### Decoration time

```python
@decorator
def greet():
    ...
```

The decorator executes.

### Call time

```python
greet()
```

The wrapper executes.

Think:

```text
             DECORATION TIME
                    │
                    ▼
          decorator(original)
                    │
                    ▼
                wrapper
                    │
                    ▼
             assigned to greet
                    │
                    │
                    ▼
               CALL TIME
                    │
                    ▼
                 greet()
                    │
                    ▼
                wrapper()
                    │
                    ▼
             original function
```

---

# 7. Closure Connection

Your previous topic was closures.

Now you can see why closures matter.

```python
def decorator(func):

    def wrapper():
        func()

    return wrapper
```

`wrapper()` accesses:

```python
func
```

which belongs to the enclosing `decorator()` scope.

Therefore:

```text
decorator()
    │
    ├── func
    │
    └── wrapper()
            │
            └── references func
```

When `wrapper` is returned, it retains access to `func`.

That is closure behavior.

So decorators commonly use:

```text
Higher-order function
        +
Nested function
        +
Closure
        ↓
    Decorator
```

---

# 8. Simple Logging Decorator

A realistic example:

```python
def log_execution(func):

    def wrapper():
        print(f"Starting {func.__name__}")

        result = func()

        print(f"Finished {func.__name__}")

        return result

    return wrapper
```

Usage:

```python
@log_execution
def process():
    print("Processing...")
```

Calling:

```python
process()
```

Output:

```text
Starting process
Processing...
Finished process
```

The `process()` function contains only business logic:

```python
def process():
    print("Processing...")
```

Logging is externalized into the decorator.

---

# 9. Decorator as Separation of Concerns

Without decorator:

```python
def process():
    print("Starting")
    # business logic
    print("Finished")
```

Now logging and business logic are mixed.

With decorator:

```python
@log_execution
def process():
    # business logic
```

Conceptually:

```text
process()
   │
   ├── logging responsibility → decorator
   │
   └── business responsibility → process
```

This is a major reason decorators are valuable in production systems.

---

# 10. Returning the Original Function's Result

A decorator should normally preserve the result of the decorated function.

Good:

```python
def decorator(func):

    def wrapper():
        result = func()
        return result

    return wrapper
```

If:

```python
@decorator
def add():
    return 10 + 20
```

then:

```python
result = add()
```

should give:

```text
30
```

If the wrapper doesn't return the result:

```python
def wrapper():
    func()
```

then:

```python
result = add()
```

becomes:

```text
None
```

This is a very common decorator mistake.

---

# 11. Passing Arguments Through a Decorator

Consider:

```python
@decorator
def add(a, b):
    return a + b
```

A wrapper like this will fail:

```python
def decorator(func):

    def wrapper():
        return func()

    return wrapper
```

Why?

Because:

```python
add(10, 20)
```

passes arguments to `wrapper()`, but `wrapper()` accepts none.

The general solution is:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

This is why your previous `*args` and `**kwargs` topic becomes important here.

We will cover this properly in the **next decorator topic**.

---

# 12. `functools.wraps`

Consider:

```python
def decorator(func):

    def wrapper():
        return func()

    return wrapper
```

After decoration:

```python
@decorator
def greet():
    """Say hello."""
    print("Hello")
```

The name and metadata of `greet` may now appear to belong to `wrapper`.

For example:

```python
print(greet.__name__)
```

may produce:

```text
wrapper
```

This is undesirable for debugging and tooling.

Python provides:

```python
from functools import wraps
```

Use:

```python
def decorator(func):

    @wraps(func)
    def wrapper():
        return func()

    return wrapper
```

Now metadata such as the function name and docstring are preserved more appropriately.

---

# 13. Why `functools.wraps` Exists

A decorator changes the reference:

```text
greet → wrapper
```

But logically, developers still think:

```text
greet → original function
```

`functools.wraps` helps the wrapper present important metadata from the original function.

Conceptually:

```text
Original function
      │
      │ metadata
      ▼
   @wraps
      │
      ▼
Wrapper
```

It is not merely cosmetic.

It improves:

* Debugging
* Logging
* Documentation
* Introspection
* Tracebacks
* Tooling

---

# 14. A Production-Quality Basic Decorator

A better basic pattern is:

```python
from functools import wraps


def log_execution(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print(f"Starting {func.__name__}")

        result = func(*args, **kwargs)

        print(f"Finished {func.__name__}")

        return result

    return wrapper
```

Usage:

```python
@log_execution
def add(a, b):
    return a + b
```

Then:

```python
result = add(10, 20)
```

Output:

```text
Starting add
Finished add
```

and:

```python
result == 30
```

---

# 15. Decorators Can Modify Behavior

A decorator doesn't only add behavior.

It can also:

* Change arguments
* Change return values
* Prevent execution
* Retry execution
* Cache results
* Validate inputs
* Handle exceptions

Example:

```python
def uppercase(func):

    def wrapper():
        result = func()
        return result.upper()

    return wrapper
```

Then:

```python
@uppercase
def greet():
    return "hello"
```

Now:

```python
print(greet())
```

returns:

```text
HELLO
```

So a decorator can transform behavior.

---

# 16. Decorators Can Prevent Execution

Example:

```python
def check_access(func):

    def wrapper(user):
        if user != "admin":
            print("Access denied")
            return

        return func(user)

    return wrapper
```

Usage:

```python
@check_access
def delete_data(user):
    print("Deleting data...")
```

Now:

```python
delete_data("guest")
```

doesn't execute the original function.

This is a simplified example of how authorization decorators can work.

---

# 17. Decorator as a Gatekeeper

Think of a decorator as a gate:

```text
                 Function Call
                      │
                      ▼
               ┌─────────────┐
               │   Wrapper   │
               └─────────────┘
                  │       │
               allowed   denied
                  │       │
                  ▼       ▼
             Original    Stop
             Function
```

This mental model is useful for:

* Authentication
* Authorization
* Validation
* Rate limiting
* Feature flags

---

# 18. Multiple Decorators — Basic Idea

Python allows:

```python
@decorator1
@decorator2
def greet():
    ...
```

Conceptually:

```python
greet = decorator1(decorator2(greet))
```

So decoration occurs from the **bottom upward**.

```text
greet
  ↓
decorator2
  ↓
decorator1
```

But when the resulting function is called, execution flows through the outermost decorator first.

We'll study stacked decorators separately.

---

# 19. Decorator Doesn't Change the Original Function Object

This is an important mental distinction.

Suppose:

```python
def decorator(func):

    def wrapper():
        return func()

    return wrapper
```

Then:

```python
@decorator
def greet():
    print("Hello")
```

The original function object still exists because the wrapper's closure references it.

Conceptually:

```text
greet
 ↓
wrapper
 ↓
closure
 ↓
original greet function
```

The name `greet` now points to the wrapper.

---

# 20. Reference Transformation

This is perhaps the most important diagram in decorators.

Before:

```text
greet ─────────► Original Function
```

After:

```text
greet ─────────► Wrapper
                    │
                    │ closure reference
                    ▼
              Original Function
```

Therefore:

```python
greet()
```

doesn't directly call the original function.

It calls the wrapper.

The wrapper decides whether and when to call the original function.

---

# 21. Decorators and Callables

Decorators aren't restricted to normal functions.

A decorator can work with any suitable **callable**.

A callable is something that can be invoked using:

```python
obj()
```

Examples include:

* Functions
* Methods
* Classes
* Objects implementing `__call__`

This becomes important when we later study **classes as decorators**.

---

# 22. Common Mistakes

## Mistake 1 — Forgetting to return the wrapper

Wrong:

```python
def decorator(func):

    def wrapper():
        func()
```

There is no:

```python
return wrapper
```

So:

```python
@decorator
def greet():
    ...
```

causes the decorated name to become `None`.

Correct:

```python
def decorator(func):

    def wrapper():
        func()

    return wrapper
```

---

## Mistake 2 — Forgetting to return the original result

Wrong:

```python
def wrapper():
    func()
```

Correct:

```python
def wrapper():
    return func()
```

---

## Mistake 3 — Confusing `func` with `func()`

```python
func
```

means:

> Function object/reference.

```python
func()
```

means:

> Execute the function.

This distinction is fundamental.

---

## Mistake 4 — Thinking `@decorator` executes the decorated function immediately

It doesn't.

During decoration:

```python
greet = decorator(greet)
```

The original `greet()` body is not necessarily executed.

The wrapper runs when:

```python
greet()
```

is later called.

---

## Mistake 5 — Forgetting `*args` and `**kwargs`

A wrapper that accepts no arguments cannot transparently wrap arbitrary functions.

For general-purpose decorators:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

## Mistake 6 — Forgetting `@wraps`

Without:

```python
@wraps(func)
```

function metadata can become misleading.

---

# 23. Production Perspective

Decorators are especially useful for **cross-cutting concerns**.

Examples:

```text
Authentication
      ↓
Authorization
      ↓
Logging
      ↓
Metrics
      ↓
Caching
      ↓
Retry
      ↓
Business Logic
```

For example, a web application might conceptually have:

```python
@authenticate
@authorize
@log_request
def get_user():
    ...
```

Each decorator handles a separate concern.

The business function stays focused.

---

# 24. Production Example — Timing

A simple timing decorator:

```python
import time
from functools import wraps


def timer(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        start = time.perf_counter()

        result = func(*args, **kwargs)

        end = time.perf_counter()

        print(f"{func.__name__}: {end - start:.4f}s")

        return result

    return wrapper
```

Usage:

```python
@timer
def process():
    ...
```

The business function doesn't need timing code.

The decorator handles it.

---

# 25. Production Example — Retry

Conceptually:

```python
def retry(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        for attempt in range(3):
            try:
                return func(*args, **kwargs)
            except Exception:
                if attempt == 2:
                    raise

    return wrapper
```

Then:

```python
@retry
def call_external_service():
    ...
```

The retry behavior is separated from the business operation.

In real systems, retry decorators should also consider:

* Which exceptions are retryable
* Backoff
* Jitter
* Maximum delay
* Idempotency
* Observability

The simple example is only the conceptual foundation.

---

# 26. Decorators and Observability

Decorators are often useful for instrumenting code.

For example:

```text
Function
   ↓
Decorator
   ├── Log
   ├── Measure latency
   ├── Count calls
   └── Record errors
   ↓
Original Function
```

This is valuable because instrumentation can be added without putting monitoring code throughout business logic.

---

# 27. When NOT to Use Decorators

Decorators are powerful, but overusing them creates hidden behavior.

Consider:

```python
@a
@b
@c
@d
@e
def process():
    ...
```

A developer reading `process()` now needs to understand five layers before understanding what happens.

Problems can include:

* Difficult debugging
* Hidden control flow
* Unexpected side effects
* Hard-to-follow stack traces
* Complicated testing
* Ordering dependencies

Senior engineering principle:

> **Use decorators when the added behavior is conceptually orthogonal to the function's primary responsibility.**

---

# 28. Decorator vs Explicit Function Call

Decorator:

```python
@log_execution
def process():
    ...
```

Explicit:

```python
def process():
    ...


log_execution(process)
```

The decorator syntax provides a cleaner declaration that:

> `process` should always be wrapped by `log_execution`.

Use decorators when that relationship is stable and meaningful.

---

# 29. Important Relationships

Your learning path now becomes:

```text
FUNCTION
   │
   ▼
Function is an object
   │
   ▼
Can pass function
   │
   ▼
Higher-Order Function
   │
   ▼
Nested Function
   │
   ▼
Closure
   │
   ▼
Decorator
   │
   ├── receives original function
   │
   ├── creates wrapper
   │
   ├── wrapper closes over original function
   │
   └── returns wrapper
```

And:

```text
@decorator
     ↓
decorator syntax
     ↓
greet = decorator(greet)
```

This one line is the foundation you should remember.

---

# 30. Common Interview Questions

### Basic

1. What is a decorator in Python?
2. Why do we use decorators?
3. How does `@decorator` work?
4. What is a wrapper function?
5. Why does a decorator usually return a function?
6. What is the difference between `func` and `func()`?

### Internal Working

7. What happens when Python encounters `@decorator`?
8. When does the decorator execute?
9. When does the wrapper execute?
10. Does decoration execute the original function?
11. What happens to the original function after decoration?
12. How does the wrapper access the original function?

### Connections

13. Why are decorators considered higher-order functions?
14. How do closures enable decorators?
15. How are decorators related to first-class functions?
16. How are `*args` and `**kwargs` used in decorators?
17. Why is `functools.wraps` important?

### Senior

18. What is the difference between decoration time and call time?
19. Explain `greet = decorator(greet)` internally.
20. How would you design a logging decorator?
21. How would you design a retry decorator?
22. What problems can excessive decorator usage create?
23. When would you prefer explicit composition over decorators?
24. Can a class be a decorator?
25. Can a decorator decorate a class?
26. What is a callable?
27. Can an object be used as a decorator?
28. How do stacked decorators execute?

---

# 31. Senior-Level Mental Model

Don't memorize:

> "A decorator is a function with `@`."

That's syntax, not the concept.

Instead remember:

> **A decorator is a callable transformation mechanism. It receives a callable and returns another callable that changes, extends, restricts, or observes the original behavior.**

The fundamental transformation:

```text
Before decoration:

name
 │
 ▼
Original Function
```

After:

```text
name
 │
 ▼
Wrapper
 │
 │ closure
 ▼
Original Function
```

And:

```python
@decorator
def greet():
    ...
```

means:

```python
greet = decorator(greet)
```

The complete mental model:

```text
                    DECORATOR
                        │
                        ▼
                 receives callable
                        │
                        ▼
                  creates wrapper
                        │
                        ▼
              wrapper closes over
               original callable
                        │
                        ▼
                 returns wrapper
                        │
                        ▼
              name points to wrapper
                        │
                        ▼
                 caller invokes
                        │
                        ▼
                     wrapper
                   /        \
                  /          \
             extra logic    original
                              logic
```

---

# 32. Final Revision Table

| Concept               | Mental Model                                     |
| --------------------- | ------------------------------------------------ |
| Decorator             | Callable that transforms another callable        |
| `@decorator`          | Syntax for applying a decorator                  |
| `wrapper`             | Function surrounding the original function       |
| Decoration time       | Time when decorator transforms the function      |
| Call time             | Time when decorated function is invoked          |
| Closure               | Allows wrapper to retain original function       |
| `functools.wraps`     | Preserves important function metadata            |
| `*args`               | Forwards positional arguments                    |
| `**kwargs`            | Forwards keyword arguments                       |
| Cross-cutting concern | Behavior shared across many functions            |
| Stacked decorators    | Multiple transformations applied to one callable |
| Callable              | Object that can be invoked with `()`             |

---

# 33. Final Mental Model

```text
                  PYTHON FUNCTIONS
                         │
                         ▼
                 First-Class Objects
                         │
                         ▼
                Higher-Order Functions
                         │
                         ▼
                  Nested Functions
                         │
                         ▼
                     Closures
                         │
                         ▼
                    DECORATORS
                         │
          ┌──────────────┴──────────────┐
          │                             │
     Original Function              Wrapper
          │                             │
          │                             ├── logging
          │                             ├── validation
          │                             ├── auth
          │                             ├── timing
          │                             ├── retry
          │                             └── caching
          │
          └───────────────┬─────────────┘
                          ▼
                  Transformed Behavior
```

> **Key takeaway:**
> **A decorator is not magic and `@` is not the important part. The real mechanism is function transformation: `greet = decorator(greet)`. The decorator receives the original function, creates or returns another callable, and the new callable can retain the original function through a closure. This is why your previous topics—first-class functions, higher-order functions, nested functions, and closures—directly lead into decorators.**
