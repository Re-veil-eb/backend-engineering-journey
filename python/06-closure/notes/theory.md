# `06-closures.md`

> **Learning level:** Intermediate → Senior
> **Purpose:** Understand how Python functions retain access to variables from their enclosing scope, how closures work internally, and why closures are the foundation of decorators and function factories.

---

# 1. What is it?

A **closure** is a function that **remembers and can access variables from its enclosing scope even after the enclosing function has finished executing**.

Example:

```python
def outer(name):

    def inner():
        print(name)

    return inner
```

Now:

```python
greet = outer("Chinnu")
greet()
```

Output:

```text
Chinnu
```

The important question is:

> `outer()` has already finished. How can `inner()` still access `name`?

Because `inner()` forms a **closure** over `name`.

### Core idea

```text
outer()
   ↓
creates variable
   ↓
creates inner()
   ↓
inner() references outer variable
   ↓
inner() is returned
   ↓
outer() finishes
   ↓
inner() still remembers the variable
```

---

# 2. Why does it exist?

Closures exist because functions sometimes need to **carry state or configuration with them**.

Without closures, we might need global variables or classes for some simple stateful behavior.

For example:

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

Now:

```python
double = create_multiplier(2)
triple = create_multiplier(3)
```

Each function remembers its own configuration.

```text
double → remembers 2
triple → remembers 3
```

Closures are useful for:

* Maintaining state
* Function factories
* Decorators
* Configuration-specific functions
* Encapsulation
* Callbacks
* Creating customized behavior

---

# 3. How does it work internally?

Consider:

```python
def outer():

    x = 10

    def inner():
        return x

    return inner
```

When Python executes:

```python
inner = outer()
```

the following happens conceptually:

```text
outer()
   │
   ├── x = 10
   │
   ├── create inner()
   │
   ├── inner needs x
   │
   └── return inner
          │
          ▼
       inner function
          │
          ▼
    remembers enclosing x
```

After `outer()` finishes, `inner()` can still access `x`.

```python
print(inner())
```

Output:

```text
10
```

The enclosing state required by the function is retained.

---

# 4. Core concepts

## 4.1 Closure requires an enclosing scope

Example:

```python
def outer():
    message = "Hello"

    def inner():
        print(message)

    return inner
```

`message` belongs to `outer()`.

`inner()` accesses it.

Therefore:

```text
message
   ↑
   │
inner()
   ↑
   │
outer()
```

---

## 4.2 The inner function must reference the enclosing variable

This:

```python
def outer():

    x = 10

    def inner():
        print("Hello")

    return inner
```

contains a nested function, but `inner()` does not use `x`.

The important closure relationship is missing.

Compare:

```python
def outer():

    x = 10

    def inner():
        print(x)

    return inner
```

Now `inner()` depends on `x`.

That is the important closure pattern.

---

# 5. Execution flow

Consider:

```python
def outer(name):

    def inner():
        print(name)

    return inner


greet = outer("Chinnu")

greet()
```

Execution:

```text
1. Python creates outer()
        ↓
2. outer("Chinnu") is called
        ↓
3. name = "Chinnu"
        ↓
4. inner() is created
        ↓
5. inner() references name
        ↓
6. inner is returned
        ↓
7. greet references inner
        ↓
8. outer() has finished
        ↓
9. greet() is called
        ↓
10. inner() accesses remembered name
        ↓
11. "Chinnu" is printed
```

---

# 6. Closure vs Nested Function

These concepts are related but not identical.

### Nested function

A function defined inside another function.

```python
def outer():

    def inner():
        pass
```

### Closure

A function that retains access to variables from its enclosing lexical scope.

```python
def outer(x):

    def inner():
        return x

    return inner
```

Think:

```text
Nested Function
       +
Enclosing Variable Access
       +
Function escapes enclosing execution
       ↓
Closure
```

This distinction is important in interviews.

---

# 7. Closure and LEGB

You previously learned LEGB:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

Closures make the **E — Enclosing** part extremely important.

Example:

```python
def outer():

    x = 10

    def inner():
        print(x)

    return inner
```

When Python evaluates:

```python
print(x)
```

inside `inner()`:

```text
Local scope
    ↓
x not found
    ↓
Enclosing scope
    ↓
x = 10 found
```

So:

```text
Closure
   ↓
depends heavily on
   ↓
Enclosing scope
   ↓
LEGB
```

---

# 8. Returning the Inner Function

This is one of the most important closure patterns:

```python
def outer(x):

    def inner():
        return x

    return inner
```

Notice:

```python
return inner
```

not:

```python
return inner()
```

### `return inner`

Returns the function itself.

### `return inner()`

Executes the function immediately and returns its result.

This distinction is fundamental.

---

# 9. Closure Example — Multiplier Factory

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

Create functions:

```python
double = create_multiplier(2)
triple = create_multiplier(3)
```

Now:

```python
print(double(10))
print(triple(10))
```

Output:

```text
20
30
```

Why?

```text
double
  ↓
multiply()
  ↓
remembers number = 2


triple
  ↓
multiply()
  ↓
remembers number = 3
```

---

# 10. Multiple Closures Have Independent State

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
print(c1())
print(c1())
print(c2())
print(c2())
```

Output:

```text
1
2
1
2
```

Each invocation of `counter()` creates a separate enclosing environment.

Conceptually:

```text
c1
 │
 └── count = 0


c2
 │
 └── count = 0
```

They are independent.

---

# 11. `nonlocal` and Closures

Closures can read enclosing variables:

```python
def outer():

    count = 0

    def inner():
        return count

    return inner
```

But if we want to modify the enclosing variable:

```python
def outer():

    count = 0

    def inner():
        nonlocal count
        count += 1
        return count

    return inner
```

`nonlocal` is required.

It tells Python:

> This variable belongs to the nearest enclosing function scope.

---

# 12. Why `nonlocal` Matters

Without `nonlocal`:

```python
def outer():

    count = 0

    def inner():
        count += 1

    return inner
```

Python treats `count` as a local variable inside `inner()` because of the assignment.

Conceptually:

```text
inner()
  ↓
count += 1
  ↓
count considered local
  ↓
read before assignment
  ↓
UnboundLocalError
```

With:

```python
nonlocal count
```

Python knows:

```text
inner.count
      ↓
look in enclosing scope
      ↓
outer.count
```

---

# 13. Closure Internals — `__closure__`

Python exposes information about a function's closure through:

```python
__closure__
```

Example:

```python
def outer():

    x = 10

    def inner():
        return x

    return inner


func = outer()
```

You can inspect:

```python
print(func.__closure__)
```

You may see something similar to:

```text
(<cell at 0x...: int object at 0x...>,)
```

The exact memory address will differ.

The important idea is:

```text
function
   ↓
__closure__
   ↓
cell objects
   ↓
captured variables
```

---

# 14. Closure Cells

The captured variable is represented internally through a **cell object**.

For example:

```python
def outer():

    x = 100

    def inner():
        return x

    return inner
```

Conceptually:

```text
inner function
      │
      ▼
  closure
      │
      ▼
    cell
      │
      ▼
     x = 100
```

The cell provides the connection between the function and the captured variable.

---

# 15. `__code__.co_freevars`

Python also exposes the names of variables captured from the enclosing scope.

Example:

```python
def outer():

    x = 100

    def inner():
        return x

    return inner
```

You can inspect:

```python
func = outer()

print(func.__code__.co_freevars)
```

You may get:

```text
('x',)
```

This tells us that `x` is a **free variable** for `inner()`.

---

# 16. Connecting `co_freevars` and `__closure__`

For:

```python
def outer():

    x = 100

    def inner():
        return x

    return inner
```

conceptually:

```text
func.__code__.co_freevars
        ↓
      ("x",)

func.__closure__
        ↓
   cell containing 100
```

So Python has a relationship similar to:

```text
"x"
 ↓
closure cell
 ↓
100
```

This is useful when you want to understand how closures work internally rather than treating them as magic.

---

# 17. Closure State Is Not Global State

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

`count` is not global.

It belongs to the closure.

Therefore:

```text
Global state
    ❌

Closure state
    ✅
```

The state is associated with the returned function.

---

# 18. Closure as Encapsulation

Closures can hide state.

Example:

```python
def bank_account(balance):

    def deposit(amount):
        nonlocal balance
        balance += amount
        return balance

    return deposit
```

The caller cannot directly access `balance` through a normal variable name.

They interact through:

```python
account = bank_account(1000)

account(500)
```

This provides a lightweight form of encapsulation.

---

# 19. Closure vs Class

The same stateful behavior could be implemented using a class:

```python
class Counter:

    def __init__(self):
        self.count = 0

    def increment(self):
        self.count += 1
        return self.count
```

Or a closure:

```python
def counter():

    count = 0

    def increment():
        nonlocal count
        count += 1
        return count

    return increment
```

Both can maintain state.

### Closure

Good when:

* State is simple
* Behavior is limited
* You need a small factory
* You don't need multiple public operations

### Class

Often better when:

* State is complex
* Multiple operations are needed
* The object has a meaningful identity
* You need explicit public methods
* The design is expected to grow

Senior engineers choose based on design, not because one is "more Pythonic."

---

# 20. Function Factory

A function that creates customized functions is commonly called a **function factory**.

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
```

Conceptually:

```text
Function Factory
      │
      ├── configuration
      │
      ▼
creates customized function
      │
      ├── double
      ├── triple
      └── ...
```

Closures make this possible.

---

# 21. Closure + Decorator

This is one of the most important connections in your learning path.

Example:

```python
def decorator(func):

    def wrapper():
        print("Before")
        func()
        print("After")

    return wrapper
```

`wrapper()` accesses:

```python
func
```

from the enclosing `decorator()` scope.

Therefore:

```text
decorator()
    │
    ├── func
    │
    └── wrapper()
            │
            └── accesses func
```

When `wrapper` is returned, it retains access to `func`.

That is closure behavior.

---

# 22. Why Decorators Need Closures

Consider:

```python
@decorator
def greet():
    print("Hello")
```

Conceptually:

```python
greet = decorator(greet)
```

Inside `decorator()`:

```text
original greet
      ↓
      func
      ↓
wrapper created
      ↓
wrapper remembers func
      ↓
wrapper returned
      ↓
greet now references wrapper
```

Therefore:

```text
Decorator
   ↓
Higher-order function
   ↓
Nested function
   ↓
Closure
```

These concepts are not separate islands.

They are connected.

---

# 23. Closure and Late Binding

A very important Python behavior is **late binding**.

Consider:

```python
def create_functions():

    functions = []

    for i in range(3):

        def func():
            return i

        functions.append(func)

    return functions
```

Now:

```python
functions = create_functions()

for func in functions:
    print(func())
```

You might expect:

```text
0
1
2
```

But the result is:

```text
2
2
2
```

Why?

The functions don't capture the value of `i` at each iteration.

They reference the same enclosing variable.

When the functions are eventually called, `i` has the final value:

```text
2
```

This is called **late binding**.

---

# 24. Fixing Late Binding

One common solution is using a default argument:

```python
def create_functions():

    functions = []

    for i in range(3):

        def func(i=i):
            return i

        functions.append(func)

    return functions
```

Now:

```python
for func in create_functions():
    print(func())
```

Output:

```text
0
1
2
```

Here each function receives the current value as a default argument.

---

# 25. Why Late Binding Matters

This behavior matters when working with:

* Loops
* Callbacks
* Event handlers
* Async programming
* GUI programming
* Comprehensions
* Deferred execution

Senior-level question:

> Does the closure capture the value or the variable?

The useful mental model is:

> **A closure captures access to the variable/binding, not simply a frozen copy of its value.**

---

# 26. Common Mistakes

## Mistake 1 — Thinking every nested function is automatically a closure

Not every nested function demonstrates closure behavior.

The important relationship is the inner function's access to enclosing state.

---

## Mistake 2 — Confusing closure with default arguments

These are different mechanisms.

Closure:

```python
def outer(x):

    def inner():
        return x

    return inner
```

Default argument:

```python
def inner(x=x):
    return x
```

They can sometimes produce similar results but work differently.

---

## Mistake 3 — Forgetting `nonlocal`

If the inner function needs to modify enclosing state:

```python
nonlocal variable
```

is usually required.

---

## Mistake 4 — Confusing `return inner` and `return inner()`

```python
return inner
```

returns a function.

```python
return inner()
```

executes it.

---

## Mistake 5 — Ignoring late binding

Be especially careful with closures created inside loops.

---

# 27. Production Perspective

Closures are useful in production for:

### Configuration

```python
def create_client(host):

    def request(path):
        return send_request(host, path)

    return request
```

The generated function remembers configuration.

### Decorators

```python
def logger(func):

    def wrapper(*args, **kwargs):
        print("Calling", func.__name__)
        return func(*args, **kwargs)

    return wrapper
```

### Callbacks

A callback can retain configuration needed when it executes later.

### State

Closures can maintain small amounts of private state without introducing a full class.

---

# 28. When NOT to Use Closures

Closures are not always the best choice.

Avoid complicated closure-based designs when:

* State becomes large
* Many operations are needed
* Debugging becomes difficult
* The state needs a clear public interface
* Team members may struggle to understand the design

For example, if you have:

```text
balance
transactions
withdraw()
deposit()
transfer()
history()
validate()
audit()
```

a class may communicate the design much more clearly than a large closure system.

---

# 29. Closure and Memory Management

A closure can keep an object alive because the closure retains a reference to the object.

Example:

```python
def outer():

    large_object = create_large_object()

    def inner():
        return large_object

    return inner
```

As long as `inner` remains reachable, the referenced object may also remain reachable.

Therefore, closures can affect object lifetime and memory usage.

This becomes important when closures capture:

* Large objects
* Database connections
* File handles
* Request objects
* Large data structures

Senior engineers should understand what a closure is retaining.

---

# 30. Production Debugging

When debugging a closure, inspect:

```python
func.__closure__
```

and:

```python
func.__code__.co_freevars
```

Example:

```python
def outer(x):

    def inner():
        return x

    return inner


func = outer(100)
```

Then:

```python
print(func.__code__.co_freevars)
print(func.__closure__)
```

You can investigate what the function is capturing.

This is useful when debugging unexpected state retention.

---

# 31. Execution Model

A useful senior mental model is:

```text
             outer()
                │
                ▼
       creates local binding
                │
                ▼
          creates inner()
                │
                ▼
     inner references outer variable
                │
                ▼
       closure relationship
                │
                ▼
        return inner function
                │
                ▼
       outer() execution ends
                │
                ▼
    returned function remains alive
                │
                ▼
       captured environment
                │
                ▼
        inner() can access it
```

---

# 32. Important Relationships

Your learning chain is now:

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
LEGB
   │
   ▼
Nested Functions
   │
   ▼
Higher-Order Functions
   │
   ▼
CLOSURES
   │
   ├──────────────┐
   ▼              ▼
State          Function Factory
   │              │
   └──────┬───────┘
          ▼
      Decorators
```

The most important relationship:

```text
Nested Function
      +
Enclosing Variable
      +
Function escapes outer execution
      ↓
Closure
```

---

# 33. Common Interview Questions

Before moving to decorators, you should be able to answer these without memorizing definitions.

### Basic

1. What is a closure?
2. Why do closures exist?
3. How is a closure different from a nested function?
4. How does LEGB relate to closures?
5. What does a closure remember?

### Execution

6. What happens when an outer function returns an inner function?
7. How can the inner function access a variable after the outer function finishes?
8. What is `__closure__`?
9. What are closure cells?
10. What is `co_freevars`?

### `nonlocal`

11. Why is `nonlocal` required?
12. What happens if you modify an enclosing variable without `nonlocal`?
13. What is the difference between `global` and `nonlocal`?

### Advanced

14. What is late binding?
15. Why do closures created in loops sometimes produce unexpected results?
16. How can late binding be fixed?
17. Can closures cause objects to remain in memory?
18. When would you use a class instead of a closure?
19. How are closures used in decorators?
20. How are closures used in function factories?

---

# 34. Senior-Level Interview Question

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

What happens here?

```python
c1 = counter()
c2 = counter()

print(c1())
print(c1())
print(c2())
print(c1())
```

Expected output:

```text
1
2
1
3
```

Senior explanation:

```text
counter() #1
    ↓
creates closure #1
    ↓
count = 0

counter() #2
    ↓
creates closure #2
    ↓
count = 0

c1 and c2 do NOT share the same count.
```

Each invocation creates a new enclosing environment.

---

# 35. Senior-Level Mental Model

Don't think:

> "A closure is an inner function."

Think:

> **A closure is a function together with the preserved access to variables from its enclosing lexical environment.**

The mental model:

```text
                 CLOSURE
                    │
          ┌─────────┴─────────┐
          │                   │
      Function            Environment
          │                   │
      inner()              x = 10
          │                   │
          └─────────┬─────────┘
                    │
                    ▼
            Function + State
```

This is why closures are powerful.

A function doesn't merely carry **code**.

It can carry access to the **environment required by that code**.

---

# 36. Final Revision Table

| Concept             | Mental Model                                         |
| ------------------- | ---------------------------------------------------- |
| Closure             | Function + preserved enclosing environment           |
| Enclosing scope     | Scope where the outer function's variable exists     |
| Captured variable   | Variable accessed from enclosing scope               |
| `nonlocal`          | Modify variable in enclosing function scope          |
| `__closure__`       | Shows closure cell information                       |
| `co_freevars`       | Shows names of captured free variables               |
| Closure cell        | Internal object holding captured binding             |
| Function factory    | Function that creates customized functions           |
| Late binding        | Closure looks up the variable when executed          |
| State encapsulation | Keeping state accessible through controlled behavior |

---

# 37. Final Mental Model

```text
                    FUNCTION
                       │
                       ▼
               First-Class Object
                       │
                       ▼
               Nested Function
                       │
                       ▼
              Enclosing Variable
                       │
                       ▼
                  LEGB → E
                       │
                       ▼
                Variable Captured
                       │
                       ▼
              Function Returned
                       │
                       ▼
             Outer Function Ends
                       │
                       ▼
            Captured Environment
               Still Accessible
                       │
                       ▼
                    CLOSURE
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
      Function Factory       Decorator
             │                   │
             └─────────┬─────────┘
                       ▼
                 Stateful/
              Configured Behavior
```

> **Key takeaway:**
> **A closure is not simply a nested function. It is a function that retains access to the relevant enclosing environment, allowing the function to carry state or configuration with it even after the outer function has completed. This mechanism is one of the core building blocks behind Python decorators and function factories.**
