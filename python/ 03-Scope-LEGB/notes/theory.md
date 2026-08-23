# Python Scope and LEGB

> **Learning level:** Intermediate → Senior
> **Purpose:** Understand how Python finds variables, how namespaces and scopes work, and how Local, Enclosing, Global, and Built-in scopes connect to nested functions, closures, and decorators.

---

# 1. What is Scope?

**Scope** defines the region in which a name can be accessed.

For example:

```python
def test():
    x = 10
```

Here, `x` belongs to the local scope of `test()`.

The important question is:

> When Python sees a name such as `x`, where does it look for that name?

Python follows a defined lookup mechanism called **LEGB**.

---

# 2. LEGB Rule

LEGB stands for:

```text
L → Local
E → Enclosing
G → Global
B → Built-in
```

When Python needs to resolve a name, it searches in this order:

```text
Local
  ↓
Enclosing
  ↓
Global
  ↓
Built-in
  ↓
NameError
```

This is the core mental model for Python variable lookup.

---

# 3. Local Scope

Local scope belongs to the currently executing function.

Example:

```python
def test():
    x = 10
    print(x)
```

When `test()` executes:

```text
test()
 │
 └── x = 10
```

`x` is local to `test()`.

Outside the function:

```python
print(x)
```

Python cannot find `x` in the current scope chain and may raise:

```text
NameError
```

---

# 4. Local Scope Is Created During Function Execution

Consider:

```python
def test():
    x = 10
```

The local variable `x` is associated with the function's execution context.

Conceptually:

```text
Call test()
    ↓
Create execution frame
    ↓
Local namespace
    ↓
x = 10
    ↓
Execute function
    ↓
Function returns
```

The local execution state belongs to that function invocation.

---

# 5. Enclosing Scope

The **enclosing scope** exists when functions are nested.

Example:

```python
def outer():

    x = 10

    def inner():
        print(x)

    inner()
```

For `inner()`:

```text
Local
  ↓
Enclosing → outer()
  ↓
Global
  ↓
Built-in
```

`x` isn't local to `inner()`.

Python therefore searches the enclosing scope and finds:

```python
x = 10
```

---

# 6. Why Is Enclosing Scope Important?

Enclosing scope is the foundation of **closures**.

Consider:

```python
def outer():

    message = "Hello"

    def inner():
        print(message)

    return inner
```

`inner()` accesses `message` from the enclosing scope.

This is not a global variable.

It belongs to `outer()`.

Therefore:

```text
inner()
  │
  ├── Local
  │
  ├── Enclosing → message
  │
  ├── Global
  │
  └── Built-in
```

---

# 7. Global Scope

A variable defined at the module level belongs to the global scope.

```python
x = 100

def test():
    print(x)
```

Inside `test()`, Python doesn't find `x` locally.

There is no enclosing function.

So it searches the global scope:

```text
Local
  ↓
Enclosing
  ↓
Global → x = 100
```

and finds it.

---

# 8. Built-in Scope

Python also provides a built-in namespace containing names such as:

```python
len
print
sum
max
min
str
int
list
```

Example:

```python
def test():
    values = [1, 2, 3]
    print(len(values))
```

If Python doesn't find `len` in:

```text
Local
Enclosing
Global
```

it finds it in:

```text
Built-in
```

---

# 9. Complete LEGB Example

```python
x = "global"

def outer():

    x = "enclosing"

    def inner():

        x = "local"

        print(x)

    inner()

outer()
```

Output:

```text
local
```

Why?

Because `inner()` has its own local `x`.

Lookup stops immediately:

```text
Local → found
```

Python doesn't need to search further.

---

# 10. LEGB Search Example

Now remove the local variable:

```python
x = "global"

def outer():

    x = "enclosing"

    def inner():

        print(x)

    inner()

outer()
```

Now:

```text
inner local
    ↓
not found

enclosing
    ↓
x = "enclosing"
    ↓
found
```

Output:

```text
enclosing
```

---

# 11. Another Example

Remove the enclosing variable too:

```python
x = "global"

def outer():

    def inner():

        print(x)

    inner()

outer()
```

Lookup becomes:

```text
Local
  ↓
not found

Enclosing
  ↓
not found

Global
  ↓
x = "global"
```

Output:

```text
global
```

---

# 12. Final LEGB Example

If `x` doesn't exist anywhere:

```python
def test():
    print(x)

test()
```

Python searches:

```text
Local       → not found
Enclosing   → not applicable
Global      → not found
Built-in    → not found
```

Result:

```text
NameError
```

---

# 13. Scope vs Namespace

These concepts are related but not identical.

### Namespace

A namespace is a mapping between names and objects.

Conceptually:

```text
"name" → object
```

Example:

```python
x = 10
```

can be thought of as:

```text
x → 10
```

### Scope

Scope determines **where a particular namespace is searched/accessed**.

A useful mental model is:

> Namespace stores name-to-object mappings; scope determines the visibility and lookup rules for those names.

---

# 14. Global Namespace

At module level:

```python
x = 10
y = 20
```

the module's global namespace contains references such as:

```text
x → 10
y → 20
```

Functions defined in that module can normally access global names.

---

# 15. Local Namespace

Inside:

```python
def test():
    x = 10
    y = 20
```

the executing function has local state containing:

```text
x → 10
y → 20
```

This local namespace is associated with the current function execution.

---

# 16. Name Lookup Is Not the Same as Assignment

This distinction is very important.

Consider:

```python
x = 10

def test():
    print(x)
```

Python performs a lookup.

It finds the global `x`.

But:

```python
x = 20
```

inside the function is an assignment.

Python normally treats that `x` as local to the function.

Example:

```python
x = 10

def test():
    x = 20
    print(x)
```

Output:

```text
20
```

The assignment creates/binds a local name.

---

# 17. The Assignment Trap

Consider:

```python
x = 10

def test():
    print(x)
    x = 20

test()
```

This may surprise beginners.

Python determines that `x` is a local variable in `test()` because of the assignment:

```python
x = 20
```

Therefore the earlier:

```python
print(x)
```

tries to access the local `x` before it has been assigned.

This results in:

```text
UnboundLocalError
```

---

# 18. Why Does This Happen?

The important concept is:

> Python determines local variable binding for a function based on assignments in that function.

Conceptually:

```text
test()
 │
 ├── x is considered local
 │
 ├── print(x)
 │      ↓
 │   local x doesn't have a value yet
 │
 └── x = 20
```

So Python does not simply decide:

> "There is a global x, so use it."

The local binding takes precedence.

---

# 19. `global` Keyword

The `global` keyword tells Python:

> This name refers to the global variable rather than creating a local binding.

Example:

```python
x = 10

def test():
    global x
    x = 20

test()

print(x)
```

Output:

```text
20
```

Without:

```python
global x
```

the assignment would normally create a local `x`.

---

# 20. What `global` Actually Changes

Consider:

```python
x = 10

def test():
    global x
    x = 20
```

Now the assignment:

```python
x = 20
```

targets the global binding.

Conceptually:

```text
Global namespace
    │
    └── x → 10

test()
    │
    └── global x
          ↓
       x → 20
```

---

# 21. `nonlocal` Keyword

`nonlocal` is used with nested functions.

It tells Python:

> Use a variable from an enclosing function scope instead of creating a new local variable.

Example:

```python
def outer():

    count = 0

    def inner():
        nonlocal count
        count += 1

    inner()
```

Here:

```text
inner()
  │
  ├── Local
  │
  └── Enclosing → count
```

`nonlocal` tells Python to modify that enclosing `count`.

---

# 22. `global` vs `nonlocal`

This distinction must be very clear.

### `global`

Refers to:

```text
Global/module scope
```

### `nonlocal`

Refers to:

```text
Nearest enclosing function scope
```

Example:

```python
x = "global"

def outer():

    x = "enclosing"

    def inner():

        nonlocal x
        x = "changed"

    inner()
```

`nonlocal x` modifies:

```text
outer.x
```

not:

```text
global x
```

---

# 23. Scope Chain With `nonlocal`

Consider:

```python
def outer():

    count = 0

    def inner():

        nonlocal count
        count += 1

    return inner
```

The relationship is:

```text
Global
  │
  ▼
outer()
  │
  └── count
       ▲
       │
     nonlocal
       │
       ▼
inner()
```

This mechanism is fundamental to closures.

---

# 24. LEGB and Closures

Consider:

```python
def outer():

    message = "Hello"

    def inner():
        print(message)

    return inner
```

When `inner()` executes:

```text
Local
  ↓
message not found

Enclosing
  ↓
message found
```

The function retains access to the enclosing variable.

That leads to the concept of a **closure**.

Therefore:

```text
LEGB
  ↓
Enclosing scope
  ↓
Nested function
  ↓
Captured variable
  ↓
Closure
```

---

# 25. Scope and Decorators

Decorators also depend on this mechanism.

Example:

```python
def decorator(func):

    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)

    return wrapper
```

`wrapper()` uses:

```python
func
```

which belongs to the enclosing scope of `decorator()`.

Conceptually:

```text
decorator(func)
    │
    ├── func
    │
    └── wrapper()
          │
          └── accesses func
```

This is why closures are an important part of understanding decorators.

---

# 26. Shadowing

A local variable can have the same name as a global variable.

```python
x = "global"

def test():
    x = "local"
    print(x)
```

Output:

```text
local
```

The local variable **shadows** the global variable within that scope.

The global variable still exists.

The local name simply takes precedence during lookup.

---

# 27. Built-in Shadowing

You can technically create a variable with the same name as a built-in:

```python
len = 100
```

Now:

```python
len([1, 2, 3])
```

will fail because Python finds your global `len` before it reaches the built-in namespace.

Lookup:

```text
Local
  ↓
Enclosing
  ↓
Global → len = 100
  ↓
STOP
```

It never reaches:

```text
Built-in → len()
```

Therefore:

> Avoid unnecessarily shadowing built-in names.

---

# 28. Scope Does Not Mean Lifetime in a Simple One-to-One Way

A beginner may think:

> "When the function ends, everything related to it disappears."

That is a useful basic model, but closures show why it is incomplete.

Example:

```python
def outer():

    message = "Hello"

    def inner():
        return message

    return inner
```

After:

```python
func = outer()
```

the returned function can still access `message`.

This is possible because the required enclosing state is retained as part of the closure mechanism.

This leads directly into the next topic: **Nested Functions** and then **Closures**.

---

# 29. Common Mistakes

## Mistake 1 — Thinking Python searches Global before Local

Incorrect:

```text
Global → Local
```

Correct:

```text
Local → Enclosing → Global → Built-in
```

---

## Mistake 2 — Thinking `nonlocal` means global

Incorrect:

```text
nonlocal = global
```

Correct:

```text
nonlocal = enclosing function scope
```

---

## Mistake 3 — Thinking assignment searches LEGB

Reading a variable and assigning a variable are different operations.

```python
print(x)
```

performs name lookup.

But:

```python
x = 10
```

creates/binds a name in the appropriate scope.

---

## Mistake 4 — Thinking nested functions automatically create closures

A nested function by itself isn't enough to understand a closure.

The important part is that the inner function **uses/captures a variable from the enclosing scope**.

---

## Mistake 5 — Overusing `global`

Using `global` everywhere can create difficult-to-maintain code.

Prefer:

* Function arguments
* Return values
* Objects/classes
* Encapsulation
* Dependency injection

when appropriate.

---

# 30. Production Perspective

In production systems, understanding scope helps with:

* Debugging `NameError`
* Debugging `UnboundLocalError`
* Understanding closures
* Understanding decorators
* Managing state
* Avoiding accidental global state
* Designing clean APIs
* Understanding concurrency-related state issues
* Understanding framework callbacks and wrappers

Senior engineers should be careful with global mutable state because it can create:

* Hidden dependencies
* Difficult testing
* Unexpected side effects
* Coupling
* Concurrency problems

Prefer explicit data flow where practical.

---

# 31. Scope and Dependency Flow

Compare:

```python
database = ...

def save_user(user):
    database.save(user)
```

with:

```python
def save_user(user, database):
    database.save(user)
```

The second version makes the dependency explicit.

This is easier to:

* Test
* Mock
* Replace
* Understand

The underlying lesson is:

> Scope determines visibility, but good architecture determines how dependencies should flow.

---

# 32. Senior-Level Mental Model

Don't think:

> "LEGB is just four places where Python searches."

Think:

> **LEGB is Python's name-resolution model for determining which object a name refers to during execution.**

The complete mental model:

```text
                    NAME LOOKUP
                         │
                         ▼
                     Local
                         │
                   not found?
                         ▼
                    Enclosing
                         │
                   not found?
                         ▼
                     Global
                         │
                   not found?
                         ▼
                    Built-in
                         │
                   not found?
                         ▼
                    NameError
```

---

# 33. The Bigger Connection

Your learning is now building a very important chain:

```text
Functions
   │
   ▼
Parameters / Arguments
   │
   ▼
*args / **kwargs
   │
   ▼
Scope / LEGB
   │
   ▼
Nested Functions
   │
   ▼
Higher-Order Functions
   │
   ▼
Closures
   │
   ▼
Decorators
```

Each topic exists for a reason.

You don't need to memorize decorators as an isolated feature.

You understand them because you understand:

```text
Functions are objects
        ↓
Functions can be passed
        ↓
Functions can be returned
        ↓
Nested functions can access enclosing variables
        ↓
Enclosing variables can be retained
        ↓
Wrapper can capture original function
        ↓
Decorator
```

---

# 34. Senior Interview Questions

Before moving to the next topic, you should be able to explain:

1. What is scope in Python?
2. What does LEGB stand for?
3. In what order does Python search for a name?
4. What is local scope?
5. What is enclosing scope?
6. What is global scope?
7. What is built-in scope?
8. What is the difference between scope and namespace?
9. What happens when Python cannot find a name?
10. What is variable shadowing?
11. Why can a local variable shadow a global variable?
12. Why does assignment inside a function affect Python's interpretation of a variable?
13. What causes `UnboundLocalError`?
14. What does the `global` keyword do?
15. What does the `nonlocal` keyword do?
16. What is the difference between `global` and `nonlocal`?
17. Why is enclosing scope important for closures?
18. How does scope help decorators work?
19. Why should global mutable state generally be avoided in production?
20. Explain the LEGB lookup process without memorizing the acronym.

---

# 35. Final Revision Table

| Concept    | Mental Model                                     |
| ---------- | ------------------------------------------------ |
| Scope      | Where a name is accessible/resolved              |
| Namespace  | Mapping of names to objects                      |
| Local      | Current function scope                           |
| Enclosing  | Scope of an outer/nested function                |
| Global     | Module-level scope                               |
| Built-in   | Python-provided built-in names                   |
| LEGB       | Local → Enclosing → Global → Built-in            |
| `global`   | Bind/access the module-level name                |
| `nonlocal` | Bind/access a name in an enclosing function      |
| Shadowing  | Inner binding hides an outer binding             |
| Closure    | Inner function retains access to enclosing state |

---

# 36. Final Mental Model

```text
                    PYTHON NAME
                         │
                         ▼
                       Local
                         │
                         ▼
                     Enclosing
                         │
                         ▼
                       Global
                         │
                         ▼
                      Built-in
                         │
                         ▼
                     NameError
```

For nested functions:

```text
Global
  │
  ▼
outer()
  │
  ├── enclosing variables
  │
  ▼
inner()
  │
  ├── Local
  └── can access Enclosing
```

For closures:

```text
outer()
   │
   ├── variable
   │
   └── inner()
          │
          └── remembers/accesses variable
```

For decorators:

```text
decorator(func)
      │
      ├── func exists in enclosing scope
      │
      ▼
wrapper()
      │
      └── accesses func
```

> **Key takeaway:**
> **LEGB is the foundation for understanding how Python resolves names. Once you understand Local, Enclosing, Global, and Built-in scopes, nested functions and closures stop looking mysterious—and decorators become much easier to reason about.**
