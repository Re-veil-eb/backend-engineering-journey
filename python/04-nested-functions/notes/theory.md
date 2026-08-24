# Python Nested Functions

> **Learning level:** Intermediate → Senior
> **Purpose:** Understand why Python allows functions inside functions, how nested functions access enclosing variables, how they are created and executed, and why nested functions are the foundation for closures and decorators.

---

# 1. What is a Nested Function?

A **nested function** is a function defined inside another function.

Example:

```python
def outer():

    def inner():
        print("Hello")

    inner()
```

Here:

```text
outer()
  │
  └── inner()
```

`inner()` is nested inside `outer()`.

The outer function is commonly called the **outer function**, and the inner function is called the **nested/inner function**.

---

# 2. Why Do Nested Functions Exist?

Nested functions allow us to:

* Keep helper logic private to another function
* Organize related logic
* Access variables from an enclosing function
* Create closures
* Build decorators
* Create function factories
* Maintain state without global variables
* Encapsulate implementation details

The most important reason for your learning path is:

```text
Nested Function
      ↓
Enclosing Scope
      ↓
Closure
      ↓
Decorator
```

---

# 3. Basic Structure

The basic structure is:

```python
def outer():

    def inner():
        pass

    inner()
```

Conceptually:

```text
outer function
│
├── local variables
│
├── inner function definition
│
└── inner function call
```

The nested function belongs to the execution context of the outer function.

---

# 4. Defining a Nested Function Does Not Mean Executing It

Consider:

```python
def outer():

    def inner():
        print("Hello")
```

Calling:

```python
outer()
```

does not automatically execute:

```python
inner()
```

The inner function is defined, but it is not called.

To execute it:

```python
def outer():

    def inner():
        print("Hello")

    inner()
```

Now:

```text
outer()
  ↓
create/define inner
  ↓
call inner()
  ↓
print Hello
```

---

# 5. Execution Flow

Consider:

```python
def outer():

    print("Outer")

    def inner():
        print("Inner")

    inner()

outer()
```

Execution:

```text
1. Python calls outer()
        ↓
2. Execute print("Outer")
        ↓
3. Define/create inner function
        ↓
4. Call inner()
        ↓
5. Execute print("Inner")
        ↓
6. inner() returns
        ↓
7. outer() returns
```

Output:

```text
Outer
Inner
```

---

# 6. Nested Functions Have Their Own Local Scope

Consider:

```python
def outer():

    def inner():
        x = 10
        print(x)

    inner()
```

`x` belongs to `inner()`.

Conceptually:

```text
outer()
  │
  └── inner()
        │
        └── x = 10
```

`x` is not automatically available in `outer()`.

---

# 7. Outer Variables Can Be Accessed by Inner Functions

This is where nested functions become more interesting.

```python
def outer():

    x = 10

    def inner():
        print(x)

    inner()
```

`inner()` doesn't have a local `x`.

Python therefore follows LEGB:

```text
inner local
    ↓
not found
    ↓
enclosing scope
    ↓
x = 10
    ↓
found
```

Output:

```text
10
```

This is the foundation of closures.

---

# 8. Inner Functions Can Read Enclosing Variables

Example:

```python
def outer():

    message = "Hello"

    def inner():
        print(message)

    inner()
```

The inner function reads:

```python
message
```

from the enclosing scope.

The relationship is:

```text
outer()
  │
  ├── message
  │
  └── inner()
        │
        └── reads message
```

---

# 9. Inner Functions Cannot Normally Modify Enclosing Variables

Consider:

```python
def outer():

    count = 0

    def inner():
        count = count + 1

    inner()
```

This causes an error because:

```python
count = count + 1
```

makes `count` a local variable inside `inner()`.

Python effectively sees:

```text
inner local:
    count
```

Then the right side tries to read that local variable before it has been assigned.

This leads to:

```text
UnboundLocalError
```

---

# 10. Using `nonlocal`

To modify a variable from the enclosing function:

```python
def outer():

    count = 0

    def inner():
        nonlocal count
        count += 1

    inner()

    print(count)
```

Output:

```text
1
```

The `nonlocal` keyword tells Python:

> Use the variable from the nearest enclosing function scope.

---

# 11. `nonlocal` Relationship

```text
outer()
  │
  ├── count = 0
  │
  └── inner()
        │
        ├── nonlocal count
        │
        └── modifies outer.count
```

Without `nonlocal`:

```text
inner.count
```

would be treated as a local variable.

With `nonlocal`:

```text
outer.count
```

is modified.

---

# 12. Returning a Nested Function

A very important pattern is returning the inner function.

```python
def outer():

    def inner():
        print("Hello")

    return inner
```

Now:

```python
result = outer()
```

`result` contains a reference to the nested function.

So:

```python
result()
```

executes `inner()`.

Conceptually:

```text
outer()
   │
   ▼
inner function object
   │
   ▼
result
   │
   ▼
result()
   │
   ▼
inner()
```

---

# 13. Function Definition vs Function Execution

This distinction is important.

```python
def outer():

    def inner():
        print("Hello")

    return inner
```

When `outer()` executes:

```text
Create/define inner
       ↓
Return inner function object
```

It does **not** execute `inner()`.

The returned function is executed only later:

```python
result()
```

---

# 14. Why Return a Nested Function?

Returning a nested function allows us to create a function dynamically.

Example:

```python
def create_greeting():

    def greet():
        print("Hello")

    return greet
```

Then:

```python
greet_function = create_greeting()
```

Now:

```python
greet_function()
```

executes the returned function.

This becomes especially powerful when the nested function uses variables from the outer function.

---

# 15. Nested Function + Enclosing Variable

Consider:

```python
def outer(name):

    def inner():
        print(name)

    return inner
```

Now:

```python
greet = outer("Chinnu")
```

and:

```python
greet()
```

Output:

```text
Chinnu
```

The important question is:

> How does `inner()` still know `"Chinnu"`?

This leads to the concept of a **closure**.

---

# 16. Nested Function → Closure

The progression is:

```text
Nested Function
       ↓
Inner function accesses
enclosing variable
       ↓
Inner function returned
       ↓
Enclosing state remains accessible
       ↓
Closure
```

Example:

```python
def outer(name):

    def inner():
        print(name)

    return inner
```

`inner` uses `name` from `outer`.

When the inner function is returned, Python preserves the required enclosing state.

---

# 17. A Simple Closure Example

```python
def create_greeting(name):

    def greet():
        return f"Hello {name}"

    return greet
```

Usage:

```python
greet_chinnu = create_greeting("Chinnu")
greet_developer = create_greeting("Developer")
```

Now:

```python
print(greet_chinnu())
print(greet_developer())
```

Output:

```text
Hello Chinnu
Hello Developer
```

Each returned function has access to its own enclosing state.

Conceptually:

```text
create_greeting("Chinnu")
        │
        └── greet → remembers "Chinnu"


create_greeting("Developer")
        │
        └── greet → remembers "Developer"
```

---

# 18. Nested Functions as Private Helpers

Nested functions can be useful when a helper should only be used inside one function.

Example:

```python
def process_user(user):

    def validate():
        return user is not None

    if validate():
        print("Processing user")
```

`validate()` is an implementation detail of `process_user()`.

It doesn't need to be exposed at module level.

This can improve organization and encapsulation when used appropriately.

---

# 19. Nested Function vs Module-Level Function

Compare:

```python
def validate_user(user):
    return user is not None


def process_user(user):
    if validate_user(user):
        ...
```

with:

```python
def process_user(user):

    def validate_user():
        return user is not None

    if validate_user():
        ...
```

The second version communicates:

> `validate_user()` exists only as part of the implementation of `process_user()`.

However, nesting is not automatically better.

If the helper is reusable or independently testable, a module-level function may be more appropriate.

---

# 20. Nested Functions and Encapsulation

Nested functions can hide implementation details.

Example:

```python
def calculate():

    def helper():
        ...
    
    ...
```

The helper is not intended to become part of the module's public API.

This creates a limited form of implementation encapsulation.

However:

> Nested functions should be used for meaningful locality, not simply to make code look advanced.

---

# 21. Nested Functions and Higher-Order Functions

A nested function becomes especially useful when combined with first-class functions.

Example:

```python
def outer():

    def inner():
        return "Hello"

    return inner
```

Here:

```text
outer()
  ↓
returns a function
```

Therefore `outer()` participates in higher-order function behavior.

The concepts connect:

```text
Functions are objects
       ↓
Functions can be returned
       ↓
Nested function
       ↓
Higher-order function
```

---

# 22. Nested Functions and Callbacks

A nested function can be passed as a callback.

Example:

```python
def process():

    def callback():
        print("Completed")

    execute(callback)
```

The nested function can be passed like any other function.

This is another consequence of Python's first-class function model.

---

# 23. Nested Functions and Decorators

Decorators commonly use nested functions.

Example:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

Here:

```text
decorator()
  │
  ├── func
  │
  └── wrapper()
```

`wrapper()` is a nested function.

It accesses:

```python
func
```

from the enclosing scope.

This creates the relationship:

```text
Nested Function
      ↓
Enclosing Scope
      ↓
Captured Function
      ↓
Closure
      ↓
Decorator
```

---

# 24. Why Decorators Need This Pattern

Suppose:

```python
def greet():
    print("Hello")
```

A decorator wants to add behavior:

```text
Before
Hello
After
```

A nested wrapper can do this:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

The wrapper:

1. Is created inside the decorator.
2. Has access to `func`.
3. Calls `func()`.
4. Adds additional behavior.
5. Is returned.

That is the basic architecture of a decorator.

---

# 25. Nested Functions and State

Nested functions can maintain state without using global variables.

Example:

```python
def counter():

    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Usage:

```python
c = counter()

print(c())
print(c())
print(c())
```

Output:

```text
1
2
3
```

The state belongs to the enclosing scope.

Conceptually:

```text
counter()
   │
   ├── count = 0
   │
   └── increment()
          │
          └── modifies count
```

This is a closure-based state pattern.

---

# 26. Multiple Instances of Nested Functions

Consider:

```python
def counter():

    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Create two counters:

```python
c1 = counter()
c2 = counter()
```

Now:

```python
c1()
c1()
c2()
```

produces:

```text
1
2
1
```

Why?

Because each call to `counter()` creates a separate enclosing environment.

Conceptually:

```text
c1
 │
 └── count = 0


c2
 │
 └── count = 0
```

They maintain independent state.

---

# 27. Execution Flow of the Counter

For:

```python
c = counter()
```

the flow is:

```text
Call counter()
     ↓
Create local count = 0
     ↓
Create increment function
     ↓
increment captures count
     ↓
Return increment
     ↓
c references increment
```

Then:

```python
c()
```

does:

```text
Call increment()
     ↓
Access captured count
     ↓
count += 1
     ↓
Return count
```

---

# 28. Nested Function and Variable Lookup

Consider:

```python
def outer():

    x = 10

    def inner():

        print(x)

    return inner
```

When `inner()` executes:

```text
Look for x
   ↓
Local scope
   ↓
Not found
   ↓
Enclosing scope
   ↓
Found x = 10
```

This is the LEGB rule you just learned.

So nested functions are where your LEGB knowledge becomes practical.

---

# 29. Common Mistakes

## Mistake 1 — Thinking nested function executes automatically

```python
def outer():

    def inner():
        print("Hello")
```

Defining `inner()` doesn't call it.

You need:

```python
inner()
```

or:

```python
return inner
```

---

## Mistake 2 — Confusing `inner` and `inner()`

```python
return inner
```

returns the function object.

```python
return inner()
```

executes the function and returns its result.

This distinction is extremely important.

---

## Mistake 3 — Forgetting `nonlocal`

This:

```python
def outer():

    count = 0

    def inner():
        count += 1
```

doesn't modify the outer `count`.

You need:

```python
def inner():
    nonlocal count
    count += 1
```

---

## Mistake 4 — Assuming every nested function is a closure

A nested function becomes relevant to closure behavior when it accesses variables from its enclosing scope and that relationship needs to persist.

Simply placing one function inside another is not the complete mental model of a closure.

---

## Mistake 5 — Nesting everything

Nested functions are not automatically better.

Use them when:

* The helper is local to one operation.
* The helper needs enclosing state.
* You are implementing a closure.
* You are implementing a decorator.
* Locality improves readability.

Otherwise, a normal module-level function may be clearer.

---

# 30. Production Perspective

Nested functions appear in production code for:

* Decorators
* Closures
* Function factories
* Local helper logic
* Callbacks
* State encapsulation
* Adapters
* Middleware
* Configuration-specific functions

A senior engineer asks:

> Why does this function need to be nested?

Good answers include:

```text
It needs enclosing state.
It is implementation-specific.
It is part of a closure.
It is a decorator wrapper.
It should not be exposed as a module-level API.
```

Bad reason:

```text
"Because Python allows it."
```

---

# 31. Nested Functions and Testability

Nested functions can make isolated testing more difficult.

For example:

```python
def process():

    def helper():
        ...
```

Testing `helper()` independently may not be straightforward because it isn't exposed at module level.

Therefore:

> Use nested functions when locality or closure behavior provides real value.

If the helper contains substantial business logic, extracting it into a separate function may improve testability and maintainability.

---

# 32. Nested Functions and Memory

When an inner function retains access to an enclosing variable, Python may preserve the required state so the function can continue using it.

Example:

```python
def outer():

    value = 100

    def inner():
        return value

    return inner
```

After:

```python
func = outer()
```

`func()` can still return:

```text
100
```

even though the original call to `outer()` has completed.

This behavior is one of the key properties behind closures.

---

# 33. Senior-Level Mental Model

Don't think:

> "A nested function is simply a function written inside another function."

Think:

> **A nested function is a function whose definition occurs within another function's scope, giving it access to the enclosing lexical environment and enabling patterns such as closures, state encapsulation, function factories, and decorators.**

The important structure is:

```text
                 outer()
                    │
          ┌─────────┴─────────┐
          │                   │
     local state          inner function
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
             Enclosing Scope
                    │
                    ▼
                Closure
```

---

# 34. The Bigger Learning Chain

You have now learned:

```text
FUNCTION
   │
   ▼
Functions are objects
   │
   ▼
*args / **kwargs
   │
   ▼
Scope / LEGB
   │
   ▼
NESTED FUNCTIONS
   │
   ▼
Higher-Order Functions
   │
   ▼
CLOSURES
   │
   ▼
DECORATORS
```

The next major concept is **Higher-Order Functions**.

---

# 35. Senior Interview Questions

Before moving forward, you should be able to explain:

1. What is a nested function?
2. Why does Python allow functions inside functions?
3. Does defining a nested function execute it?
4. What happens when an outer function calls its inner function?
5. Can an inner function access variables from its outer function?
6. How does LEGB work with nested functions?
7. What is the difference between `return inner` and `return inner()`?
8. Why do nested functions sometimes need `nonlocal`?
9. What happens if an inner function assigns to an enclosing variable without `nonlocal`?
10. How can a nested function maintain state?
11. How can two returned nested functions maintain separate state?
12. How are nested functions related to closures?
13. How are nested functions used in decorators?
14. Why shouldn't every helper function be nested?
15. What are the testability trade-offs of nested functions?
16. How does a nested function access an enclosing variable after the outer function returns?
17. Explain the execution flow of a function factory.
18. Why is `return inner` different from `return inner()`?
19. How can nested functions provide implementation encapsulation?
20. Explain the relationship:

```text
Nested Function
→ Enclosing Scope
→ Closure
→ Decorator
```

---

# 36. Final Revision Table

| Concept           | Mental Model                                       |
| ----------------- | -------------------------------------------------- |
| Nested function   | Function defined inside another function           |
| Outer function    | Function containing the nested function            |
| Inner function    | Function defined inside another function           |
| Enclosing scope   | Scope belonging to the outer function              |
| `nonlocal`        | Access/modify an enclosing function variable       |
| `return inner`    | Return function object                             |
| `return inner()`  | Execute inner function and return its result       |
| Closure           | Inner function retaining access to enclosing state |
| Function factory  | Function that creates/returns another function     |
| Decorator wrapper | Nested function used to wrap another callable      |

---

# 37. Final Mental Model

```text
                    outer()
                       │
                       ▼
              Create outer state
                       │
                       ▼
              Define inner()
                       │
             ┌─────────┴─────────┐
             │                   │
        call inner()        return inner
             │                   │
             ▼                   ▼
       execute now          function object
                                 │
                                 ▼
                         enclosing state
                                 │
                                 ▼
                              Closure
                                 │
                                 ▼
                             Decorator
```

> **Key takeaway:**
> **Nested functions are not just about putting one function inside another. They create a powerful relationship between functions and their enclosing scope. That relationship becomes the foundation for closures, function factories, state encapsulation, and decorators.**
