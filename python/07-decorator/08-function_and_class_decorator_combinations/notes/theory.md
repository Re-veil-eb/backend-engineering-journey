# `08-function-and-class-decorator-combinations.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, `*args/**kwargs`, LEGB, nested functions, higher-order functions, closures, decorators, decorator factories, stacked decorators, class-based decorators, class decorators
> **Core idea:** Functions and classes are both objects, decorators transform objects, and the same decorator protocol can be combined across functions, classes, function-based decorators, and class-based decorators.

---

# 1. What is it?

Python decorators are not fundamentally about functions.

A decorator is a mechanism that:

```text
takes an object
     ↓
modifies / wraps / replaces it
     ↓
returns an object
```

The object can be:

```text
Function
Class
Callable Object
```

Therefore, we can combine:

```text
Function decorators
Class decorators
Class-based decorators
Decorator factories
Stacked decorators
```

Example:

```python
@logging
@authentication
def process_payment():
    pass
```

or:

```python
@register
@validate
class PaymentService:
    pass
```

or even:

```python
@ClassBasedDecorator
@function_decorator
def process():
    pass
```

The key is understanding **what object exists at each stage**.

---

# 2. Why Does It Exist?

Real production systems rarely have only one cross-cutting requirement.

A backend function may need:

```text
Authentication
Authorization
Logging
Metrics
Caching
Retry
Validation
Tracing
Rate limiting
Transaction management
```

Instead of putting everything inside the business logic:

```python
def process_payment():

    authenticate()

    authorize()

    log()

    metrics()

    validate()

    retry()

    actual_payment_logic()

    cache()

    trace()
```

we can separate concerns:

```python
@cache
@retry
@metrics
@authorization
@logging
def process_payment():
    ...
```

This allows business logic to remain focused.

---

# 3. How Does It Work Internally?

Consider:

```python
@decorator_a
@decorator_b
def process():
    pass
```

Python applies decorators from:

```text
bottom → top
```

Equivalent to:

```python
process = decorator_a(
    decorator_b(
        process
    )
)
```

Therefore:

```text
Original Function
      ↓
decorator_b
      ↓
decorator_a
      ↓
Final process
```

But when the function is called:

```text
Final process
      ↓
decorator_a wrapper
      ↓
decorator_b wrapper
      ↓
Original process
```

This distinction between **decoration order** and **execution order** is critical.

---

# 4. Core Concept — Decoration Order

Given:

```python
@A
@B
def process():
    pass
```

think:

```python
process = A(B(process))
```

Therefore:

```text
B is applied first
A is applied second
```

But at runtime:

```text
A wrapper
   ↓
B wrapper
   ↓
Original function
```

So the outermost decorator executes first.

---

# 5. Execution Flow

Example:

```python
def A(func):

    def wrapper():
        print("A before")

        func()

        print("A after")

    return wrapper


def B(func):

    def wrapper():
        print("B before")

        func()

        print("B after")

    return wrapper
```

Use:

```python
@A
@B
def process():
    print("Process")
```

Transformation:

```python
process = A(B(process))
```

Runtime:

```python
process()
```

Output:

```text
A before
B before
Process
B after
A after
```

Visualize:

```text
process()
   │
   ▼
A wrapper
   │
   ├── A before
   │
   ▼
B wrapper
   │
   ├── B before
   │
   ▼
Original process
   │
   └── Process
   │
   ▼
B after
   │
   ▼
A after
```

---

# 6. Why Stacked Decorators Behave Like Nested Functions

This:

```python
@A
@B
def process():
    pass
```

is effectively:

```python
process = A(B(process))
```

which resembles:

```python
A(
    B(
        process
    )
)
```

So mentally visualize decorators as layers:

```text
┌───────────────────────┐
│ A                     │
│   ┌────────────────┐  │
│   │ B              │  │
│   │   ┌─────────┐  │  │
│   │   │ process │  │  │
│   │   └─────────┘  │  │
│   └────────────────┘  │
└───────────────────────┘
```

The call enters from the outside and travels inward.

---

# 7. Function Decorator + Class-Based Decorator

A class can be used as a decorator for a function:

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

Now combine it with a function decorator:

```python
def authentication(func):

    def wrapper(*args, **kwargs):

        print("Checking authentication")

        return func(*args, **kwargs)

    return wrapper
```

Usage:

```python
@Logger
@authentication
def process_payment():
    print("Payment processing")
```

Transformation:

```python
process_payment = Logger(
    authentication(
        process_payment
    )
)
```

Runtime:

```text
process_payment()
       ↓
Logger.__call__()
       ↓
authentication.wrapper()
       ↓
Original process_payment()
```

---

# 8. Class Decorator + Class-Based Decorator

Now consider a class:

```python
class PaymentService:
    def process(self):
        print("Payment")
```

A class decorator can modify the class:

```python
def register(cls):

    print(f"Registering {cls.__name__}")

    return cls
```

And another decorator could be class-based:

```python
class Validate:

    def __init__(self, cls):
        self.cls = cls

    def __call__(self, *args, **kwargs):

        print("Validating object creation")

        return self.cls(*args, **kwargs)
```

Then:

```python
@register
@Validate
class PaymentService:
    pass
```

Equivalent:

```python
PaymentService = register(
    Validate(
        PaymentService
    )
)
```

The important thing is that **each decorator receives whatever object the previous decorator returned**.

---

# 9. Decorator Chain Mental Model

This is the most important model for this topic.

Suppose:

```python
@A
@B
@C
def process():
    pass
```

Think:

```text
Original
   ↓
C
   ↓
B
   ↓
A
   ↓
Final object
```

At runtime:

```text
Final object
   ↓
A
   ↓
B
   ↓
C
   ↓
Original
```

So:

> **The decorator stack is a chain of object transformations.**

---

# 10. Decorators Don't Know About Previous Layers

Suppose:

```python
@A
@B
def process():
    pass
```

When `A` runs, it doesn't necessarily know that `B` exists.

It simply receives:

```text
whatever B(process) returned
```

Therefore:

```python
process = A(B(process))
```

This gives us a powerful abstraction:

```text
Decorator A
    ↓
receives object
    ↓
returns object
    ↓
Decorator B / previous layer
```

Each layer only needs to obey the contract:

```text
Object → Object
```

---

# 11. Decorators Can Change Object Type

This is especially important with class-based decorators.

Suppose:

```python
@Logger
class PaymentService:
    pass
```

and:

```python
class Logger:

    def __init__(self, cls):
        self.cls = cls

    def __call__(self, *args, **kwargs):
        return self.cls(*args, **kwargs)
```

Before:

```text
PaymentService → Class
```

After:

```text
PaymentService → Logger instance
```

Therefore, decorators can change what a name refers to.

This is why senior engineers always ask:

> **What is the type of the object after decoration?**

---

# 12. Function Decorator That Returns a Class

A decorator technically doesn't have to return the same type.

For example:

```python
def replace(func):

    class Replacement:
        pass

    return Replacement
```

Then:

```python
@replace
def process():
    pass
```

Now:

```text
process
   ↓
Replacement class
```

This is legal Python.

But it may be a terrible design.

The lesson:

> **Legal Python does not automatically mean good production design.**

---

# 13. Decorator Factories in a Stack

You can combine decorator factories.

Example:

```python
def retry(attempts):

    def decorator(func):

        def wrapper(*args, **kwargs):

            for _ in range(attempts):
                try:
                    return func(*args, **kwargs)
                except Exception:
                    pass

        return wrapper

    return decorator
```

Another:

```python
def cache(ttl):

    def decorator(func):

        def wrapper(*args, **kwargs):
            print(f"Cache TTL = {ttl}")
            return func(*args, **kwargs)

        return wrapper

    return decorator
```

Now:

```python
@cache(ttl=60)
@retry(attempts=3)
def fetch_data():
    pass
```

Equivalent:

```python
fetch_data = cache(ttl=60)(
    retry(attempts=3)(
        fetch_data
    )
)
```

This is a direct combination of your previous topics.

---

# 14. Three Different Configuration Levels

In:

```python
@cache(ttl=60)
@retry(attempts=3)
def fetch_data(user_id):
    pass
```

there are three types of data.

### Decorator Configuration

```text
ttl = 60
attempts = 3
```

### Function Object

```text
fetch_data
```

### Runtime Data

```text
user_id
```

Flow:

```text
Configuration
      ↓
Decorator Factory
      ↓
Function
      ↓
Wrapper
      ↓
Runtime Arguments
```

This is exactly the mental model you built earlier.

---

# 15. Argument Flow Through Multiple Decorators

Consider:

```python
@A
@B
def process(x, y):
    return x + y
```

Call:

```python
process(10, 20)
```

Runtime:

```text
process(10, 20)
       ↓
A.wrapper(10, 20)
       ↓
B.wrapper(10, 20)
       ↓
original process(10, 20)
       ↓
30
```

Therefore every wrapper that sits in the chain must correctly forward:

```python
*args
**kwargs
```

Usually:

```python
return func(*args, **kwargs)
```

---

# 16. Return Value Flow

The return value travels outward.

```text
Original function
      ↓
return result
      ↓
B wrapper
      ↓
return result
      ↓
A wrapper
      ↓
return result
      ↓
Caller
```

If one wrapper forgets:

```python
return
```

the caller may receive:

```text
None
```

instead of the original result.

This is one of the most common decorator bugs.

---

# 17. Exception Flow

Exceptions also travel through the decorator chain.

Example:

```python
@logging
@retry
def process():
    raise ValueError("Failed")
```

Possible flow:

```text
logging wrapper
      ↓
retry wrapper
      ↓
process()
      ↓
Exception
      ↑
retry handles/re-raises
      ↑
logging sees exception
      ↑
caller
```

The exact behavior depends on where `try/except` exists.

Therefore decorator order can change exception semantics.

---

# 18. Production Perspective — Order Matters

Consider:

```python
@cache
@authentication
def get_user():
    ...
```

versus:

```python
@authentication
@cache
def get_user():
    ...
```

These are **not necessarily equivalent**.

First:

```text
cache
  ↓
authentication
  ↓
function
```

Second:

```text
authentication
  ↓
cache
  ↓
function
```

This can affect:

```text
Security
Performance
Correctness
Cache isolation
Logging
Metrics
Retry behavior
Transaction boundaries
```

A senior engineer never blindly stacks decorators.

They ask:

> **What should happen first?**

---

# 19. Example — Authentication and Logging

Suppose:

```python
@logging
@authentication
def delete_user():
    pass
```

Execution:

```text
logging
   ↓
authentication
   ↓
delete_user
```

This means logging sees the authentication attempt.

If reversed:

```python
@authentication
@logging
def delete_user():
    pass
```

then:

```text
authentication
   ↓
logging
   ↓
delete_user
```

Now unauthorized requests may never reach the logging wrapper.

This can change observability.

---

# 20. Example — Retry and Transaction

Imagine:

```python
@transaction
@retry(3)
def process_payment():
    ...
```

versus:

```python
@retry(3)
@transaction
def process_payment():
    ...
```

These can have completely different semantics.

First:

```text
Transaction
   ↓
Retry
   ↓
Payment
```

Second:

```text
Retry
   ↓
Transaction
   ↓
Payment
```

Potential difference:

```text
Retry entire transaction
```

versus:

```text
Retry inside transaction
```

In production financial systems, this distinction can be extremely important.

---

# 21. Class Decorators and Method Decorators Together

A class can itself be decorated while its methods are also decorated.

Example:

```python
def log_method(func):

    def wrapper(*args, **kwargs):

        print(f"Calling {func.__name__}")

        return func(*args, **kwargs)

    return wrapper
```

Then:

```python
def register(cls):

    print(f"Registering {cls.__name__}")

    return cls
```

Usage:

```python
@register
class PaymentService:

    @log_method
    def process_payment(self):
        print("Payment")
```

Execution:

```text
Class definition
      ↓
Method decorator applied
      ↓
PaymentService class created
      ↓
Class decorator applied
      ↓
PaymentService available
```

When:

```python
service.process_payment()
```

runs:

```text
service.process_payment()
          ↓
log_method.wrapper()
          ↓
original process_payment()
```

---

# 22. Important Execution Order — Class + Method

For:

```python
@class_decorator
class Service:

    @method_decorator
    def process(self):
        pass
```

think:

```text
1. Define method
2. Apply method decorator
3. Build class
4. Apply class decorator
```

So:

```text
method decoration
      ↓
class creation
      ↓
class decoration
```

This is an important Python lifecycle detail.

---

# 23. Combining Everything

Consider:

```python
@register
@Logger
class PaymentService:

    @metrics
    @authentication
    def process(self, amount):
        return amount
```

There are two separate decorator chains.

## Class chain

```text
PaymentService
     ↓
Logger
     ↓
register
```

Equivalent:

```python
PaymentService = register(
    Logger(
        PaymentService
    )
)
```

## Method chain

```text
process
   ↓
authentication
   ↓
metrics
```

Equivalent:

```python
process = metrics(
    authentication(
        process
    )
)
```

You must reason about these chains independently.

---

# 24. The Senior Debugging Question

When a decorated system behaves unexpectedly, don't immediately inspect the business logic.

First ask:

```text
What does this name point to now?
```

For example:

```python
PaymentService
```

might point to:

```text
Original class
```

or:

```text
Modified class
```

or:

```text
Decorator object
```

Similarly:

```python
process
```

might point to:

```text
Original function
```

or:

```text
wrapper A
```

or:

```text
wrapper B
```

or:

```text
Callable object
```

This mental model is extremely useful for debugging.

---

# 25. Common Mistakes

## Mistake 1 — Assuming Top-to-Bottom Execution

Wrong mental model:

```text
A
↓
B
↓
function
```

because it appears visually that way.

Correct decoration transformation:

```python
function = A(B(function))
```

Runtime enters:

```text
A
 ↓
B
 ↓
function
```

Remember the two different phases.

---

# 26. Mistake 2 — Ignoring Return Values

Every decorator layer must preserve the contract.

Bad:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        func(*args, **kwargs)

    return wrapper
```

Correct:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

---

# 27. Mistake 3 — Losing Arguments

Bad:

```python
def wrapper():
    return func()
```

Better:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

---

# 28. Mistake 4 — Ignoring Metadata

Repeated decorators can make debugging difficult.

Use:

```python
from functools import wraps
```

For function-based decorators:

```python
def decorator(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

This helps preserve function metadata.

---

# 29. Mistake 5 — Too Many Decorators

This:

```python
@a
@b
@c
@d
@e
@f
@function
```

may be technically valid but difficult to understand.

Too many layers can create:

```text
Debugging difficulty
Hidden control flow
Unexpected ordering
Performance overhead
Testing complexity
```

Senior engineers optimize for **clarity**, not cleverness.

---

# 30. Mistake 6 — Using Decorators for Everything

Not every reusable behavior needs a decorator.

Sometimes better options are:

```text
Explicit function call
Helper function
Class
Dependency injection
Middleware
Context manager
Service layer
Composition
```

Choose decorators when the behavior naturally surrounds or transforms another callable.

---

# 31. Important Relationships

You have now built almost the complete Python callable model:

```text
Functions
   ↓
Functions are Objects
   ↓
Higher-Order Functions
   ↓
Nested Functions
   ↓
Closures
   ↓
Function Factories
   ↓
Decorators
   ↓
Decorator Factories
   ↓
Stacked Decorators
   ↓
Classes as Callables
   ↓
Class-Based Decorators
   ↓
Classes as Decoration Targets
   ↓
Combined Decorator Chains
```

This is no longer just "decorator syntax."

It is understanding **object transformation and callable composition**.

---

# 32. Production Mental Model

Think of decorators as middleware around a callable.

For example:

```text
Incoming Request
       ↓
Authentication
       ↓
Authorization
       ↓
Logging
       ↓
Metrics
       ↓
Retry
       ↓
Business Logic
       ↓
Response
```

Each decorator adds one responsibility.

This is conceptually similar to middleware pipelines in backend systems.

---

# 33. Interview Questions

### Basic

1. What happens when multiple decorators are stacked?
2. Explain:

```python
@A
@B
def f():
    pass
```

3. What is the equivalent assignment?
4. Which decorator is applied first?
5. Which wrapper executes first?

### Intermediate

6. How do arguments travel through multiple decorators?
7. How does the return value travel through multiple decorators?
8. How do exceptions propagate through decorator layers?
9. Why does decorator order matter?
10. What happens if one decorator forgets to return the result?

### Advanced

11. Can a class be decorated by another decorator?
12. Can a class be used as a decorator?
13. Can a decorator change the type of the decorated object?
14. How would you debug a deeply decorated function?
15. Why is `functools.wraps` important?
16. What is the difference between decoration time and execution time?
17. How would you design a retry decorator for production?
18. How would you make a stateful class decorator thread-safe?
19. When should you avoid decorators?
20. How do decorators relate to middleware?

---

# 34. Senior-Level Mental Model

Forget the syntax temporarily.

Don't think:

```python
@something
```

Think:

```text
TRANSFORMATION
```

A decorator is:

```text
                    ┌──────────────┐
                    │  Decorator   │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │    Object    │
                    └──────┬───────┘
                           │
                           ▼
                    ┌──────────────┐
                    │ New Object   │
                    └──────────────┘
```

For stacked decorators:

```text
Original Object
      ↓
Decorator C
      ↓
Decorator B
      ↓
Decorator A
      ↓
Final Object
```

For execution:

```text
Final Object
      ↓
A behavior
      ↓
B behavior
      ↓
C behavior
      ↓
Original behavior
```

---

# 35. The Most Important Senior Rule

Whenever you see:

```python
@A
@B
@C
def function():
    pass
```

write this mentally:

```python
function = A(B(C(function)))
```

Then ask:

### 1. What does `C` receive?

```text
Original function
```

### 2. What does `B` receive?

```text
Whatever C returned
```

### 3. What does `A` receive?

```text
Whatever B returned
```

### 4. What does `function` finally reference?

```text
Whatever A returned
```

That is the entire decorator chain.

---

# 36. Final Mental Model

Your decorator knowledge can now be summarized as:

```text
                    PYTHON CALLABLE MODEL
                            │
             ┌──────────────┴──────────────┐
             │                             │
         Functions                      Classes
             │                             │
             └───────────┬─────────────────┘
                         │
                     Callables
                         │
                         ▼
                    Decorators
                         │
            ┌────────────┼────────────┐
            │            │            │
        Function      Factory      Class-based
        Decorator     Decorator     Decorator
            │            │            │
            └────────────┼────────────┘
                         │
                         ▼
                  Decorator Stacking
                         │
                         ▼
                 Object Transformation
                         │
                         ▼
                  Runtime Composition
```

The senior mental model is:

> **Decorators are a form of composition. Each decorator receives an object, adds or changes behavior, and returns another object. Stacking decorators creates a chain of transformations. The order of that chain determines runtime behavior.**

---

# 37. Final Production Checklist

Before using a decorator in production, ask:

```text
□ What object am I decorating?
□ What does the decorator return?
□ Does the returned object preserve the expected interface?
□ Are *args and **kwargs forwarded?
□ Is the return value preserved?
□ Are exceptions preserved or intentionally changed?
□ Is metadata preserved?
□ Does decorator order matter?
□ Does the decorator maintain state?
□ Is that state thread-safe?
□ Does it introduce performance overhead?
□ Is the behavior obvious to another engineer?
□ Would middleware/composition be clearer?
```

If you can answer all of these, you are thinking about decorators at a production level.

---

