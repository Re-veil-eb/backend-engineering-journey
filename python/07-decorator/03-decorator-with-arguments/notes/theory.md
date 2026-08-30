# `09-decorators-with-arguments.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, `*args`, `**kwargs`, LEGB, nested functions, higher-order functions, closures, decorators, argument forwarding
> **Core idea:** A decorator with arguments is a **function factory**. The arguments configure the decorator first; the resulting decorator then receives the function.

---

# 1. What is it?

A normal decorator looks like:

```python
@decorator
def greet():
    print("Hello")
```

A decorator with arguments looks like:

```python
@decorator("ADMIN")
def greet():
    print("Hello")
```

The important difference is:

```text
@decorator
```

versus:

```text
@decorator("ADMIN")
```

In the second case, `decorator("ADMIN")` must **first execute and return a decorator**.

So there are multiple layers:

```text
decorator arguments
       ↓
decorator factory
       ↓
actual decorator
       ↓
wrapper
       ↓
original function
```

---

# 2. Why does it exist?

Sometimes the decorator behavior needs configuration.

For example, suppose we want a logging decorator.

Without arguments:

```python
@logger
def process():
    ...
```

Maybe we want different log levels:

```python
@logger("INFO")
def process():
    ...
```

Or authorization:

```python
@requires_role("admin")
def delete_user():
    ...
```

Or retry configuration:

```python
@retry(max_attempts=3)
def call_api():
    ...
```

Or caching:

```python
@cache(ttl=60)
def get_user():
    ...
```

The decorator argument allows the caller to **configure the behavior**.

---

# 3. How does it work internally?

This is the most important part.

Consider:

```python
def repeat(count):

    def decorator(func):

        def wrapper(*args, **kwargs):
            for _ in range(count):
                func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@repeat(3)
def greet():
    print("Hello")
```

Python conceptually transforms this into:

```python
def greet():
    print("Hello")

greet = repeat(3)(greet)
```

This is the key mental model.

---

# 4. The Three Function Calls

A decorator with arguments introduces another layer.

Consider:

```python
@repeat(3)
def greet():
    print("Hello")
```

Think of it as:

```text
repeat(3)
   ↓
returns decorator
   ↓
decorator(greet)
   ↓
returns wrapper
   ↓
greet = wrapper
```

So there are three important stages:

### Stage 1

```python
repeat(3)
```

Configures the decorator.

### Stage 2

```python
decorator(greet)
```

Receives the original function.

### Stage 3

```python
greet()
```

Executes the wrapper.

---

# 5. Core Concepts

## 5.1 Decorator Factory

A function that accepts configuration and returns a decorator is commonly called a **decorator factory**.

Example:

```python
def repeat(count):

    def decorator(func):

        def wrapper():
            for _ in range(count):
                func()

        return wrapper

    return decorator
```

Here:

```python
repeat
```

is the decorator factory.

It produces:

```python
decorator
```

which then produces:

```python
wrapper
```

---

# 6. The Function Layers

This is the structure you should memorize:

```text
repeat(count)
     │
     │ receives configuration
     ▼
 decorator(func)
     │
     │ receives original function
     ▼
 wrapper(*args, **kwargs)
     │
     │ receives runtime arguments
     ▼
 original function
```

There are **three different types of data** moving through these layers:

```text
Configuration
     ↓
count = 3

Function
     ↓
func = greet

Runtime arguments
     ↓
args / kwargs
```

Do not mix these three levels.

---

# 7. Execution Flow

Consider:

```python
from functools import wraps


def repeat(count):

    def decorator(func):

        @wraps(func)
        def wrapper(*args, **kwargs):

            for _ in range(count):
                func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@repeat(3)
def greet(name):
    print(f"Hello {name}")
```

Call:

```python
greet("Chinnu")
```

Execution flow:

```text
                 @repeat(3)
                     │
                     ▼
                repeat(3)
                     │
                     ▼
               decorator
                     │
                     ▼
              decorator(greet)
                     │
                     ▼
                  wrapper
                     │
                     ▼
             greet("Chinnu")
                     │
                     ▼
            wrapper("Chinnu")
                     │
                     ▼
            count = 3
                     │
              ┌──────┼──────┐
              ▼      ▼      ▼
            greet   greet   greet
```

Output:

```text
Hello Chinnu
Hello Chinnu
Hello Chinnu
```

---

# 8. Why Can't We Write `@decorator(3)` With a Normal Decorator?

Suppose:

```python
def decorator(func):

    def wrapper():
        func()

    return wrapper
```

Then:

```python
@decorator(3)
def greet():
    ...
```

is incorrect.

Why?

Because Python interprets:

```python
decorator(3)
```

as:

```python
decorator(func=3)
```

But `decorator()` expects a function.

You are giving it:

```text
3
```

instead of:

```text
function
```

Therefore, a separate outer function is required to receive the configuration.

---

# 9. Normal Decorator vs Decorator With Arguments

## Normal decorator

```python
@decorator
def greet():
    ...
```

Equivalent to:

```python
greet = decorator(greet)
```

Only one transformation layer is needed.

---

## Decorator with arguments

```python
@decorator("INFO")
def greet():
    ...
```

Equivalent to:

```python
greet = decorator("INFO")(greet)
```

There are two function calls before the final decorated function exists:

```text
decorator("INFO")
        ↓
returns decorator
        ↓
decorator(greet)
        ↓
returns wrapper
```

This distinction is extremely important in interviews.

---

# 10. Closure Connection

Decorator factories rely heavily on closures.

Consider:

```python
def repeat(count):

    def decorator(func):

        def wrapper():
            for _ in range(count):
                func()

        return wrapper

    return decorator
```

`wrapper()` accesses:

```python
count
```

from the outer function.

It also accesses:

```python
func
```

from the enclosing `decorator()` function.

Therefore, there are nested scopes.

```text
repeat()
   │
   ├── count
   │
   └── decorator()
          │
          ├── func
          │
          └── wrapper()
                 │
                 ├── count
                 └── func
```

The wrapper retains access to both values through closures.

This is a direct application of your previous **closure + LEGB** learning.

---

# 11. Two Closure Variables

This is particularly important.

In:

```python
def repeat(count):

    def decorator(func):

        def wrapper():
            for _ in range(count):
                func()

        return wrapper

    return decorator
```

`wrapper()` uses:

```text
count
func
```

Where do they come from?

```text
count
  ↑
repeat()

func
  ↑
decorator()
```

So the wrapper's enclosing environment effectively contains access to both.

Conceptually:

```text
wrapper
  │
  ├── count = 3
  │
  └── func = original greet
```

---

# 12. Passing Arguments to the Decorated Function

Don't confuse:

```python
@repeat(3)
```

with:

```python
greet("Chinnu")
```

These are arguments at **different levels**.

### Decorator configuration

```python
@repeat(3)
```

`3` configures the decorator.

### Function runtime argument

```python
greet("Chinnu")
```

`"Chinnu"` goes to the decorated function.

The complete flow:

```text
@repeat(3)
     │
     ▼
configuration
     │
     ▼
decorator
     │
     ▼
greet function
     │
     ▼
greet("Chinnu")
     │
     ▼
wrapper("Chinnu")
     │
     ▼
original greet("Chinnu")
```

---

# 13. Using `*args` and `**kwargs`

A reusable decorator with arguments should usually support arbitrary function arguments.

Example:

```python
from functools import wraps


def repeat(count):

    def decorator(func):

        @wraps(func)
        def wrapper(*args, **kwargs):

            result = None

            for _ in range(count):
                result = func(*args, **kwargs)

            return result

        return wrapper

    return decorator
```

Usage:

```python
@repeat(3)
def add(a, b):
    return a + b
```

Call:

```python
result = add(10, 20)
```

The runtime arguments flow:

```text
add(10, 20)
      ↓
wrapper(*args, **kwargs)
      ↓
args = (10, 20)
kwargs = {}
      ↓
func(*args, **kwargs)
      ↓
original add(10, 20)
```

---

# 14. Decorator Arguments Can Be Keyword Arguments

You don't have to use only positional configuration.

Example:

```python
@retry(max_attempts=3)
def call_api():
    ...
```

Implementation:

```python
def retry(max_attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts - 1:
                        raise

        return wrapper

    return decorator
```

Here:

```text
max_attempts
```

belongs to the **decorator configuration**, not the original function.

---

# 15. Multiple Configuration Arguments

A decorator factory can accept multiple parameters.

```python
def retry(max_attempts, delay):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    if attempt == max_attempts - 1:
                        raise

        return wrapper

    return decorator
```

Usage:

```python
@retry(max_attempts=3, delay=2)
def call_api():
    ...
```

Now:

```text
retry()
 ├── max_attempts = 3
 └── delay = 2
       ↓
 decorator(func)
       ↓
 wrapper(*args, **kwargs)
```

---

# 16. Argument Scope Separation

This is a common interview trap.

Consider:

```python
@retry(max_attempts=3)
def get_user(user_id):
    ...
```

There are two completely different argument groups:

```text
Decorator configuration
        ↓
max_attempts = 3
```

and:

```text
Function runtime argument
        ↓
user_id
```

The scopes are:

```text
retry(max_attempts)
       │
       ▼
 decorator(func)
       │
       ▼
 wrapper(*args, **kwargs)
```

---

# 17. Example — Authorization

A useful example:

```python
from functools import wraps


def requires_role(required_role):

    def decorator(func):

        @wraps(func)
        def wrapper(user, *args, **kwargs):

            if user.role != required_role:
                raise PermissionError("Access denied")

            return func(user, *args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@requires_role("admin")
def delete_user(user, user_id):
    print(f"Deleting user {user_id}")
```

Here:

```python
@requires_role("admin")
```

configures the decorator.

While:

```python
delete_user(user, user_id)
```

provides runtime arguments.

---

# 18. Example — Logging Level

```python
from functools import wraps


def log(level):

    def decorator(func):

        @wraps(func)
        def wrapper(*args, **kwargs):

            print(f"[{level}] Calling {func.__name__}")

            return func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@log("INFO")
def process_payment():
    print("Processing payment")
```

Output:

```text
[INFO] Calling process_payment
Processing payment
```

The configuration:

```text
"INFO"
```

is captured by the closure.

---

# 19. Example — Cache Configuration

Conceptually:

```python
def cache(ttl):

    def decorator(func):

        def wrapper(*args, **kwargs):
            # cache logic using ttl
            return func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@cache(ttl=60)
def get_user(user_id):
    ...
```

Here:

```text
ttl = 60
```

configures the caching behavior.

---

# 20. Function Factory Connection

A decorator with arguments is essentially a **factory that creates decorators**.

Compare:

### Normal factory

```python
def create_multiplier(x):

    def multiply(value):
        return value * x

    return multiply
```

Usage:

```python
double = create_multiplier(2)
```

Similarly:

### Decorator factory

```python
def repeat(count):

    def decorator(func):
        ...
        return wrapper

    return decorator
```

Usage:

```python
@repeat(3)
def greet():
    ...
```

So:

```text
Function Factory
       ↓
Creates functions

Decorator Factory
       ↓
Creates decorators
```

---

# 21. Execution Timeline

Let's carefully separate **definition time**, **decoration time**, and **call time**.

Consider:

```python
def repeat(count):

    print("repeat called")

    def decorator(func):

        print("decorator called")

        def wrapper(*args, **kwargs):

            print("wrapper called")

            return func(*args, **kwargs)

        return wrapper

    return decorator


@repeat(3)
def greet(name):

    print("Hello", name)
```

When Python loads this code:

### Step 1

The `repeat` function is defined.

No execution of its body yet.

### Step 2

Python creates the original `greet` function.

### Step 3

Python evaluates:

```python
repeat(3)
```

Output:

```text
repeat called
```

### Step 4

`repeat(3)` returns:

```text
decorator
```

### Step 5

Python applies it:

```python
decorator(greet)
```

Output:

```text
decorator called
```

### Step 6

`decorator()` returns:

```text
wrapper
```

### Step 7

The name `greet` now points to:

```text
wrapper
```

No `"Hello"` has been printed yet.

### Step 8

Later:

```python
greet("Chinnu")
```

Output:

```text
wrapper called
Hello Chinnu
```

---

# 22. Complete Execution Diagram

```text
             FUNCTION DEFINITION
                     │
                     ▼
                  greet
                     │
                     ▼
            @repeat(3) evaluated
                     │
                     ▼
                repeat(3)
                     │
                     ▼
                decorator
                     │
                     ▼
             decorator(greet)
                     │
                     ▼
                  wrapper
                     │
                     ▼
              greet = wrapper
                     │
                     │
                     ▼
                 CALL TIME
                     │
                     ▼
             greet("Chinnu")
                     │
                     ▼
            wrapper("Chinnu")
                     │
                     ▼
            original greet()
```

---

# 23. Common Mistakes

## Mistake 1 — Treating decorator arguments as function arguments

Incorrect mental model:

```python
@retry(3)
def process():
    ...
```

Thinking:

```text
3 → process()
```

Wrong.

`3` belongs to the decorator factory.

---

## Mistake 2 — Forgetting the extra function layer

Incorrect:

```python
def retry(max_attempts):

    def wrapper(*args, **kwargs):
        ...

    return wrapper
```

This structure makes `max_attempts` directly receive the function later, but it doesn't follow the usual decorator-factory structure correctly.

Correct:

```python
def retry(max_attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):
            ...

        return wrapper

    return decorator
```

There are three layers:

```text
factory
  ↓
decorator
  ↓
wrapper
```

---

# 24. Mistake — Calling the Original Function Too Early

Wrong:

```python
def decorator(func):

    result = func()

    def wrapper():
        return result

    return wrapper
```

This executes the original function during decoration.

Usually you want:

```python
def decorator(func):

    def wrapper():
        return func()

    return wrapper
```

The function should execute when the wrapper is called.

---

# 25. Mistake — Forgetting `@wraps`

Use:

```python
from functools import wraps
```

and:

```python
@wraps(func)
def wrapper(*args, **kwargs):
    ...
```

This keeps the wrapper's metadata aligned with the original function.

---

# 26. Mistake — Forgetting to Return the Wrapper

Wrong:

```python
def decorator(func):

    def wrapper():
        ...

```

Correct:

```python
def decorator(func):

    def wrapper():
        ...

    return wrapper
```

---

# 27. Mistake — Forgetting to Forward Runtime Arguments

Wrong:

```python
def wrapper(*args, **kwargs):
    return func()
```

Correct:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

# 28. Important Relationships

Your learning chain is now:

```text
Functions
   ↓
First-class functions
   ↓
Higher-order functions
   ↓
Nested functions
   ↓
Closures
   ↓
Decorators
   ↓
*args / **kwargs
   ↓
Argument forwarding
   ↓
Decorator Factory
   ↓
Decorator With Arguments
```

And the decorator factory adds another layer:

```text
                    Factory
                       │
                 configuration
                       │
                       ▼
                   Decorator
                       │
                   function
                       │
                       ▼
                    Wrapper
                       │
                runtime arguments
                       │
                       ▼
                 Original function
```

---

# 29. Normal Decorator vs Decorator Factory

| Normal Decorator           | Decorator With Arguments                             |
| -------------------------- | ---------------------------------------------------- |
| `@decorator`               | `@decorator(...)`                                    |
| `decorator(func)`          | `decorator(...)(func)`                               |
| One main decorator layer   | Extra factory layer                                  |
| Receives function directly | First receives configuration                         |
| Returns wrapper            | Factory returns decorator; decorator returns wrapper |
| No configuration required  | Behavior can be configured                           |

Example:

```python
@logger
def process():
    ...
```

Equivalent:

```python
process = logger(process)
```

With arguments:

```python
@logger("INFO")
def process():
    ...
```

Equivalent:

```python
process = logger("INFO")(process)
```

---

# 30. Senior-Level Production Perspective

Decorator factories are useful when behavior needs configuration.

Common patterns:

```python
@retry(max_attempts=3)
```

```python
@cache(ttl=60)
```

```python
@requires_role("admin")
```

```python
@rate_limit(requests=100)
```

```python
@timeout(seconds=5)
```

```python
@log(level="INFO")
```

The architecture becomes:

```text
Configuration
      │
      ▼
Decorator Factory
      │
      ▼
Configured Decorator
      │
      ▼
Function
      │
      ▼
Wrapper
```

This makes the behavior reusable while allowing different functions to use different configurations.

---

# 31. Production Design Considerations

A senior engineer should consider more than simply making the decorator work.

### 1. Preserve metadata

Use:

```python
@wraps(func)
```

### 2. Preserve arguments

Use:

```python
*args, **kwargs
```

when appropriate.

### 3. Preserve return values

Use:

```python
return func(*args, **kwargs)
```

### 4. Preserve exception semantics

Don't silently swallow errors.

### 5. Avoid hidden side effects

The decorator should have a clear responsibility.

### 6. Keep configuration explicit

Prefer:

```python
@retry(max_attempts=3)
```

over unclear magic values.

### 7. Be careful with state

Decorator factories can capture configuration and state through closures.

This can be powerful but can also create unexpected shared state.

---

# 32. State and Closures

Consider:

```python
def counter_decorator():

    count = 0

    def decorator(func):

        def wrapper(*args, **kwargs):
            nonlocal count
            count += 1
            return func(*args, **kwargs)

        return wrapper

    return decorator
```

Usage:

```python
@counter_decorator()
def process():
    ...
```

Now `count` is retained by the closure.

This demonstrates how decorators can maintain state.

Conceptually:

```text
counter_decorator()
       │
       ├── count = 0
       │
       ▼
    decorator
       │
       ▼
    wrapper
       │
       └── remembers count
```

This becomes important when thinking about:

* Thread safety
* Concurrency
* Memory lifetime
* Shared state

---

# 33. Decorator Arguments vs Function Arguments

This distinction should be automatic in your mind.

Given:

```python
@retry(max_attempts=3)
def fetch_user(user_id, timeout):
    ...
```

There are **three levels**:

### Level 1 — Decorator configuration

```python
max_attempts=3
```

### Level 2 — Original function

```python
fetch_user
```

### Level 3 — Runtime function arguments

```python
user_id
timeout
```

Diagram:

```text
retry(max_attempts=3)
        │
        ▼
    decorator
        │
        ▼
 fetch_user
        │
        ▼
fetch_user(user_id, timeout)
```

This separation is one of the most important concepts in decorator factories.

---

# 34. Interview Questions

### Basic

1. What is a decorator with arguments?
2. What is a decorator factory?
3. Why do we need an extra function layer?
4. What is the difference between `@decorator` and `@decorator()`?
5. What does `@decorator(args)` mean internally?

### Execution

6. Explain:

```python
@repeat(3)
def greet():
    ...
```

7. What does this become internally?

```python
greet = repeat(3)(greet)
```

8. When does `repeat(3)` execute?
9. When does `decorator(greet)` execute?
10. When does `wrapper()` execute?

### Closures

11. How does a decorator factory use closures?
12. How does the wrapper access decorator configuration?
13. Why doesn't `count` disappear after `repeat()` returns?
14. What variables are captured by the wrapper?

### Advanced

15. How would you implement `@retry(max_attempts=3)`?
16. How would you implement `@cache(ttl=60)`?
17. How would you implement `@requires_role("admin")`?
18. What is the difference between decorator configuration and runtime arguments?
19. Why should `*args` and `**kwargs` usually be used in the wrapper?
20. Why should `functools.wraps` be used?
21. Can a decorator factory maintain state?
22. What problems can closure-based state cause in concurrent applications?
23. How would you test a decorator with arguments?
24. How would you debug a decorator factory with multiple layers?

---

# 35. Senior-Level Mental Model

Do not memorize this:

```python
def decorator(x):
    def inner(func):
        ...
```

Instead, understand the **three-stage transformation**.

```text
             STAGE 1
       Configure behavior
              │
              ▼
       decorator(config)
              │
              │ returns
              ▼
             STAGE 2
       Receive the function
              │
              ▼
       decorator(func)
              │
              │ returns
              ▼
             STAGE 3
       Execute decorated call
              │
              ▼
       wrapper(*args, **kwargs)
              │
              ▼
       original function
```

The most important equation:

```python
@decorator(config)
def function(...):
    ...
```

means:

```python
function = decorator(config)(function)
```

Compare:

```python
@decorator
```

with:

```python
@decorator(config)
```

### Without configuration

```text
decorator(function)
```

### With configuration

```text
decorator(config)(function)
```

That's the entire conceptual difference.

---

# 36. Final Mental Model

```text
                         @retry(3)
                            │
                            ▼
                    retry(max_attempts=3)
                            │
                            │
                     configuration
                            │
                            ▼
                       decorator
                            │
                            │ receives
                            ▼
                      original function
                            │
                            ▼
                         wrapper
                            │
                  receives runtime args
                            │
                            ▼
                   *args / **kwargs
                            │
                            ▼
                    original function
                            │
                            ▼
                         result
```

Think of the layers as:

```text
┌──────────────────────────────────┐
│  1. FACTORY                      │
│                                  │
│  retry(3)                         │
│  "How should the decorator work?"│
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  2. DECORATOR                    │
│                                  │
│  decorator(func)                 │
│  "Which function should I wrap?" │
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  3. WRAPPER                      │
│                                  │
│  wrapper(*args, **kwargs)        │
│  "The function was called."      │
└───────────────┬──────────────────┘
                │
                ▼
┌──────────────────────────────────┐
│  4. ORIGINAL FUNCTION            │
│                                  │
│  func(*args, **kwargs)           │
└──────────────────────────────────┘
```

> **Key takeaway:**
> A decorator with arguments is not simply a decorator that receives extra arguments. It is usually a **decorator factory**: the outer function receives configuration and returns a decorator; that decorator receives the original function and returns a wrapper. The fundamental transformation is:
>
> ```python
> @decorator(config)
> def function(...):
>     ...
> ```
>
> becomes:
>
> ```python
> function = decorator(config)(function)
> ```
>
> The configuration is captured through a **closure**, while runtime arguments are normally forwarded through `*args` and `**kwargs`.

---


