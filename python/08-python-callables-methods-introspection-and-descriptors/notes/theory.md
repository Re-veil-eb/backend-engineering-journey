# `08-python-callables-methods-introspection-and-descriptors.md`

# Python Callables, `__call__`, `__getattr__`, Introspection, Descriptors & How Methods Work Internally

> **Level:** Intermediate → Senior  
> **Prerequisites:** Functions, `*args/**kwargs`, LEGB, nested functions, higher-order functions, closures, decorators, decorator factories, stacked decorators, class-based decorators, class decorators
>
> **Core idea:** Python's functions, methods, classes, and callable objects are all connected through the object model. Understanding `__call__`, attribute lookup, introspection, descriptors, and method binding explains what Python is actually doing underneath familiar syntax.

---

# 1. What is it?

This topic explains several mechanisms that appear separate but are actually deeply connected:

```text
__call__
__getattr__
__getattribute__
Introspection
Descriptors
Method binding
Bound methods
Unbound functions
staticmethod
classmethod
property
```

Together, they explain questions such as:

```python
obj.method()
```

Why does this work?

Why does:

```python
obj.method
```

behave differently from:

```python
Class.method
```

Why does:

```python
instance.attribute
```

sometimes call hidden Python machinery?

Why can an object be called like:

```python
obj()
```

even though it isn't technically a function?

Why does:

```python
@property
def name(self):
    ...
```

allow:

```python
obj.name
```

without:

```python
obj.name()
```

Why can decorators use:

```python
@SomeClass
```

?

And why can Python frameworks perform so much behavior simply by inspecting classes and objects?

The answer lies heavily in:

> **Python's object model + attribute lookup + descriptors + call protocol.**

---

# 2. Why does it exist?

Python is designed around objects.

Functions are objects.

Classes are objects.

Methods are objects.

Instances are objects.

Even classes themselves are created from other objects called metaclasses.

Python therefore needs a consistent mechanism for:

```text
Calling objects
Looking up attributes
Binding methods
Creating properties
Inspecting objects
Customizing behavior
```

Instead of hard-coding every possible behavior into the language syntax, Python exposes protocols such as:

```text
__call__
__getattribute__
__getattr__
__get__
__set__
__delete__
```

These protocols allow objects to participate in Python's runtime behavior.

---

# 3. How does it work internally?

Consider:

```python
obj.method()
```

At a beginner level, we say:

```text
"Call method"
```

At a deeper level, Python must perform several operations:

```text
obj
 ↓
attribute lookup
 ↓
find "method"
 ↓
descriptor machinery may execute
 ↓
function may become bound method
 ↓
bound method is called
 ↓
instance becomes self
 ↓
function executes
```

So:

```python
obj.method()
```

is not simply:

```text
find function → execute function
```

There is an important transformation happening:

```text
Class function
      ↓
descriptor protocol
      ↓
bound method
      ↓
call
```

This is one of the most important concepts in Python's object model.

---

# 4. Core Concept — Everything Is an Object

Consider:

```python
def greet():
    print("Hello")
```

`greet` is an object.

```python
print(type(greet))
```

Conceptually:

```text
<class 'function'>
```

Now:

```python
class Person:
    pass
```

`Person` is also an object.

```python
print(type(Person))
```

Conceptually:

```text
<class 'type'>
```

An instance:

```python
person = Person()
```

is another object.

So:

```text
greet
 ↓
function object


Person
 ↓
class object


person
 ↓
instance object
```

This object-centric model explains why Python can pass classes and functions around.

---

# 5. `__call__` — Making Objects Callable

Normally:

```python
def greet():
    print("Hello")

greet()
```

works because a function is callable.

But Python allows ordinary objects to become callable.

Example:

```python
class Greeter:

    def __call__(self):
        print("Hello")


g = Greeter()

g()
```

Output:

```text
Hello
```

Why?

Because:

```python
g()
```

causes Python to use the object's call protocol.

Conceptually:

```text
g()
 ↓
object is callable?
 ↓
__call__()
 ↓
execute
```

---

# 6. `__call__` Internally

When you write:

```python
g()
```

Python performs the call operation.

For a user-defined object, the object's class can provide:

```python
__call__
```

So:

```python
class Greeter:

    def __call__(self):
        print("Hello")
```

creates an object that behaves like a function.

Therefore:

```text
Function-like behavior
        ↓
does not necessarily require
        ↓
a function object
```

A callable object can provide function-like behavior through:

```python
__call__
```

---

# 7. Why `__call__` Matters for Decorators

This directly connects to your previous topic.

Remember:

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
def process():
    pass
```

becomes:

```python
process = Logger(process)
```

Now:

```python
process()
```

works because:

```text
process
 ↓
Logger instance
 ↓
__call__()
 ↓
original function
```

So your earlier class-based decorator knowledge is actually built directly on the `__call__` protocol.

---

# 8. Callable Objects

You can check whether an object can be called:

```python
callable(obj)
```

Example:

```python
class Service:

    def __call__(self):
        return "running"


service = Service()

print(callable(service))
```

Output:

```text
True
```

Compare:

```python
class Service:
    pass

service = Service()

print(callable(service))
```

Output:

```text
False
```

---

# 9. `callable()` Is Better Than Checking `__call__` Manually

Avoid:

```python
hasattr(obj, "__call__")
```

as your main test for callability.

Prefer:

```python
callable(obj)
```

because Python's callability is a runtime protocol.

Senior mental model:

> Ask Python whether the object is callable instead of making assumptions from its attributes.

---

# 10. `__call__` With Arguments

A callable object can behave almost exactly like a function.

```python
class Multiplier:

    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        return value * self.factor
```

Usage:

```python
double = Multiplier(2)

print(double(10))
```

Output:

```text
20
```

Here:

```text
double
 ↓
object with state
 ↓
__call__(10)
 ↓
20
```

This is one major advantage over a simple function:

> A callable object can carry state naturally.

---

# 11. Function vs Callable Object

Function:

```python
def multiply(value):
    return value * 2
```

Callable object:

```python
class Multiplier:

    def __init__(self, factor):
        self.factor = factor

    def __call__(self, value):
        return value * self.factor
```

The callable object can maintain:

```text
Configuration
State
Counters
Dependencies
Resources
Caches
Metrics
```

This is why class-based decorators can be useful.

---

# 12. `__getattribute__` — Attribute Lookup Entry Point

Now we move to a much deeper concept.

When you write:

```python
obj.name
```

Python has to find the attribute.

The process is associated with:

```python
__getattribute__
```

Every normal object attribute access goes through this mechanism.

Conceptually:

```text
obj.name
 ↓
obj.__getattribute__("name")
 ↓
attribute lookup machinery
 ↓
result
```

This is much deeper than simply checking:

```python
obj.__dict__
```

---

# 13. `__getattr__` vs `__getattribute__`

These are frequently confused.

## `__getattribute__`

Runs for essentially **every attribute access** on the object.

```python
class Demo:

    def __getattribute__(self, name):
        print("Looking for:", name)
        return object.__getattribute__(self, name)
```

Now:

```python
obj = Demo()

obj.value
```

causes the custom lookup hook to participate.

---

## `__getattr__`

Runs only when normal attribute lookup fails.

```python
class Demo:

    def __getattr__(self, name):
        return f"{name} does not exist"
```

Then:

```python
obj = Demo()

print(obj.xyz)
```

Output:

```text
xyz does not exist
```

Mental model:

```text
obj.attribute
      ↓
__getattribute__()
      ↓
Normal lookup succeeds?
      │
   ┌──┴───┐
  YES     NO
   │       │
   ▼       ▼
return   __getattr__()
```

This distinction is critical.

---

# 14. `__getattr__` Is a Fallback

Example:

```python
class Config:

    def __getattr__(self, name):
        return None
```

Then:

```python
config = Config()

print(config.database)
```

If `database` isn't found normally:

```text
database lookup
      ↓
not found
      ↓
__getattr__("database")
      ↓
None
```

This allows dynamic attributes.

---

# 15. Dangerous `__getattr__` Recursion

Bad:

```python
class Demo:

    def __getattr__(self, name):
        return self.__dict__[name]
```

Why can this be dangerous?

Because:

```python
self.__dict__
```

itself is an attribute access.

That can trigger lookup machinery again.

Safer low-level access can use:

```python
object.__getattribute__(self, "__dict__")
```

This avoids accidentally routing through your overridden logic.

Senior rule:

> When overriding attribute lookup, understand how to bypass your own override safely.

---

# 16. `__getattribute__` Can Easily Cause Infinite Recursion

Bad:

```python
class Demo:

    def __getattribute__(self, name):
        return self.name
```

Suppose:

```python
obj.value
```

Python enters:

```text
__getattribute__("value")
```

Inside it:

```python
self.name
```

which calls:

```text
__getattribute__("name")
```

which calls:

```text
__getattribute__("name")
```

and so on.

Result:

```text
RecursionError
```

Correct delegation:

```python
class Demo:

    def __getattribute__(self, name):
        return object.__getattribute__(self, name)
```

---

# 17. Attribute Lookup Is More Complicated Than `__dict__`

A beginner might think:

```python
obj.attribute
```

means:

```python
obj.__dict__["attribute"]
```

Not necessarily.

Python may search:

```text
Data descriptors
Instance dictionary
Non-data descriptors
Class attributes
Base classes
__getattr__ fallback
```

The exact lookup rules matter.

This leads directly to descriptors.

---

# 18. What Is a Descriptor?

A descriptor is an object that defines one or more of:

```python
__get__
__set__
__delete__
```

A descriptor controls attribute access.

Basic example:

```python
class Descriptor:

    def __get__(self, instance, owner):
        return "Hello"
```

Use it:

```python
class Person:

    name = Descriptor()
```

Then:

```python
p = Person()

print(p.name)
```

The descriptor's:

```python
__get__()
```

is invoked.

So:

```text
p.name
 ↓
descriptor
 ↓
__get__()
 ↓
result
```

---

# 19. Why Descriptors Exist

Descriptors allow Python to implement behavior behind ordinary attribute syntax.

They power important Python features:

```text
Methods
@property
staticmethod
classmethod
ORM fields
Validation frameworks
Lazy loading
Computed attributes
Dependency injection
Many framework APIs
```

This is one of Python's most important internal mechanisms.

---

# 20. The Descriptor Protocol

A descriptor can implement:

```python
__get__
__set__
__delete__
```

These correspond conceptually to:

```text
Read
Write
Delete
```

Example:

```python
class Descriptor:

    def __get__(self, instance, owner):
        ...

    def __set__(self, instance, value):
        ...

    def __delete__(self, instance):
        ...
```

You don't necessarily need all three.

---

# 21. Data Descriptor vs Non-Data Descriptor

This distinction is extremely important.

## Data Descriptor

Defines:

```python
__set__
```

or:

```python
__delete__
```

Usually together with:

```python
__get__
```

Example:

```python
class Descriptor:

    def __get__(self, instance, owner):
        ...

    def __set__(self, instance, value):
        ...
```

This is a data descriptor.

---

## Non-Data Descriptor

Defines only:

```python
__get__
```

Example:

```python
class Descriptor:

    def __get__(self, instance, owner):
        ...
```

This is a non-data descriptor.

---

# 22. Why Data vs Non-Data Matters

This affects lookup precedence.

Conceptually:

```text
Data descriptor
      ↓
Instance dictionary
      ↓
Non-data descriptor
      ↓
Class attribute
```

Therefore:

> A data descriptor can take priority over an instance attribute.

This is one of the most important descriptor rules.

---

# 23. Example — Data Descriptor Wins

```python
class Descriptor:

    def __get__(self, instance, owner):
        return "descriptor"

    def __set__(self, instance, value):
        print("setting:", value)


class Person:

    name = Descriptor()
```

Now:

```python
p = Person()

p.__dict__["name"] = "instance"

print(p.name)
```

The descriptor can still control access because it is a data descriptor.

Conceptually:

```text
p.name
 ↓
data descriptor found
 ↓
Descriptor.__get__()
```

The instance dictionary does not win.

---

# 24. Non-Data Descriptor Can Be Shadowed

Consider:

```python
class Descriptor:

    def __get__(self, instance, owner):
        return "descriptor"


class Person:

    name = Descriptor()
```

Because this descriptor only has:

```python
__get__
```

it is a non-data descriptor.

Now:

```python
p = Person()

p.__dict__["name"] = "instance"

print(p.name)
```

The instance value can win.

Conceptually:

```text
p.name
 ↓
no data descriptor
 ↓
instance dictionary
 ↓
"name" found
 ↓
"instance"
```

This difference is extremely important when debugging Python frameworks.

---

# 25. Functions Are Descriptors

This is the breakthrough concept for understanding methods.

When you define:

```python
class Person:

    def greet(self):
        print("Hello")
```

the function stored in the class is a descriptor.

Functions implement the descriptor protocol.

Conceptually:

```text
Person.__dict__["greet"]
        ↓
function object
        ↓
descriptor
```

This is why:

```python
person.greet
```

doesn't simply return the same function object stored in the class.

Python binds it.

---

# 26. How Methods Actually Work

Consider:

```python
class Person:

    def greet(self):
        print("Hello")


p = Person()

p.greet()
```

What happens?

Step 1:

```text
p.greet
```

requests attribute lookup.

Step 2:

Python searches the class.

It finds:

```text
Person.__dict__["greet"]
```

which is a function object.

Step 3:

The function's descriptor behavior runs.

Conceptually:

```python
Person.__dict__["greet"].__get__(p, Person)
```

This produces a bound method.

Step 4:

The bound method is called.

```python
p.greet()
```

becomes conceptually:

```python
Person.greet(p)
```

This is why `self` appears automatically.

---

# 27. The `self` Myth

Python does not magically insert `self` into the function definition.

Instead:

```python
p.greet
```

produces a bound method whose underlying function has:

```text
self = p
```

So:

```python
p.greet()
```

is conceptually similar to:

```python
Person.greet(p)
```

The binding occurs during attribute lookup.

This is a much better mental model than:

> "Python automatically passes self."

More precisely:

> **Function descriptors bind the instance to the function when the function is accessed through an instance.**

---

# 28. Bound Method

Consider:

```python
p.greet
```

Store it:

```python
method = p.greet
```

Now:

```python
method()
```

still knows which instance it belongs to.

Conceptually:

```text
bound method
│
├── underlying function → Person.greet
│
└── bound instance       → p
```

Therefore:

```python
method()
```

can call:

```python
Person.greet(p)
```

---

# 29. Inspecting a Bound Method

You can inspect:

```python
method.__self__
```

and:

```python
method.__func__
```

Conceptually:

```text
method.__self__
    ↓
instance p

method.__func__
    ↓
Person.greet function
```

This is an excellent introspection tool when learning or debugging methods.

---

# 30. Class Access vs Instance Access

This distinction is critical.

Given:

```python
class Person:

    def greet(self):
        print("Hello")
```

### Through class

```python
Person.greet
```

returns the function-like descriptor result without an instance being bound.

Conceptually:

```python
Person.__dict__["greet"].__get__(None, Person)
```

### Through instance

```python
p.greet
```

binds:

```text
p
```

to the function.

Conceptually:

```python
Person.__dict__["greet"].__get__(p, Person)
```

So:

```text
Person.greet
     ↓
function


p.greet
     ↓
bound method
```

---

# 31. Explicitly Calling `__get__`

You can demonstrate the mechanism:

```python
class Person:

    def greet(self):
        print("Hello")


p = Person()

function = Person.__dict__["greet"]

bound_method = function.__get__(p, Person)

bound_method()
```

This exposes the descriptor mechanism that normally stays hidden.

---

# 32. Why `Person.__dict__` Matters

When studying Python internals, this is extremely useful:

```python
Person.__dict__
```

It exposes the class namespace mapping.

You can inspect:

```python
Person.__dict__["greet"]
```

and discover that the method is actually a function object.

Similarly:

```python
p.__dict__
```

shows instance-level attributes.

Compare:

```text
Person.__dict__
    ↓
Class namespace


p.__dict__
    ↓
Instance namespace
```

This distinction is fundamental.

---

# 33. `property` Is a Descriptor

Consider:

```python
class Person:

    @property
    def name(self):
        return "Chinnu"
```

Then:

```python
p = Person()

print(p.name)
```

Notice:

```python
p.name
```

not:

```python
p.name()
```

Why?

Because `property` creates a descriptor.

Conceptually:

```text
@property
   ↓
property descriptor
   ↓
__get__()
   ↓
getter function
```

So:

```python
p.name
```

triggers descriptor behavior.

---

# 34. `property` Internally

Conceptually:

```python
class Person:

    def get_name(self):
        return "Chinnu"

    name = property(get_name)
```

Therefore:

```python
p.name
```

causes:

```text
property.__get__()
       ↓
get_name(p)
       ↓
result
```

This is why properties can look like ordinary attributes while executing code.

---

# 35. `staticmethod` Is Descriptor-Based

Consider:

```python
class Math:

    @staticmethod
    def add(a, b):
        return a + b
```

Usage:

```python
Math.add(10, 20)
```

and:

```python
m = Math()

m.add(10, 20)
```

No instance is automatically passed.

Conceptually:

```text
staticmethod descriptor
        ↓
returns underlying function
        ↓
no self binding
```

Therefore:

```python
m.add(10, 20)
```

does not become:

```python
Math.add(m, 10, 20)
```

---

# 36. `classmethod` Is Descriptor-Based

Consider:

```python
class Person:

    @classmethod
    def create(cls):
        return cls()
```

Usage:

```python
Person.create()
```

or:

```python
p = Person()
p.create()
```

The method receives the class:

```text
cls = Person
```

instead of the instance.

Conceptually:

```text
classmethod descriptor
        ↓
bind class
        ↓
Person.create()
        ↓
cls = Person
```

---

# 37. Method Types — Mental Comparison

```text
Normal method
─────────────
instance → self


classmethod
──────────
class → cls


staticmethod
────────────
nothing automatically bound


property
────────
attribute access → descriptor logic
```

All of these behaviors are connected to descriptors.

---

# 38. Introspection — What Is It?

Introspection means examining objects at runtime.

Python provides many tools:

```python
type()
id()
dir()
vars()
hasattr()
getattr()
setattr()
callable()
isinstance()
issubclass()
```

And the `inspect` module provides deeper capabilities.

---

# 39. `type()`

```python
type(obj)
```

answers:

> What is the object's immediate type?

Example:

```python
type(10)
```

returns:

```text
int
```

For:

```python
class Person:
    pass

p = Person()
```

```python
type(p)
```

returns:

```text
Person
```

---

# 40. `isinstance()`

Use:

```python
isinstance(p, Person)
```

to ask:

> Is this object an instance of this class or a compatible subclass?

Example:

```python
class Employee(Person):
    pass
```

Then:

```python
employee = Employee()

isinstance(employee, Person)
```

returns:

```text
True
```

because inheritance matters.

---

# 41. `issubclass()`

Use:

```python
issubclass(Employee, Person)
```

to ask about class relationships.

Important:

```python
issubclass()
```

expects classes, not arbitrary instances.

---

# 42. `dir()`

```python
dir(obj)
```

provides a list of names that can be relevant to attribute lookup.

But:

> `dir()` is not a perfect representation of `obj.__dict__`.

It is intended to provide a useful list for interactive exploration.

Don't treat it as the exact storage mechanism.

---

# 43. `vars()`

For objects with a `__dict__`:

```python
vars(obj)
```

is conceptually:

```python
obj.__dict__
```

Example:

```python
class Person:

    def __init__(self):
        self.name = "Alice"
        self.age = 25
```

Then:

```python
p = Person()

print(vars(p))
```

might show:

```text
{'name': 'Alice', 'age': 25}
```

---

# 44. `getattr()`

Instead of:

```python
obj.name
```

you can dynamically access:

```python
getattr(obj, "name")
```

This is useful when the attribute name is dynamic.

Example:

```python
attribute = "name"

value = getattr(obj, attribute)
```

Conceptually:

```text
getattr(obj, "name")
        ↓
attribute lookup
```

---

# 45. `getattr()` With Default

You can provide a fallback:

```python
getattr(obj, "name", None)
```

If the attribute doesn't exist:

```text
None
```

is returned.

This can be cleaner than manually catching:

```python
AttributeError
```

depending on the use case.

---

# 46. `setattr()`

Dynamic assignment:

```python
setattr(obj, "name", "Alice")
```

is conceptually similar to:

```python
obj.name = "Alice"
```

But if descriptors exist, the assignment can invoke descriptor behavior.

This is important.

Dynamic attribute operations still participate in Python's object model.

---

# 47. `hasattr()`

```python
hasattr(obj, "name")
```

checks whether attribute access succeeds without raising `AttributeError`.

Important edge case:

If your custom `__getattr__` dynamically supplies attributes, `hasattr()` may return `True`.

So:

```python
class Demo:

    def __getattr__(self, name):
        return "dynamic"
```

Then:

```python
hasattr(Demo(), "anything")
```

can be:

```text
True
```

This means `hasattr()` asks something closer to:

> "Can this attribute be successfully retrieved?"

not:

> "Is this exact name physically stored in `__dict__`?"

---

# 48. `inspect` Module

For deeper introspection:

```python
import inspect
```

Useful functions include:

```python
inspect.signature()
inspect.getsource()
inspect.getmembers()
inspect.isfunction()
inspect.ismethod()
inspect.isclass()
inspect.isroutine()
```

Example:

```python
def add(a, b=10):
    return a + b

print(inspect.signature(add))
```

Conceptually:

```text
(a, b=10)
```

This is especially useful for:

```text
Frameworks
Dependency injection
Testing
Documentation generation
CLI systems
RPC systems
Validation
Debugging
```

---

# 49. `inspect.signature()` and Decorators

This connects directly to your decorator learning.

A poorly written decorator:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

can make introspection report:

```text
(*args, **kwargs)
```

instead of the original function signature.

Using:

```python
from functools import wraps
```

helps:

```python
def decorator(func):

    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

This is why `functools.wraps` matters beyond aesthetics.

It improves:

```text
Metadata
Introspection
Debugging
Documentation
Tooling
```

---

# 50. `__name__`, `__qualname__`, `__module__`

Functions carry metadata.

Examples:

```python
func.__name__
func.__qualname__
func.__module__
func.__doc__
```

`__qualname__` is particularly useful for understanding where an object was defined.

For nested/class methods it can reveal contextual naming.

---

# 51. `__dict__` on Functions

Functions can themselves have attributes.

Example:

```python
def greet():
    pass

greet.role = "admin"
```

Then:

```python
print(greet.__dict__)
```

may show:

```text
{'role': 'admin'}
```

This demonstrates again:

> Functions are ordinary Python objects with their own attributes.

---

# 52. Descriptors and Attribute Lookup — Full Mental Model

A simplified but useful model for instance attribute lookup is:

```text
obj.attribute
      ↓
__getattribute__
      ↓
Look in object's class hierarchy
      ↓
Is there a data descriptor?
      │
   ┌──┴───┐
  YES     NO
   │       │
   ▼       ▼
__get__  instance __dict__
           │
        found?
        │
     ┌──┴──┐
    YES    NO
     │      │
     ▼      ▼
  return   class attribute
            │
            ▼
       descriptor?
            │
            ▼
          __get__
            │
            ▼
         result
            │
            ▼
      __getattr__ fallback
```

This is simplified because the actual algorithm includes details around class hierarchy and descriptor lookup, but it is an excellent working model.

---

# 53. Attribute Lookup Precedence

For practical reasoning, remember:

```text
1. Data descriptor
2. Instance dictionary
3. Non-data descriptor
4. Ordinary class attribute
5. Parent classes
6. __getattr__ fallback
```

The exact implementation details are handled by Python's object/type machinery.

The critical distinction is:

```text
Data descriptor
      >
Instance attribute
```

while:

```text
Instance attribute
      >
Non-data descriptor
```

---

# 54. Method Lookup Through the Same System

A method is not a magical special case.

Consider:

```python
obj.process()
```

Python first resolves:

```python
obj.process
```

That attribute lookup finds the function in the class.

The function is a descriptor.

It binds:

```text
obj
```

and returns a bound method.

Then:

```text
()
```

calls that bound method.

Therefore:

```text
obj.process()
```

is really two major operations:

```text
1. obj.process
2. result()
```

This is a powerful debugging model.

---

# 55. Why `obj.method` and `obj.method()` Are Different

This distinction is fundamental.

```python
obj.method
```

means:

> Retrieve the method object.

Whereas:

```python
obj.method()
```

means:

> Retrieve the method object and then call it.

You can prove this:

```python
method = obj.method

method()
```

The lookup happens once.

The call happens later.

---

# 56. Method Binding Is Dynamic

Consider inheritance:

```python
class Parent:

    def greet(self):
        print("Parent")


class Child(Parent):

    def greet(self):
        print("Child")
```

Then:

```python
obj = Child()

obj.greet()
```

Python's attribute lookup searches the class hierarchy and finds:

```text
Child.greet
```

This is why method resolution order matters.

---

# 57. MRO

Python uses the Method Resolution Order:

```python
Class.mro()
```

Example:

```python
Child.mro()
```

might produce:

```text
Child
Parent
object
```

This determines where Python searches for attributes and methods in inheritance hierarchies.

---

# 58. `super()` Connection

Consider:

```python
class Child(Parent):

    def greet(self):
        super().greet()
```

`super()` provides a proxy for continuing attribute lookup according to the MRO.

It is not simply:

```text
"call parent"
```

More accurately:

> `super()` performs attribute lookup starting at a specific point in the MRO.

This distinction becomes very important with multiple inheritance.

---

# 59. Descriptors + Inheritance

Descriptors participate in inherited attribute lookup too.

Suppose:

```python
class Parent:

    @property
    def name(self):
        return "Parent"
```

Then:

```python
class Child(Parent):
    pass
```

Now:

```python
Child().name
```

still triggers the inherited property descriptor.

So descriptor behavior naturally combines with:

```text
Inheritance
MRO
Attribute lookup
```

---

# 60. Production Perspective

These concepts are heavily used behind the scenes in Python frameworks.

Examples include:

```text
ORM model fields
@property-based domain models
Dependency injection
Validation frameworks
Lazy loading
Caching
Routing
Plugin systems
Mocking
Serialization
API frameworks
CLI frameworks
Testing tools
```

When a framework allows:

```python
class User(Model):
    name = StringField()
```

that `name` object may be doing far more than simply storing a value.

It may implement:

```python
__get__
__set__
```

to control access.

---

# 61. ORM Example Mental Model

Imagine:

```python
class User:

    name = StringField()
```

Then:

```python
user.name
```

could internally mean:

```text
user.name
   ↓
StringField.__get__()
   ↓
retrieve value from internal storage
   ↓
return string
```

And:

```python
user.name = "Alice"
```

could mean:

```text
assignment
   ↓
StringField.__set__()
   ↓
validate value
   ↓
store value
```

This is one reason descriptors are so useful for frameworks.

---

# 62. Lazy Loading Example

A descriptor can delay expensive work:

```text
user.profile
     ↓
descriptor
     ↓
already loaded?
   /      \
 yes       no
  ↓         ↓
return    load data
            ↓
          cache
            ↓
          return
```

This can implement lazy-loading behavior without changing the caller syntax.

The caller still writes:

```python
user.profile
```

---

# 63. Common Mistakes

## Mistake 1 — Thinking `__getattr__` Handles Every Attribute

It doesn't.

```text
__getattribute__
    ↓
every lookup

__getattr__
    ↓
fallback after normal lookup fails
```

---

# 64. Mistake 2 — Confusing `__getattribute__` With `__getattr__`

Remember:

```text
__getattribute__ = primary interception point

__getattr__ = fallback
```

---

# 65. Mistake 3 — Forgetting `__call__` Changes Callability

An instance:

```python
obj()
```

requires a callable object.

If the class doesn't provide the required call behavior, Python raises:

```text
TypeError
```

---

# 66. Mistake 4 — Thinking Methods Are Stored as Bound Methods in the Class

The class stores a function.

The bound method is produced when accessing the function through an instance.

Think:

```text
Class
 ↓
function descriptor

Instance
 ↓
bound method
```

---

# 67. Mistake 5 — Thinking `self` Is Magic

Better mental model:

```text
obj.method
      ↓
function.__get__(obj, Class)
      ↓
bound method
      ↓
obj becomes self
```

---

# 68. Mistake 6 — Thinking `obj.__dict__` Contains Everything

It doesn't.

Some behavior can come from:

```text
Class attributes
Descriptors
Base classes
Properties
Dynamic lookup
__getattr__
Slots
```

Therefore:

```python
obj.__dict__
```

is only one part of the object's accessible state.

---

# 69. Mistake 7 — Treating `dir()` as Storage

`dir()` is for discovering names.

It does not mean:

```text
every name in dir(obj)
=
obj.__dict__
```

These are different concepts.

---

# 70. Mistake 8 — Overriding `__getattribute__` Carelessly

This can cause:

```text
Infinite recursion
Unexpected behavior
Performance overhead
Broken framework behavior
Hard-to-debug attribute access
```

Only override it when you genuinely need that level of control.

---

# 71. Mistake 9 — Forgetting Descriptor Precedence

When debugging:

```python
obj.value
```

always ask:

```text
Is value a data descriptor?
Is value in instance __dict__?
Is value a non-data descriptor?
Is value inherited?
```

This often explains surprising behavior.

---

# 72. Mistake 10 — Using `__getattr__` to Hide Bugs

This:

```python
class Demo:

    def __getattr__(self, name):
        return None
```

can make missing attributes silently look valid.

Instead of:

```text
AttributeError
```

you get:

```text
None
```

This can hide programming errors.

In production, use dynamic fallback intentionally.

---

# 73. Production Safety Rules

When using these mechanisms:

```text
Prefer normal Python behavior by default.

Override __getattribute__ only when necessary.

Use __getattr__ for intentional fallback behavior.

Use descriptors when attribute access genuinely represents behavior.

Preserve metadata when wrapping callables.

Be careful with dynamic attribute generation.

Understand whether state belongs to class or instance.

Test descriptor precedence explicitly.

Measure performance when intercepting frequent attribute access.
```

---

# 74. Interview Questions

## `__call__`

1. What does `__call__` do?
2. How can an object become callable?
3. Why are callable objects useful?
4. How do class-based decorators use `__call__`?
5. What does `callable()` check?

## Attribute Access

6. What is the difference between `__getattribute__` and `__getattr__`?
7. When is `__getattr__` called?
8. How can overriding `__getattribute__` cause recursion?
9. How can you safely delegate to the default implementation?
10. How does `getattr()` relate to attribute lookup?

## Descriptors

11. What is a descriptor?
12. What methods define the descriptor protocol?
13. What is a data descriptor?
14. What is a non-data descriptor?
15. Which takes precedence: data descriptor or instance attribute?
16. Can a non-data descriptor be shadowed?
17. Why is `property` a descriptor?
18. Why are functions descriptors?

## Methods

19. How does Python bind `self`?
20. What is a bound method?
21. What is the difference between `Class.method` and `instance.method`?
22. What are `method.__self__` and `method.__func__`?
23. How does `super()` interact with MRO?
24. Why does `obj.method()` work without explicitly passing `self`?

## Introspection

25. Difference between `type()` and `isinstance()`?
26. What does `dir()` actually provide?
27. What does `vars()` return?
28. How do `getattr()` and `setattr()` work?
29. Why is `functools.wraps` important for introspection?
30. How would you inspect a function's signature?

## Senior

31. Explain the attribute lookup precedence.
32. Explain how `obj.method()` works internally.
33. Explain how `property`, `classmethod`, and `staticmethod` relate to descriptors.
34. How would you implement a validation descriptor?
35. How would you implement lazy loading with a descriptor?
36. Why might a framework use descriptors instead of ordinary properties?
37. How would you debug unexpected attribute resolution?
38. What risks exist when overriding `__getattribute__`?
39. How can decorators interfere with introspection?
40. Explain the relationship between functions, descriptors, bound methods, and `self`.

---

# 75. Senior-Level Mental Model

This entire topic can be compressed into one model:

```text
                         PYTHON OBJECT
                              │
                ┌─────────────┼─────────────┐
                │             │             │
             Callable     Attribute      Introspection
                │             │             │
             __call__    __getattribute__   type()
                │             │             │
                │         __getattr__       dir()
                │             │             │
                │             ▼             vars()
                │        Descriptors        inspect
                │             │
                │       ┌─────┴─────┐
                │       │           │
                │     __get__     __set__
                │       │
                │       ▼
                │   Method binding
                │       │
                │       ▼
                │   Bound method
                │       │
                └───────┬┘
                        ▼
                      CALL
```

---

# 76. The Most Important Connection

Remember this chain:

```text
class definition
      ↓
function stored in class
      ↓
function is a descriptor
      ↓
instance attribute lookup
      ↓
function.__get__(instance, class)
      ↓
bound method
      ↓
self is bound
      ↓
method call
```

Therefore:

```python
obj.method()
```

is best understood as:

```text
attribute lookup
        +
descriptor binding
        +
call
```

not simply:

```text
"call method"
```

---

# 77. One Complete Example

Consider:

```python
class Validator:

    def __init__(self, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self

        return instance.__dict__.get(self.name)

    def __set__(self, instance, value):

        if not isinstance(value, str):
            raise TypeError("Expected string")

        instance.__dict__[self.name] = value


class User:

    name = Validator("name")

    def __init__(self, name):
        self.name = name

    def greet(self):
        return f"Hello {self.name}"
```

Now:

```python
user = User("Alice")
```

Assignment:

```text
self.name = "Alice"
      ↓
Validator.__set__()
      ↓
validation
      ↓
instance.__dict__["name"]
```

Reading:

```python
user.name
```

becomes:

```text
Validator.__get__()
      ↓
instance.__dict__["name"]
      ↓
"Alice"
```

Method:

```python
user.greet()
```

becomes conceptually:

```text
user.greet
      ↓
find function in User
      ↓
function descriptor
      ↓
bind user
      ↓
bound method
      ↓
call
      ↓
self = user
```

This single example connects:

```text
Attribute lookup
Descriptors
Instance state
Methods
Binding
self
```

---

# 78. Final Architecture

You should now see Python roughly like this:

```text
                         PYTHON RUNTIME
                               │
                 ┌─────────────┴─────────────┐
                 │                           │
             OBJECT MODEL               CALL MODEL
                 │                           │
        ┌────────┼────────┐                  │
        │        │        │                  │
      Class   Instance  Function          __call__
        │        │        │                  │
        └────────┼────────┘                  │
                 │                           │
                 ▼                           ▼
          Attribute Lookup             Callable Object
                 │
        ┌────────┼────────┐
        │        │        │
   Descriptor  __dict__  MRO
        │
   ┌────┼────┐
   │    │    │
 __get__ __set__ __delete__
   │
   ▼
Method Binding
   │
   ├── Normal method → self
   ├── classmethod   → cls
   ├── staticmethod  → nothing
   └── property      → attribute-like behavior
```

---

# 79. Final Senior Mental Model

Do not memorize:

```text
"__call__ does this"
"__getattr__ does that"
"descriptor does something"
```

Instead understand the protocols.

### Calling

```text
obj()
 ↓
call protocol
 ↓
__call__
```

### Attribute access

```text
obj.x
 ↓
attribute lookup
 ↓
descriptor / instance / class / MRO
 ↓
result
```

### Missing attribute

```text
obj.x
 ↓
normal lookup fails
 ↓
__getattr__
```

### Method

```text
obj.method
 ↓
class lookup
 ↓
function descriptor
 ↓
__get__(obj, Class)
 ↓
bound method
```

### Property

```text
obj.x
 ↓
property descriptor
 ↓
getter
```

### Introspection

```text
type()
isinstance()
dir()
vars()
getattr()
setattr()
inspect.*
```

These are tools for observing the object model.

---

# 80. The Senior Rule to Remember

Whenever Python syntax looks "magical", look for a protocol underneath it.

For example:

```python
obj()
```

Ask:

```text
What is the call protocol?
```

```python
obj.name
```

Ask:

```text
What is the attribute lookup protocol?
```

```python
obj.method()
```

Ask:

```text
How did the method become bound?
```

```python
@property
```

Ask:

```text
What descriptor is controlling this?
```

```python
@Decorator
```

Ask:

```text
What object was passed?
What object was returned?
Is the returned object callable?
```

That is the transition from:

> **Python syntax knowledge**

to:

> **Python runtime understanding.**

---

# 81. Topic Completion Checklist

After studying this topic, you should be able to explain without memorization:

```text
□ What makes an object callable
□ How __call__ works
□ Why class-based decorators use __call__
□ Difference between __getattribute__ and __getattr__
□ Why __getattribute__ can recurse
□ What a descriptor is
□ Data vs non-data descriptors
□ Descriptor precedence
□ Why functions are descriptors
□ How methods become bound
□ Where self actually comes from
□ Difference between Class.method and obj.method
□ What __self__ and __func__ represent
□ Why property works without ()
□ How classmethod works
□ How staticmethod works
□ What MRO contributes
□ How super() performs lookup
□ What introspection means
□ Difference between type/isinstance/issubclass
□ How getattr/setattr work
□ Why wraps matters
□ How to reason about obj.method() internally
```

---

# 82. One-Line Senior Summary

> **Python method calls, properties, callable objects, decorators, and dynamic attribute behavior are not independent magic features—they are different applications of Python's object model, call protocol, attribute lookup, descriptor protocol, and runtime introspection.**