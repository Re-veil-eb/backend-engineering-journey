# `04-multiple-stacked-decorators.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, closures, decorators, `*args/**kwargs`, decorators with arguments
> **Core idea:** Multiple decorators wrap a function in layers. Decorators are **applied bottom-to-top**, while function calls flow through the **outermost wrapper to the innermost wrapper**.

---

# 1. What is it?

Python allows multiple decorators to be applied to a single function.

Example:

```python
@decorator1
@decorator2
def greet():
    print("Hello")
```

This means the function is wrapped by more than one decorator.

Conceptually:

```python
greet = decorator1(decorator2(greet))
```

This creates multiple wrapper layers around the original function.

---

# 2. Why does it exist?

In real applications, a function may require multiple cross-cutting behaviors.

For example:

```text
Authentication
      ↓
Authorization
      ↓
Logging
      ↓
Caching
      ↓
Business Logic
```

Instead of writing everything inside one function:

```python
def get_user():
    # authentication
    # authorization
    # logging
    # caching
    # business logic
```

We can separate responsibilities:

```python
@authenticate
@authorize
@log
@cache
def get_user():
    # business logic
```

This improves:

* Separation of concerns
* Reusability
* Maintainability
* Testability
* Readability

---

# 3. Basic Example

```python
def decorator1(func):

    def wrapper():
        print("Decorator 1 - Before")

        func()

        print("Decorator 1 - After")

    return wrapper


def decorator2(func):

    def wrapper():
        print("Decorator 2 - Before")

        func()

        print("Decorator 2 - After")

    return wrapper
```

Usage:

```python
@decorator1
@decorator2
def greet():
    print("Hello")
```

Call:

```python
greet()
```

Output:

```text
Decorator 1 - Before
Decorator 2 - Before
Hello
Decorator 2 - After
Decorator 1 - After
```

---

# 4. How Does It Work Internally?

This:

```python
@decorator1
@decorator2
def greet():
    print("Hello")
```

is equivalent to:

```python
def greet():
    print("Hello")

greet = decorator1(decorator2(greet))
```

Let's break it down.

### Step 1

Original function:

```text
greet
```

### Step 2

Apply the bottom decorator first:

```python
decorator2(greet)
```

This returns:

```text
wrapper2
```

### Step 3

Apply the top decorator:

```python
decorator1(wrapper2)
```

This returns:

```text
wrapper1
```

### Final assignment

```text
greet = wrapper1
```

---

# 5. The Most Important Rule

## Decorators are applied from bottom to top.

Given:

```python
@A
@B
@C
def function():
    ...
```

Python transforms it into:

```python
function = A(B(C(function)))
```

Therefore:

```text
Application Order:

C
↓
B
↓
A
```

But when the function is called:

```python
function()
```

execution enters:

```text
A wrapper
↓
B wrapper
↓
C wrapper
↓
Original function
```

Then returns outward:

```text
Original function
↑
C wrapper
↑
B wrapper
↑
A wrapper
```

---

# 6. Application Time vs Call Time

This is where many people get confused.

There are two separate phases.

## Phase 1: Decoration Time

Python applies decorators when the function is defined.

```python
@A
@B
def func():
    pass
```

Conceptually:

```text
Original Function
       ↓
B(original function)
       ↓
A(wrapper from B)
       ↓
Final Function
```

---

## Phase 2: Call Time

Later:

```python
func()
```

Execution flows:

```text
A wrapper starts
      ↓
B wrapper starts
      ↓
Original function
      ↓
B wrapper finishes
      ↓
A wrapper finishes
```

This distinction is critical.

---

# 7. Complete Execution Flow

Consider:

```python
def A(func):

    def wrapper():
        print("A Before")

        func()

        print("A After")

    return wrapper


def B(func):

    def wrapper():
        print("B Before")

        func()

        print("B After")

    return wrapper
```

Usage:

```python
@A
@B
def hello():
    print("Hello")
```

Internal transformation:

```python
hello = A(B(hello))
```

Memory structure:

```text
hello
  │
  ▼
A.wrapper
  │
  ▼
B.wrapper
  │
  ▼
Original hello
```

Call:

```python
hello()
```

Execution:

```text
A Before
    ↓
B Before
    ↓
Hello
    ↓
B After
    ↓
A After
```

---

# 8. Visual Wrapper Model

Think of decorators like nested boxes.

```text
┌─────────────────────────────┐
│         Decorator A         │
│                             │
│   ┌─────────────────────┐   │
│   │    Decorator B      │   │
│   │                     │   │
│   │   ┌─────────────┐   │   │
│   │   │  Function   │   │   │
│   │   └─────────────┘   │   │
│   │                     │   │
│   └─────────────────────┘   │
│                             │
└─────────────────────────────┘
```

When calling the function:

```text
Enter A
   ↓
Enter B
   ↓
Execute Function
   ↓
Exit B
   ↓
Exit A
```

---

# 9. Before and After Behavior

Decorators commonly execute code before and after the original function.

Example:

```python
def logging(func):

    def wrapper():
        print("Logging started")

        result = func()

        print("Logging finished")

        return result

    return wrapper


def timing(func):

    def wrapper():
        print("Timer started")

        result = func()

        print("Timer finished")

        return result

    return wrapper
```

Usage:

```python
@logging
@timing
def process():
    print("Processing data")
```

Call:

```python
process()
```

Output:

```text
Logging started
Timer started
Processing data
Timer finished
Logging finished
```

The outer decorator controls the outer execution layer.

---

# 10. Return Values With Multiple Decorators

Every wrapper should usually return the result.

Example:

```python
def A(func):

    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result

    return wrapper


def B(func):

    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result

    return wrapper
```

Function:

```python
@A
@B
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
A.wrapper
     ↓
B.wrapper
     ↓
Original add
     ↓
30
     ↓
B returns 30
     ↓
A returns 30
     ↓
Caller receives 30
```

---

# 11. What Happens if One Decorator Doesn't Return?

Consider:

```python
def A(func):

    def wrapper():
        return func()

    return wrapper


def B(func):

    def wrapper():
        func()   # No return

    return wrapper
```

Usage:

```python
@A
@B
def get_data():
    return "Data"
```

Call:

```python
print(get_data())
```

Execution:

```text
Original function returns "Data"
        ↓
B.wrapper does not return it
        ↓
B.wrapper returns None
        ↓
A receives None
        ↓
Final result = None
```

Output:

```text
None
```

A missing `return` in any wrapper can break the entire chain.

---

# 12. Arguments With Multiple Decorators

Production decorators should generally support arguments.

```python
from functools import wraps


def A(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("A Before")

        result = func(*args, **kwargs)

        print("A After")

        return result

    return wrapper


def B(func):

    @wraps(func)
    def wrapper(*args, **kwargs):

        print("B Before")

        result = func(*args, **kwargs)

        print("B After")

        return result

    return wrapper
```

Usage:

```python
@A
@B
def add(a, b):
    return a + b
```

Call:

```python
add(10, 20)
```

Arguments flow through every wrapper.

```text
Caller
  ↓
A(*args, **kwargs)
  ↓
B(*args, **kwargs)
  ↓
Original Function(*args, **kwargs)
```

---

# 13. Arguments Can Be Modified by Each Layer

Each decorator can modify arguments.

Example:

```python
def add_one(func):

    def wrapper(number):
        number += 1
        return func(number)

    return wrapper


def multiply_two(func):

    def wrapper(number):
        number *= 2
        return func(number)

    return wrapper
```

Usage:

```python
@add_one
@multiply_two
def show(number):
    return number
```

Internally:

```python
show = add_one(multiply_two(show))
```

Call:

```python
show(10)
```

Execution:

```text
10
 ↓
add_one
 ↓
11
 ↓
multiply_two
 ↓
22
 ↓
original show
```

Result:

```text
22
```

Decorator order directly affects the result.

---

# 14. Order Matters

Consider:

```python
@A
@B
def function():
    ...
```

This is not the same as:

```python
@B
@A
def function():
    ...
```

Because:

```python
@A
@B
```

means:

```python
function = A(B(function))
```

While:

```python
@B
@A
```

means:

```python
function = B(A(function))
```

The nesting structure changes.

---

# 15. Real Production Example

Imagine an API endpoint:

```python
@authenticate
@authorize
@rate_limit
@log_request
def get_account():
    ...
```

Conceptually:

```python
get_account = authenticate(
    authorize(
        rate_limit(
            log_request(
                get_account
            )
        )
    )
)
```

Call flow:

```text
Request
   ↓
authenticate
   ↓
authorize
   ↓
rate_limit
   ↓
log_request
   ↓
Business Logic
```

Return flow:

```text
Business Logic
   ↓
log_request
   ↓
rate_limit
   ↓
authorize
   ↓
authenticate
   ↓
Response
```

---

# 16. Exception Flow

Consider:

```python
def A(func):

    def wrapper():
        print("A Before")
        result = func()
        print("A After")
        return result

    return wrapper


def B(func):

    def wrapper():
        print("B Before")
        result = func()
        print("B After")
        return result

    return wrapper
```

If the original function raises an exception:

```python
@A
@B
def process():
    raise ValueError("Something failed")
```

Call flow:

```text
A Before
B Before
Exception occurs
```

Notice:

```text
B After → does not execute
A After → does not execute
```

Because execution is interrupted.

This matters in production.

---

# 17. Using `try/finally` in Decorators

If cleanup must always happen:

```python
def cleanup(func):

    def wrapper(*args, **kwargs):

        try:
            return func(*args, **kwargs)

        finally:
            print("Cleanup always runs")

    return wrapper
```

Even if the original function raises an exception:

```text
Function starts
    ↓
Exception occurs
    ↓
finally executes
    ↓
Exception propagates
```

This pattern is useful for:

* Database connections
* Locks
* Temporary resources
* Metrics
* Cleanup operations

---

# 18. `@wraps` With Multiple Decorators

Every decorator should ideally use:

```python
from functools import wraps
```

Example:

```python
def A(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

Why?

Without `@wraps`, metadata becomes confusing.

With multiple decorators:

```text
wrapper
  ↓
wrapper
  ↓
wrapper
  ↓
original function
```

Without proper metadata preservation:

```python
function.__name__
```

may return:

```text
wrapper
```

instead of:

```text
get_user
```

`@wraps` helps preserve metadata through the chain.

---

# 19. `__wrapped__` Chain

When using `@wraps`, Python creates access to the wrapped function.

Conceptually:

```text
Final Wrapper
     │
     └── __wrapped__
              │
              ▼
        Previous Wrapper
              │
              └── __wrapped__
                       │
                       ▼
                 Original Function
```

This is useful for:

* Debugging
* Introspection
* Frameworks
* Testing
* Documentation tools

---

# 20. Multiple Decorators With Arguments

Decorators can also be stacked when they have configuration.

Example:

```python
@retry(3)
@log("INFO")
def call_api():
    ...
```

Internally:

```python
call_api = retry(3)(log("INFO")(call_api))
```

Let's break it down.

### First

```python
log("INFO")
```

returns a decorator.

### Then

```python
log("INFO")(call_api)
```

returns a wrapper.

### Then

```python
retry(3)
```

returns another decorator.

### Finally

```python
retry_decorator(wrapper_from_log)
```

returns the outer wrapper.

---

# 21. Complex Execution Model

Given:

```python
@retry(3)
@log("INFO")
def call_api():
    print("Calling API")
```

Conceptually:

```text
Original call_api
       │
       ▼
log("INFO")
       │
       ▼
Log Decorator
       │
       ▼
Log Wrapper
       │
       ▼
retry(3)
       │
       ▼
Retry Decorator
       │
       ▼
Retry Wrapper
```

Final:

```text
call_api → Retry Wrapper
```

At runtime:

```text
call_api()
    ↓
Retry Wrapper
    ↓
Log Wrapper
    ↓
Original Function
```

---

# 22. Stacking Decorators With Different Responsibilities

A clean production design might look like:

```python
@authenticate
@validate_input
@rate_limit
@log_request
@measure_time
def create_order(data):
    ...
```

Each decorator has one responsibility.

```text
authenticate
      ↓
validate input
      ↓
rate limit
      ↓
log
      ↓
measure time
      ↓
business logic
```

This is an example of the **Single Responsibility Principle** applied to cross-cutting concerns.

---

# 23. Common Mistakes

## Mistake 1 — Thinking Decorators Execute Top-to-Bottom

Given:

```python
@A
@B
def func():
    ...
```

People may think:

```text
A(func)
↓
B(...)
```

Wrong.

Actual transformation:

```python
func = A(B(func))
```

So application starts with:

```text
B
↓
A
```

---

# 24. Mistake 2 — Confusing Application Order With Runtime Order

Remember:

```text
DECORATION TIME

Bottom → Top
```

But runtime enters:

```text
CALL TIME

Outer → Inner
```

For:

```python
@A
@B
def func():
```

```text
Decoration:
B → A

Runtime Entry:
A → B → Function

Runtime Exit:
Function → B → A
```

---

# 25. Mistake 3 — Forgetting Return Values

Every wrapper should usually:

```python
return func(*args, **kwargs)
```

Otherwise the return value may be lost.

---

# 26. Mistake 4 — Forgetting `*args` and `**kwargs`

A stacked decorator should generally support arbitrary arguments.

```python
@wraps(func)
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

# 27. Mistake 5 — Incorrect Decorator Order

Order is not cosmetic.

For example:

```python
@cache
@authenticate
def get_data():
    ...
```

may behave differently from:

```python
@authenticate
@cache
def get_data():
    ...
```

The order determines which behavior runs first.

In production systems, decorator ordering can affect:

* Security
* Performance
* Logging
* Caching
* Error handling
* Rate limiting

---

# 28. Production Perspective

Multiple decorators are commonly used for cross-cutting concerns.

Examples:

| Concern        | Possible Decorator |
| -------------- | ------------------ |
| Logging        | `@log`             |
| Authentication | `@authenticate`    |
| Authorization  | `@requires_role`   |
| Caching        | `@cache`           |
| Retry          | `@retry`           |
| Timing         | `@measure_time`    |
| Validation     | `@validate`        |
| Rate limiting  | `@rate_limit`      |
| Transactions   | `@transaction`     |

Instead of mixing infrastructure concerns with business logic:

```python
def process_payment():
    # authentication
    # logging
    # retry
    # transaction
    # payment logic
```

We can write:

```python
@authenticate
@log
@retry(3)
@transaction
def process_payment():
    # payment logic
```

This keeps the business function focused.

---

# 29. Performance Consideration

Each decorator adds another function call layer.

```text
Caller
  ↓
Wrapper 1
  ↓
Wrapper 2
  ↓
Wrapper 3
  ↓
Original Function
```

Usually this overhead is small.

But in:

* Extremely high-frequency functions
* Tight computational loops
* Low-latency systems

many layers can add unnecessary overhead.

A senior engineer considers whether abstraction cost is justified.

---

# 30. Debugging Complexity

Multiple decorators can make debugging harder.

Instead of:

```text
caller → function
```

you get:

```text
caller
  ↓
wrapper
  ↓
wrapper
  ↓
wrapper
  ↓
function
```

This can affect:

* Stack traces
* Debugging
* Profiling
* Introspection

Using `@wraps` helps reduce confusion.

---

# 31. Important Relationships

Your learning progression:

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
Decorators
    ↓
Argument Forwarding
    ↓
Decorator Factory
    ↓
Multiple Decorators
```

Multiple decorators combine everything you learned:

```text
Higher-Order Functions
        +
Nested Functions
        +
Closures
        +
Wrappers
        +
Function References
        +
*args / **kwargs
```

---

# 32. Interview Questions

### Basic

1. What are stacked decorators?
2. Can Python apply multiple decorators to one function?
3. In what order are decorators applied?
4. What does this mean internally?

```python
@A
@B
def func():
    pass
```

### Execution

5. Explain:

```python
func = A(B(func))
```

6. Which decorator is applied first?
7. Which wrapper executes first?
8. Explain the difference between decoration time and call time.
9. Why does execution appear to enter top-to-bottom but decorators are applied bottom-to-top?

### Advanced

10. What happens if one decorator doesn't return the function result?
11. What happens if the original function raises an exception?
12. How can cleanup be guaranteed?
13. Why should every decorator use `@wraps`?
14. What happens to `__wrapped__` with multiple decorators?
15. How do stacked decorators affect debugging?
16. How does decorator order affect security and caching?
17. Can decorators with arguments also be stacked?
18. Explain:

```python
@retry(3)
@log("INFO")
def call_api():
    ...
```

19. How would you debug multiple decorator layers?
20. What performance impact can multiple decorators have?

---

# 33. Senior-Level Mental Model

Do not think of decorators as annotations written above a function.

Think of them as **nested function transformations**.

Given:

```python
@A
@B
@C
def func():
    ...
```

Mentally translate immediately:

```python
func = A(B(C(func)))
```

Then visualize:

```text
A
│
└── B
    │
    └── C
        │
        └── Original Function
```

At runtime:

```text
ENTER
A
↓
B
↓
C
↓
FUNCTION
↑
C
↑
B
↑
A
EXIT
```

---

# 34. Final Mental Model

There are three things to always separate.

## 1. Decorator Application

```text
Bottom → Top
```

For:

```python
@A
@B
@C
```

Application:

```text
C → B → A
```

---

## 2. Wrapper Structure

```text
A(B(C(function)))
```

So:

```text
A is outermost
C is closest to original function
```

---

## 3. Runtime Execution

```text
Enter:

A → B → C → Function

Return:

Function → C → B → A
```

Complete visualization:

```text
                     CALLER
                        │
                        ▼
              ┌─────────────────┐
              │   A Wrapper     │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │   B Wrapper     │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │   C Wrapper     │
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Original Func   │
              └─────────────────┘
```

> **Key takeaway:**
>
> Multiple decorators create nested wrapper layers around a function.
>
> ```python
> @A
> @B
> @C
> def func():
>     ...
> ```
>
> is equivalent to:
>
> ```python
> func = A(B(C(func)))
> ```
>
> **Decorators are applied bottom-to-top, but runtime execution enters from the outermost wrapper and moves inward toward the original function.**

---

