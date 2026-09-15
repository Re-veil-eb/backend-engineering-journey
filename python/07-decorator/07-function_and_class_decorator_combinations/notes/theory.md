# `07-class-decorated-with-class.md`

> **Learning level:** Intermediate → Senior
> **Prerequisites:** Functions, `*args/**kwargs`, LEGB, nested functions, closures, higher-order functions, decorators, decorator factories, `__call__`, class-based decorators
> **Core idea:** A **class is itself an object**, and a class can be passed to a decorator just like a function. A class-based decorator can therefore receive a class, modify/replace it, and return another callable object.

---

# 1. What is it?

Until now, we decorated functions:

```python
@Logger
def process():
    pass
```

But Python allows us to decorate a **class**:

```python
@Logger
class PaymentService:
    pass
```

The important point is:

> A decorator does not fundamentally care whether the target is a function or a class. It receives an object and returns an object.

The transformation is:

```python
@Logger
class PaymentService:
    pass
```

equivalent to:

```python
class PaymentService:
    pass

PaymentService = Logger(PaymentService)
```

Notice:

```text
Logger()
   ↓
receives PaymentService class
   ↓
returns something
   ↓
PaymentService name now refers to returned object
```

This is exactly the same decorator protocol you already learned.

---

# 2. Why Does It Exist?

Class decorators are useful when we want to apply behavior or configuration to an **entire class**.

Instead of modifying every method individually:

```python
class PaymentService:

    def method1(self):
        ...

    def method2(self):
        ...

    def method3(self):
        ...
```

we can apply behavior at the class level:

```python
@some_decorator
class PaymentService:
    ...
```

Potential use cases include:

```text
Registration
Configuration
Validation
Logging
Dependency injection
ORM configuration
Plugin registration
Caching
Metadata attachment
Automatic method modification
Framework integration
```

A senior engineer should recognize this pattern as:

> **Class-level metaprogramming.**

---

# 3. How Does It Work Internally?

Consider:

```python
def register(cls):
    print(f"Registering {cls.__name__}")
    return cls


@register
class PaymentService:
    pass
```

Python performs:

```python
class PaymentService:
    pass

PaymentService = register(PaymentService)
```

The class object is passed into `register()`.

Therefore:

```text
PaymentService class
        ↓
register(PaymentService)
        ↓
returns class
        ↓
PaymentService
```

The decorator can:

1. Return the same class.
2. Modify the class and return it.
3. Create a subclass and return it.
4. Replace the class with another callable object.

---

# 4. Core Concept — Classes Are Objects

This is one of the most important Python concepts behind this topic.

When you write:

```python
class PaymentService:
    pass
```

Python creates a **class object**.

Conceptually:

```text
class statement
      ↓
creates class object
      ↓
stores it in PaymentService
```

Therefore:

```python
PaymentService
```

is a value that can be:

```python
passed
returned
stored
modified
decorated
```

Just like other Python objects.

This is why:

```python
@decorator
class MyClass:
    ...
```

is possible.

---

# 5. Basic Class Decorator

Let's create a simple class decorator.

```python
def register(cls):

    print(f"Registering {cls.__name__}")

    return cls
```

Use it:

```python
@register
class PaymentService:

    def process_payment(self):
        print("Payment processed")
```

Equivalent transformation:

```python
class PaymentService:

    def process_payment(self):
        print("Payment processed")


PaymentService = register(PaymentService)
```

Output during class definition:

```text
Registering PaymentService
```

---

# 6. Execution Flow

Consider:

```python
@register
class PaymentService:
    pass
```

The execution flow is:

```text
Python executes class body
        ↓
Creates PaymentService class object
        ↓
Passes class to register()
        ↓
register(PaymentService)
        ↓
Decorator returns class
        ↓
PaymentService name points to returned object
```

Important:

> The decorator executes **after the class object has been created**, not before.

---

# 7. Class Decoration Time vs Object Creation Time

This is a common source of confusion.

Consider:

```python
@register
class PaymentService:

    def __init__(self):
        print("Object created")
```

There are two different events.

### Class Decoration

```python
@register
class PaymentService:
```

runs:

```python
register(PaymentService)
```

This happens when Python defines the class.

### Object Creation

Later:

```python
service = PaymentService()
```

runs:

```text
PaymentService()
    ↓
__init__()
```

Therefore:

```text
Class Definition
      ↓
Class Created
      ↓
Decorator Executes
      ↓
Class Name Updated
      ↓
Later...
      ↓
Object Created
      ↓
__init__()
```

These are different lifecycle stages.

---

# 8. Class Decorator Can Modify the Class

A decorator can add attributes to the class.

```python
def add_metadata(cls):

    cls.service_type = "payment"

    return cls
```

Usage:

```python
@add_metadata
class PaymentService:
    pass
```

Now:

```python
print(PaymentService.service_type)
```

Output:

```text
payment
```

The decorator changed the class object.

---

# 9. Adding Methods Dynamically

A class decorator can also add methods.

```python
def add_health_check(cls):

    def health_check(self):
        return "Service is healthy"

    cls.health_check = health_check

    return cls
```

Usage:

```python
@add_health_check
class PaymentService:
    pass
```

Now:

```python
service = PaymentService()

print(service.health_check())
```

Output:

```text
Service is healthy
```

The decorator dynamically added:

```python
health_check
```

to the class.

---

# 10. Class Decorator That Registers Classes

A realistic use case is registration.

```python
registry = {}


def register(cls):

    registry[cls.__name__] = cls

    return cls
```

Usage:

```python
@register
class PaymentService:
    pass


@register
class UserService:
    pass
```

Now:

```python
print(registry)
```

Conceptually:

```text
registry
│
├── PaymentService → PaymentService class
│
└── UserService    → UserService class
```

This pattern appears in:

```text
Plugin systems
Command systems
Dependency injection
Serialization frameworks
Web frameworks
Task registration
Event handlers
```

---

# 11. Class Decorator Can Validate a Class

Suppose every service must contain a method:

```python
process
```

We can enforce it.

```python
def validate_service(cls):

    if not hasattr(cls, "process"):
        raise TypeError(
            f"{cls.__name__} must implement process()"
        )

    return cls
```

Usage:

```python
@validate_service
class PaymentService:

    def process(self):
        print("Processing payment")
```

This succeeds.

But:

```python
@validate_service
class UserService:
    pass
```

raises an error during class definition.

This can enforce architectural rules early.

---

# 12. Class Decorator vs Class-Based Decorator

Be very careful with terminology.

These are different concepts.

## Class Decorator

A decorator is applied **to a class**:

```python
@register
class PaymentService:
    pass
```

Here:

```text
Target = Class
```

---

## Class-Based Decorator

A **class is used as the decorator**:

```python
@Logger
def process():
    pass
```

Here:

```text
Decorator = Class
Target = Function
```

These are different.

```text
Class Decorator
────────────────
Decorator → Function/Class
Target    → Class


Class-Based Decorator
─────────────────────
Decorator → Class
Target    → Function/Class
```

And they can be combined.

---

# 13. Class Used as Decorator for a Class

Now combine the two concepts.

```python
class Logger:

    def __init__(self, target):
        self.target = target

    def __call__(self, *args, **kwargs):
        print("Creating object")
        return self.target(*args, **kwargs)
```

Then:

```python
@Logger
class PaymentService:

    def __init__(self):
        print("PaymentService initialized")
```

Python transforms:

```python
PaymentService = Logger(PaymentService)
```

Now `PaymentService` refers to a **Logger object**.

Later:

```python
service = PaymentService()
```

actually means:

```python
service = Logger_instance()
```

which invokes:

```python
Logger.__call__()
```

and inside:

```python
self.target()
```

creates the actual `PaymentService` object.

Flow:

```text
PaymentService()
      ↓
Logger.__call__()
      ↓
self.target()
      ↓
Original PaymentService class
      ↓
PaymentService instance
```

This is a very powerful pattern.

---

# 14. The Identity Changes

Before decoration:

```text
PaymentService
      ↓
PaymentService class
```

After:

```python
@Logger
class PaymentService:
    ...
```

the name becomes:

```text
PaymentService
      ↓
Logger instance
      ↓
self.target
      ↓
Original PaymentService class
```

This is a critical mental model.

The identifier:

```python
PaymentService
```

doesn't necessarily continue referring directly to the original class.

---

# 15. Why This Can Be Dangerous

Suppose:

```python
@Logger
class PaymentService:
    ...
```

and `Logger` returns a completely different object.

Then:

```python
PaymentService.__name__
```

may no longer behave as expected.

Also:

```python
isinstance(service, PaymentService)
```

can become problematic if `PaymentService` no longer refers to a class.

For example:

```text
PaymentService
      ↓
Logger instance
```

rather than:

```text
PaymentService
      ↓
class
```

Therefore:

> Replacing a class with a non-class object can break assumptions made by frameworks, type checkers, introspection tools, and application code.

---

# 16. Safer Pattern — Modify and Return the Class

Often the cleaner approach is:

```python
def add_metadata(cls):

    cls.version = "1.0"

    return cls
```

Then:

```python
@add_metadata
class PaymentService:
    pass
```

The class remains a class:

```text
PaymentService
      ↓
same class object
```

but it has been enhanced.

This usually preserves compatibility better.

---

# 17. Class Decorator With Arguments

Just like function decorators, class decorators can have configuration.

Example:

```python
def service(name):

    def decorator(cls):

        cls.service_name = name

        return cls

    return decorator
```

Usage:

```python
@service("payment")
class PaymentService:
    pass
```

Equivalent to:

```python
PaymentService = service("payment")(PaymentService)
```

Execution:

```text
service("payment")
       ↓
returns decorator
       ↓
decorator(PaymentService)
       ↓
adds configuration
       ↓
returns PaymentService
```

This directly connects to your previous topic:

> **Decorator Factory**

---

# 18. Three Layers Again

With:

```python
@service("payment")
class PaymentService:
    pass
```

there are three conceptual stages:

```text
Configuration
     ↓
service("payment")
     ↓
Decorator
     ↓
PaymentService class
     ↓
Modified class
```

Compare with your earlier function decorator factory:

```text
retry(3)
    ↓
decorator
    ↓
function
    ↓
wrapper
```

Same fundamental decorator protocol.

---

# 19. Class Decorator That Wraps Methods

A class decorator can inspect and modify methods.

Example:

```python
def log_methods(cls):

    for name, value in cls.__dict__.items():

        if callable(value):

            original = value

            def wrapper(self, *args, **kwargs):
                print(f"Calling {name}")
                return original(self, *args, **kwargs)

            setattr(cls, name, wrapper)

    return cls
```

However, this example has an important closure problem involving loop variables.

Do **not** use this implementation blindly.

A safer factory approach is:

```python
def create_wrapper(method_name, method):

    def wrapper(self, *args, **kwargs):
        print(f"Calling {method_name}")
        return method(self, *args, **kwargs)

    return wrapper
```

Then:

```python
def log_methods(cls):

    for name, value in list(cls.__dict__.items()):

        if callable(value):
            setattr(
                cls,
                name,
                create_wrapper(name, value)
            )

    return cls
```

This demonstrates a powerful relationship:

```text
Class Decorator
      +
Function Factory
      +
Closure
      ↓
Dynamic method decoration
```

---

# 20. Why the Closure Factory Matters Here

Suppose we loop:

```python
for name, method in methods:
```

and create wrappers.

Each wrapper needs its own:

```text
name
method
```

A factory creates a separate closure environment for each method.

```text
create_wrapper("method_a", method_a)
       ↓
wrapper A
       ↓
remembers method_a


create_wrapper("method_b", method_b)
       ↓
wrapper B
       ↓
remembers method_b
```

This is a direct application of your previous learning.

---

# 21. Important Relationships

Your concepts are now connecting together:

```text
Functions
    ↓
Functions are Objects
    ↓
Classes are Objects
    ↓
Higher-Order Functions
    ↓
Nested Functions
    ↓
Closures
    ↓
Decorators
    ↓
Decorator Factories
    ↓
Class-Based Decorators
    ↓
Class Decorators
    ↓
Dynamic Class Modification
```

This is where Python starts becoming much more powerful than just:

```text
"write functions and classes"
```

You are learning how Python's object model works.

---

# 22. Production Perspective

Class decorators appear in real systems when framework-level behavior needs to be applied consistently.

Typical patterns:

### Registration

```python
@register
class PaymentProcessor:
    ...
```

### Configuration

```python
@service(name="payment")
class PaymentService:
    ...
```

### Validation

```python
@validate
class PaymentService:
    ...
```

### Plugin systems

```python
@plugin
class StripeProcessor:
    ...
```

### Dependency injection

```python
@injectable
class PaymentService:
    ...
```

The exact decorators depend on the framework or architecture.

The underlying Python mechanism remains:

```text
Create target
      ↓
Pass target to decorator
      ↓
Decorator modifies/replaces target
      ↓
Return resulting object
```

---

# 23. Common Mistakes

## Mistake 1 — Confusing Class Decorator With Class-Based Decorator

Remember:

```python
@Logger
def process():
    ...
```

means:

```text
Class used as decorator
```

while:

```python
@register
class PaymentService:
    ...
```

means:

```text
Decorator applied to a class
```

---

# 24. Mistake 2 — Accidentally Replacing the Class

This:

```python
def decorator(cls):

    return SomeObject()
```

means:

```python
@decorator
class PaymentService:
    pass
```

results in:

```python
PaymentService = SomeObject()
```

You no longer have the original class bound to that name.

Always ask:

> **What exactly is my decorator returning?**

---

# 25. Mistake 3 — Breaking Class Identity

If the decorator replaces the class:

```python
PaymentService = SomeObject()
```

then code expecting:

```python
PaymentService
```

to be a class may fail.

This matters for:

```text
isinstance()
issubclass()
type checking
reflection
framework discovery
serialization
dependency injection
```

---

# 26. Mistake 4 — Modifying Classes Without Understanding Side Effects

This:

```python
cls.some_attribute = value
```

changes the class itself.

Every instance can potentially observe that attribute.

Therefore class-level changes affect all instances.

Think:

```text
Class attribute
      ↓
Potentially shared by all instances
```

---

# 27. Mistake 5 — Closure Late Binding

When dynamically creating method wrappers:

```python
for name, method in methods:
    ...
```

be careful about closures capturing loop variables.

Without understanding closure binding, multiple wrappers can unexpectedly refer to the same final values.

Your solution:

> Use a factory to create a separate closure for each iteration.

This connects directly to:

```text
Nested functions
       ↓
Closures
       ↓
Function factories
```

---

# 28. Interview Questions

### Basic

1. Can a class be decorated?
2. What happens internally when a class is decorated?
3. Are classes objects in Python?
4. What does:

```python
@register
class PaymentService:
    pass
```

become?

5. When does the class decorator execute?

### Intermediate

6. Can a class decorator modify the class?
7. Can it add methods?
8. Can it add attributes?
9. Can it replace the class?
10. What is the difference between modifying and replacing a class?

### Important

11. What is the difference between a **class decorator** and a **class-based decorator**?
12. Why can a class be passed to a decorator?
13. What happens to the class name after decoration?
14. Why can replacing a class with an arbitrary object cause problems?

### Advanced

15. How would you implement a class registration system?
16. How would you dynamically wrap all methods of a class?
17. Why might a function factory be needed when wrapping methods in a loop?
18. How does closure late binding affect dynamic class decoration?
19. How can class decorators be used for dependency injection?
20. What are the risks of modifying a class globally?

---

# 29. Senior-Level Mental Model

Don't think:

> "Decorators are only for functions."

Think:

> **A decorator is a transformation protocol: receive an object, transform or enhance it, and return an object.**

The target can be:

```text
Function
Class
Callable Object
```

The general pattern is:

```text
                 DECORATOR
                    │
                    ▼
              ┌───────────┐
              │  TARGET   │
              └───────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Function             Class
          │                   │
          ▼                   ▼
      New Function       Modified Class
          │                   │
          └─────────┬─────────┘
                    ▼
               Returned Object
```

---

# 30. The Most Important Transformation

Whenever you see:

```python
@decorator
class MyClass:
    ...
```

immediately translate it mentally to:

```python
class MyClass:
    ...

MyClass = decorator(MyClass)
```

Then ask:

### Question 1

What is passed?

```text
MyClass
```

### Question 2

What does the decorator return?

```text
Same class?
Modified class?
New class?
Callable object?
```

### Question 3

What does `MyClass` refer to after decoration?

This three-question model will prevent many advanced Python mistakes.

---

# 31. Final Mental Model

Suppose:

```python
@service("payment")
class PaymentService:

    def process(self):
        print("Payment")
```

Think:

```text
                CLASS BODY EXECUTES
                        │
                        ▼
              PaymentService class
                        │
                        ▼
              service("payment")
                        │
                        ▼
                  decorator
                        │
                        ▼
              decorator(PaymentService)
                        │
                        ▼
             Modified/Returned Class
                        │
                        ▼
                 PaymentService
```

And if the decorator returns the same class:

```text
PaymentService
      │
      ▼
Same class object
      +
Added configuration/behavior
```

If it returns another object:

```text
PaymentService
      │
      ▼
Different object
```

That distinction is **critical in production Python**.

---

# 32. Senior Connection to Everything You Learned

You can now see the complete progression:

```text
FUNCTION
   │
   ├── *args / **kwargs
   │
   ├── Scope / LEGB
   │
   ├── Nested Function
   │
   ├── Higher-Order Function
   │
   ├── Closure
   │
   └── Function Factory
             │
             ▼
         DECORATOR
             │
             ├── Arguments
             ├── Stacking
             └── Factory
                     │
                     ▼
              CLASS DECORATOR
                     │
                     ├── Class as Target
                     ├── Class as Decorator
                     └── __call__()
```

You are no longer learning isolated syntax.

You are building an understanding of:

> **Python's callable model + object model + runtime transformation model.**

That is the senior-level direction.

---

