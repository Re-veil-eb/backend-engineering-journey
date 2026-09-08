# `5-decorator-function-factory.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, higher-order functions, closures, decorators, decorator arguments, stacked decorators
> **Core idea:** A function factory creates and returns functions dynamically. A decorator factory specifically creates and returns decorators based on configuration.

---

# 1. What is it?

A **function factory** is a function that creates and returns another function.

Example:

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

Usage:

```python
double = create_multiplier(2)
triple = create_multiplier(3)

print(double(10))
print(triple(10))
```

Output:

```text
20
30
```

Here:

```text
create_multiplier()
        ↓
creates a new function
        ↓
returns that function
```

A **decorator function factory** is a specialized function factory.

It creates and returns a decorator.

```python
def retry(max_attempts):

    def decorator(func):
        ...

    return decorator
```

So:

```text
Function Factory
      ↓
Returns a function

Decorator Factory
      ↓
Returns a decorator function
```

---

# 2. Why does it exist?

Sometimes we need to dynamically create functions with different behavior.

For example:

```python
double = create_multiplier(2)
triple = create_multiplier(3)
quadruple = create_multiplier(4)
```

Instead of writing:

```python
def double(value):
    return value * 2


def triple(value):
    return value * 3


def quadruple(value):
    return value * 4
```

we create them dynamically.

The same concept is useful for decorators.

Example:

```python
@retry(3)
def call_api():
    ...
```

and:

```python
@retry(5)
def process_payment():
    ...
```

The same decorator factory creates differently configured decorators.

```text
retry(3)
    ↓
Decorator configured for 3 attempts

retry(5)
    ↓
Decorator configured for 5 attempts
```

---

# 3. How does it work internally?

Let's first understand a normal function factory.

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

When we call:

```python
double = create_multiplier(2)
```

Execution:

```text
create_multiplier(2)
        │
        ▼
number = 2
        │
        ▼
create multiply()
        │
        ▼
return multiply function object
        │
        ▼
double points to multiply()
```

Now:

```python
double(10)
```

The returned function still remembers:

```python
number = 2
```

because of a closure.

Execution:

```text
double(10)
    ↓
multiply(10)
    ↓
10 * 2
    ↓
20
```

---

# 4. Core Concepts

A function factory involves three major concepts:

```text
Nested Functions
        +
Returning Functions
        +
Closures
        =
Function Factory
```

Example:

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

### Nested Function

```python
def multiply(value):
```

is defined inside another function.

### Returning Function

```python
return multiply
```

returns the function object.

### Closure

The returned function remembers:

```python
number
```

even after `create_multiplier()` finishes.

---

# 5. Basic Function Factory Example

```python
def create_greeting(greeting):

    def greet(name):
        return f"{greeting}, {name}"

    return greet
```

Usage:

```python
hello = create_greeting("Hello")
good_morning = create_greeting("Good Morning")
```

Now:

```python
print(hello("Chinnu"))
print(good_morning("Chinnu"))
```

Output:

```text
Hello, Chinnu
Good Morning, Chinnu
```

Internally:

```text
create_greeting("Hello")
        ↓
Creates greet()
        ↓
greet remembers greeting="Hello"
        ↓
Returns greet
        ↓
hello → greet function
```

Another call:

```text
create_greeting("Good Morning")
        ↓
Creates another greet()
        ↓
greet remembers greeting="Good Morning"
        ↓
Returns another function
        ↓
good_morning → different function object
```

---

# 6. Each Factory Call Creates Independent State

Consider:

```python
def create_counter():

    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Usage:

```python
counter1 = create_counter()
counter2 = create_counter()
```

Now:

```python
counter1()
counter1()
counter2()
```

Output:

```text
1
2
1
```

Why?

Because each factory call creates a separate closure environment.

```text
counter1
   │
   └── count = 2


counter2
   │
   └── count = 1
```

This is an important senior-level concept.

---

# 7. Function Factory Execution Flow

Consider:

```python
def create_power(power):

    def calculate(number):
        return number ** power

    return calculate
```

Usage:

```python
square = create_power(2)
cube = create_power(3)
```

Internally:

### First Call

```text
create_power(2)
       ↓
power = 2
       ↓
Create calculate()
       ↓
Return calculate()
       ↓
square references calculate()
```

### Second Call

```text
create_power(3)
       ↓
power = 3
       ↓
Create another calculate()
       ↓
Return calculate()
       ↓
cube references calculate()
```

Then:

```python
square(5)
```

becomes:

```text
calculate(5)
    ↓
5 ** 2
    ↓
25
```

While:

```python
cube(5)
```

becomes:

```text
calculate(5)
    ↓
5 ** 3
    ↓
125
```

---

# 8. Function Factory vs Normal Function

## Normal Function

```python
def add(a, b):
    return a + b
```

The function directly performs work.

```text
Input
  ↓
Function
  ↓
Output
```

---

## Function Factory

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

The function creates another function.

```text
Configuration
      ↓
Factory
      ↓
Creates Function
      ↓
Returned Function
      ↓
Runtime Input
      ↓
Result
```

---

# 9. What is a Decorator Factory?

A decorator factory is a function that returns a decorator.

Example:

```python
def repeat(count):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for _ in range(count):
                func(*args, **kwargs)

        return wrapper

    return decorator
```

Here:

```text
repeat()
   ↓
returns decorator()
```

Then:

```text
decorator()
   ↓
returns wrapper()
```

The complete structure:

```text
Factory
  │
  ▼
Decorator
  │
  ▼
Wrapper
  │
  ▼
Original Function
```

---

# 10. Function Factory vs Decorator Factory

| Function Factory             | Decorator Factory                      |
| ---------------------------- | -------------------------------------- |
| Returns a function           | Returns a decorator                    |
| Creates behavior dynamically | Creates decorator behavior dynamically |
| Usually uses closures        | Usually uses closures                  |
| Can be used anywhere         | Used with decorators                   |
| Example: multiplier factory  | Example: retry factory                 |

### Function Factory

```python
def create_multiplier(x):

    def multiply(value):
        return value * x

    return multiply
```

### Decorator Factory

```python
def retry(attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):
            ...

        return wrapper

    return decorator
```

A decorator factory is simply a specialized form of a function factory.

---

# 11. Why Decorator Factories Need Three Layers

Consider:

```python
@retry(3)
def call_api():
    ...
```

Python conceptually does:

```python
call_api = retry(3)(call_api)
```

Let's break this down.

## Layer 1: Factory

```python
retry(3)
```

Receives configuration:

```text
attempts = 3
```

Returns:

```text
decorator
```

---

## Layer 2: Decorator

```python
decorator(call_api)
```

Receives:

```text
Original Function
```

Returns:

```text
wrapper
```

---

## Layer 3: Wrapper

```python
wrapper(*args, **kwargs)
```

Receives:

```text
Runtime Arguments
```

Calls:

```python
func(*args, **kwargs)
```

---

# 12. Three Different Data Flows

This is extremely important.

```python
@retry(max_attempts=3)
def fetch_user(user_id):
    ...
```

There are three different things.

## Configuration Data

```text
max_attempts = 3
```

Belongs to:

```text
Factory
```

---

## Function Object

```text
fetch_user
```

Belongs to:

```text
Decorator
```

---

## Runtime Data

```text
user_id
```

Belongs to:

```text
Wrapper → Original Function
```

Visual model:

```text
retry(max_attempts=3)
         │
         │ Configuration
         ▼
      decorator
         │
         │ Function Object
         ▼
       wrapper
         │
         │ Runtime Arguments
         ▼
   Original Function
```

---

# 13. Complete Decorator Factory Example

```python
from functools import wraps


def retry(max_attempts):

    def decorator(func):

        @wraps(func)
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
@retry(max_attempts=3)
def call_api():
    print("Calling API")
```

Conceptually:

```python
call_api = retry(max_attempts=3)(call_api)
```

---

# 14. Detailed Execution Timeline

Consider:

```python
@retry(3)
def process():
    print("Processing")
```

## Definition Time

### Step 1

Python defines:

```python
process
```

as a function object.

---

### Step 2

Python evaluates:

```python
retry(3)
```

This creates:

```text
attempts = 3
```

and returns:

```text
decorator
```

---

### Step 3

Python calls:

```python
decorator(process)
```

The decorator receives the original function.

---

### Step 4

The decorator creates:

```python
wrapper
```

The wrapper captures:

```text
attempts
func
```

---

### Step 5

Python assigns:

```text
process = wrapper
```

Now the name `process` no longer points directly to the original function.

It points to:

```text
wrapper
```

---

## Runtime

Later:

```python
process()
```

Execution:

```text
process()
    ↓
wrapper()
    ↓
retry logic
    ↓
original process()
```

---

# 15. Closure Relationship

Decorator factories depend on closures.

Consider:

```python
def retry(max_attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for _ in range(max_attempts):
                return func(*args, **kwargs)

        return wrapper

    return decorator
```

The wrapper accesses two external variables:

```text
max_attempts
func
```

Scope structure:

```text
retry()
│
├── max_attempts
│
└── decorator()
       │
       ├── func
       │
       └── wrapper()
              │
              ├── max_attempts
              └── func
```

Even after:

```text
retry()
```

and:

```text
decorator()
```

finish execution, the wrapper still remembers these values.

That is closure behavior.

---

# 16. Generic Function Factory

A function factory does not have to be a decorator.

Example:

```python
def create_validator(minimum):

    def validate(value):

        if value < minimum:
            raise ValueError(
                f"Value must be at least {minimum}"
            )

        return value

    return validate
```

Usage:

```python
age_validator = create_validator(18)
salary_validator = create_validator(30000)
```

Now:

```python
age_validator(22)
```

works.

But:

```python
age_validator(15)
```

raises an error.

Each returned function has different configuration.

---

# 17. Dynamic Behavior Creation

Factories allow us to create behavior dynamically.

```python
def create_logger(prefix):

    def log(message):
        print(f"[{prefix}] {message}")

    return log
```

Usage:

```python
info_log = create_logger("INFO")
error_log = create_logger("ERROR")
```

Now:

```python
info_log("Application started")
error_log("Database connection failed")
```

Output:

```text
[INFO] Application started
[ERROR] Database connection failed
```

Instead of creating separate functions manually:

```python
def info_log(message):
    ...


def error_log(message):
    ...
```

the factory generates configured functions.

---

# 18. Function Factory With Multiple Arguments

A factory can accept multiple configuration values.

```python
def create_range_validator(minimum, maximum):

    def validate(value):

        if not minimum <= value <= maximum:
            raise ValueError(
                f"Value must be between {minimum} and {maximum}"
            )

        return value

    return validate
```

Usage:

```python
percentage_validator = create_range_validator(0, 100)
age_validator = create_range_validator(18, 60)
```

Each returned function remembers different values.

---

# 19. Factory vs Calling the Returned Function

This distinction is important.

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

When you write:

```python
double = create_multiplier(2)
```

you are **not calling `multiply()`**.

You are receiving the function object.

```text
create_multiplier(2)
        ↓
returns multiply function
        ↓
double stores reference
```

Later:

```python
double(10)
```

calls the returned function.

This is equivalent conceptually to:

```text
Factory Call
      ↓
Get Function

Function Call
      ↓
Execute Function
```

---

# 20. Factory Call vs Function Call

Example:

```python
double = create_multiplier(2)
```

This executes:

```text
create_multiplier()
```

but not:

```text
multiply()
```

Then:

```python
double(10)
```

executes:

```text
multiply(10)
```

Timeline:

```text
create_multiplier(2)
        ↓
Create function
        ↓
Return function
        ↓
Store reference
        │
        │ Later
        ▼
double(10)
        ↓
Execute returned function
```

---

# 21. Production Perspective

Function factories are used when behavior must be configured dynamically.

Common examples:

### Logging

```text
create_logger(level)
```

### Validation

```text
create_validator(minimum)
```

### Serialization

```text
create_serializer(format)
```

### Database Access

```text
create_connection(environment)
```

### API Clients

```text
create_client(base_url)
```

### Decorators

```text
retry(max_attempts)
cache(ttl)
rate_limit(limit)
timeout(seconds)
```

The core idea is always:

```text
Configuration
      ↓
Factory
      ↓
Configured Callable
```

---

# 22. Common Mistakes

## Mistake 1 — Returning the Result Instead of the Function

Wrong:

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply(10)
```

This immediately executes:

```python
multiply(10)
```

and returns the result.

Correct:

```python
return multiply
```

This returns the function object.

---

## Mistake 2 — Confusing `func` and `func()`

```python
return func
```

means:

> Return the function object.

```python
return func()
```

means:

> Execute the function and return its result.

This distinction is fundamental.

---

## Mistake 3 — Expecting Variables to Disappear

Consider:

```python
def factory(value):

    def inner():
        return value

    return inner
```

After `factory()` returns, people may think:

```text
value is destroyed
```

But because `inner()` references `value`, Python preserves the necessary environment through closure behavior.

---

## Mistake 4 — Sharing State Accidentally

Consider:

```python
def create_counter():

    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

The returned function maintains state.

This can be useful.

But in production, shared state can cause issues with:

* Concurrency
* Threads
* Async applications
* Testing

Always understand the lifetime of captured variables.

---

# 23. Production Concerns

## Memory Lifetime

A closure keeps referenced objects alive.

Example:

```python
def factory(large_object):

    def inner():
        return large_object

    return inner
```

As long as:

```text
inner
```

exists, the captured object may remain in memory.

Be careful when factories capture:

* Large datasets
* Database connections
* File handles
* Network clients
* Request objects

---

## Mutable State

Consider:

```python
def create_storage():

    data = []

    def add(value):
        data.append(value)

    return add
```

The list persists across calls.

This may be intentional or accidental.

---

# 24. Important Relationships

Your learning path is strongly connected.

```text
Functions
   ↓
Functions are Objects
   ↓
Functions can be Returned
   ↓
Nested Functions
   ↓
Closures
   ↓
Function Factories
   ↓
Decorator Factories
```

A decorator factory is built from everything before it.

```text
Higher-Order Function
        +
Nested Function
        +
Closure
        +
Function Factory
        =
Decorator Factory
```

---

# 25. Function Factory vs Class Factory

Python can also create configured behavior using classes.

For example:

```python
class Multiplier:

    def __init__(self, number):
        self.number = number

    def __call__(self, value):
        return value * self.number
```

Usage:

```python
double = Multiplier(2)

print(double(10))
```

This behaves similarly to a function factory.

```text
Function Factory
       ↓
Returns configured function

Class + __call__
       ↓
Returns configured callable object
```

This connects directly to your upcoming topics:

```text
Class as Decorator
Class Decorated With Class
```

---

# 26. Interview Questions

### Basic

1. What is a function factory?
2. What does a function factory return?
3. How is a function factory different from a normal function?
4. What is the difference between `return func` and `return func()`?
5. Can a function factory return different functions?

### Closures

6. Why does the returned function remember factory arguments?
7. How does closure support function factories?
8. Does each factory call create independent closure state?
9. What happens to captured variables after the outer function returns?

### Decorators

10. What is a decorator factory?
11. How is a decorator factory related to a function factory?
12. Why does a decorator with arguments need multiple function layers?
13. Explain:

```python
function = decorator(config)(function)
```

14. What are the three levels of data in a decorator factory?

### Advanced

15. What memory issues can closures create?
16. How can mutable closure state create bugs?
17. How would you implement a configurable logger factory?
18. How would you create multiple independent counters?
19. When would you use a class instead of a function factory?
20. How does `__call__` relate to function factories?

---

# 27. Senior-Level Mental Model

Don't think:

> "A function factory is just a function returning another function."

Think:

> **A function factory is a runtime mechanism for generating configured callables by combining function creation with closure-based state.**

The architecture is:

```text
Configuration
      │
      ▼
Factory Function
      │
      ├── Creates Callable
      │
      └── Captures Configuration
              │
              ▼
       Returned Callable
              │
              ▼
          Runtime Input
              │
              ▼
            Result
```

For decorators:

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
Original Function
      │
      ▼
Wrapper
      │
      ▼
Runtime Execution
```

---

# 28. Final Mental Model

Let's combine everything.

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

Think of the layers:

```text
┌──────────────────────────────────────┐
│ FACTORY                              │
│ retry(max_attempts)                  │
│                                      │
│ Creates configured decorator         │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ DECORATOR                            │
│ decorator(func)                      │
│                                      │
│ Receives function to wrap            │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ WRAPPER                              │
│ wrapper(*args, **kwargs)             │
│                                      │
│ Executes runtime behavior            │
└──────────────────┬───────────────────┘
                   │
                   ▼
┌──────────────────────────────────────┐
│ ORIGINAL FUNCTION                    │
│ func(*args, **kwargs)                │
└──────────────────────────────────────┘
```

The general pattern:

```text
CONFIGURATION
      ↓
FACTORY
      ↓
CONFIGURED FUNCTION / DECORATOR
      ↓
RUNTIME INPUT
      ↓
EXECUTION
```

> **Key takeaway:**
>
> A function factory creates and returns functions dynamically. The returned function can remember configuration from the factory through closures. A decorator factory is a specialized function factory that returns a decorator, allowing reusable behavior to be configured before being applied to a function.

---

