# Python Higher-Order Functions

> **Learning level:** Intermediate → Senior
> **Purpose:** Understand what higher-order functions are, how Python passes and returns functions, and why this concept is fundamental to callbacks, closures, decorators, and functional programming patterns.

---

## 1. What is it?

A **higher-order function** is a function that does at least one of these:

1. Accepts another function as an argument.
2. Returns another function as its result.

Example:

```python
def execute(func):
    return func()
```

Here `execute()` accepts a function.

Therefore, `execute()` is a higher-order function.

Another example:

```python
def create_function():

    def inner():
        print("Hello")

    return inner
```

Here `create_function()` returns a function.

So it is also a higher-order function.

### Core idea

```text
Function
   ↓
Function treated as an object
   ↓
Passed / returned
   ↓
Higher-Order Function
```

---

# 2. Why does it exist?

Higher-order functions allow us to make behavior **dynamic and composable**.

Instead of hard-coding what a function should execute, we can pass the behavior into it.

For example:

```python
def execute(func):
    func()
```

Now:

```python
def greet():
    print("Hello")


def bye():
    print("Bye")
```

We can choose the behavior:

```python
execute(greet)
execute(bye)
```

Output:

```text
Hello
Bye
```

The `execute()` function doesn't need to know what `func` does.

It only knows:

> "I received something callable, so I can call it."

This is a form of **behavior abstraction**.

---

# 3. How does it work internally?

Remember the most important concept from the Functions topic:

> **Functions are objects.**

Consider:

```python
def greet():
    print("Hello")
```

Python creates a function object.

The name:

```python
greet
```

references that object.

Therefore:

```python
execute(greet)
```

doesn't immediately execute `greet()`.

Instead:

```text
execute(greet)
       │
       ▼
pass reference to function object
       │
       ▼
func = greet
       │
       ▼
func()
       │
       ▼
execute greet
```

This distinction is critical:

```python
execute(greet)
```

means:

> Pass the function.

While:

```python
execute(greet())
```

means:

> Execute `greet()` first and pass its return value.

---

# 4. Core concepts

## 4.1 Functions are first-class objects

Python allows functions to be:

* Assigned to variables
* Passed as arguments
* Returned from functions
* Stored in collections
* Used as dictionary values
* Wrapped by other functions

Example:

```python
def greet():
    print("Hello")

x = greet
```

Now:

```text
greet ─────┐
           │
           ▼
      Function Object
           ▲
           │
x ─────────┘
```

Therefore:

```python
x()
```

calls the same function.

---

## 4.2 Passing a function as an argument

Example:

```python
def greet():
    print("Hello")


def execute(func):
    func()


execute(greet)
```

Flow:

```text
greet
  │
  │ function reference
  ▼
execute(func)
  │
  ▼
func()
  │
  ▼
greet()
```

The function `execute()` controls **when** the behavior runs, while `greet()` defines **what** the behavior is.

---

## 4.3 Returning a function

Example:

```python
def outer():

    def inner():
        print("Hello")

    return inner
```

Then:

```python
result = outer()
```

Now:

```text
result ─────► inner function object
```

Calling:

```python
result()
```

executes `inner()`.

---

## 4.4 Functions can be stored in data structures

Example:

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b


operations = {
    "add": add,
    "multiply": multiply
}
```

Now:

```python
operations["add"](10, 20)
```

calls:

```python
add(10, 20)
```

And:

```python
operations["multiply"](10, 20)
```

calls:

```python
multiply(10, 20)
```

This is another consequence of functions being first-class objects.

---

# 5. Execution flow

Consider:

```python
def greet():
    print("Hello")


def execute(func):
    print("Before")
    func()
    print("After")


execute(greet)
```

Execution flow:

```text
1. Python creates greet function object
          ↓
2. Python creates execute function object
          ↓
3. execute(greet)
          ↓
4. greet function reference is passed
          ↓
5. func refers to greet
          ↓
6. print("Before")
          ↓
7. func()
          ↓
8. greet() executes
          ↓
9. print("After")
          ↓
10. execute() returns
```

Output:

```text
Before
Hello
After
```

---

# 6. Function as Data vs Function as Behavior

A normal value might represent data:

```python
name = "Chinnu"
```

A function can represent **behavior**:

```python
def greet():
    print("Hello")
```

Therefore:

```text
Data
 ↓
"Chinnu"

Behavior
 ↓
greet function
```

Higher-order functions allow us to pass behavior around.

This is a powerful abstraction.

---

# 7. Higher-Order Function vs First-Class Function

These concepts are related but not identical.

### First-class function

Describes the **capability of the language**.

Python allows functions to be treated as objects.

### Higher-order function

Describes a **function that uses functions**.

Example:

```python
def execute(func):
    func()
```

So:

```text
First-class functions
        ↓
make higher-order functions possible
```

A common interview mistake is treating these two terms as synonyms.

They are related, but they describe different things.

---

# 8. Callback Functions

A function passed to another function is often called a **callback** when the receiving function controls when it will be executed.

Example:

```python
def on_success():
    print("Operation completed")


def process(callback):
    print("Processing...")
    callback()


process(on_success)
```

Here:

```text
process()
   │
   ├── performs operation
   │
   └── calls callback
```

`on_success` is the callback.

The important idea is:

> The caller provides behavior; the receiving function decides when to invoke it.

---

# 9. Why callbacks are useful

Callbacks allow behavior to be customized without modifying the function performing the main operation.

Example:

```python
def process(data, callback):
    result = data * 2
    callback(result)
```

Different callers can provide different behavior:

```python
def print_result(result):
    print(result)


def save_result(result):
    print("Saving:", result)
```

Then:

```python
process(10, print_result)
process(10, save_result)
```

The processing logic remains unchanged.

Only the callback changes.

---

# 10. Higher-Order Functions and Abstraction

Consider:

```python
def calculate(a, b, operation):
    return operation(a, b)
```

Now:

```python
def add(a, b):
    return a + b


def multiply(a, b):
    return a * b
```

We can do:

```python
calculate(10, 20, add)
calculate(10, 20, multiply)
```

The generic function:

```python
calculate()
```

doesn't need separate implementations for addition and multiplication.

It delegates the specific behavior to:

```python
operation
```

This is a powerful form of **behavior injection**.

---

# 11. Behavior Injection

Instead of:

```python
def calculate_addition(a, b):
    return a + b


def calculate_multiplication(a, b):
    return a * b
```

we can create:

```python
def calculate(a, b, operation):
    return operation(a, b)
```

Now the behavior is supplied from outside.

Conceptually:

```text
                 calculate()
                     │
          ┌──────────┴──────────┐
          │                     │
        add                  multiply
          │                     │
          └──────────┬──────────┘
                     ▼
                 operation
```

This pattern appears throughout real software systems.

---

# 12. Returning Functions Dynamically

A higher-order function can create different functions.

Example:

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

Then:

```python
double(10)
```

returns:

```text
20
```

And:

```python
triple(10)
```

returns:

```text
30
```

This is both:

* A higher-order function
* A nested function
* A closure

This demonstrates how your previous topics connect.

---

# 13. Higher-Order Functions + Closures

The previous example:

```python
def create_multiplier(number):

    def multiply(value):
        return value * number

    return multiply
```

has three important concepts.

### 1. Nested function

`multiply()` is inside `create_multiplier()`.

### 2. Higher-order function

`create_multiplier()` returns a function.

### 3. Closure

`multiply()` remembers `number`.

Therefore:

```text
Higher-Order Function
        │
        ▼
Nested Function
        │
        ▼
Enclosing Variable
        │
        ▼
Closure
```

---

# 14. Higher-Order Functions + `*args` and `**kwargs`

You previously learned:

```python
def wrapper(*args, **kwargs):
    return func(*args, **kwargs)
```

This is a higher-order pattern because:

* `wrapper` receives a function indirectly through `func`
* The function is called dynamically
* Arguments are forwarded

Example:

```python
def execute(func, *args, **kwargs):
    return func(*args, **kwargs)
```

Now:

```python
def add(a, b):
    return a + b
```

We can call:

```python
result = execute(add, 10, 20)
```

Flow:

```text
execute()
   │
   ├── func = add
   ├── args = (10, 20)
   └── kwargs = {}
           │
           ▼
    func(*args, **kwargs)
           │
           ▼
        add(10, 20)
```

This pattern becomes extremely important for decorators.

---

# 15. Higher-Order Functions and Built-in Python

Python provides several built-in functions that operate on other functions.

Examples include:

```python
map()
filter()
sorted()
```

For example:

```python
numbers = [1, 2, 3, 4]

result = map(lambda x: x * 2, numbers)
```

Here `map()` receives a function.

Similarly:

```python
numbers = [1, 2, 3, 4]

result = filter(lambda x: x % 2 == 0, numbers)
```

`filter()` receives a function that determines which values should remain.

`sorted()` can accept a key function:

```python
users = [
    {"name": "A", "age": 30},
    {"name": "B", "age": 20}
]

sorted(users, key=lambda user: user["age"])
```

The `key` function defines the behavior used for comparison.

---

# 16. Higher-Order Functions and `lambda`

Lambda functions are often used with higher-order functions.

Example:

```python
numbers = [1, 2, 3, 4]

result = list(
    map(lambda x: x * 2, numbers)
)
```

The lambda is passed as a function object.

Conceptually:

```text
lambda
  │
  ▼
Function Object
  │
  ▼
map()
  │
  ▼
Apply function to each element
```

However, lambda functions are not required for higher-order functions.

Normal functions work equally well.

---

# 17. Higher-Order Functions and Decorators

Decorators are one of the most important real-world applications of higher-order functions.

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
   ├── receives function
   │
   ├── creates function
   │
   └── returns function
```

Therefore the decorator is a higher-order function.

The chain is:

```text
Function is object
       ↓
Function passed as argument
       ↓
Function returned
       ↓
Higher-Order Function
       ↓
Nested wrapper
       ↓
Closure
       ↓
Decorator
```

---

# 18. Decorator Transformation

Consider:

```python
@decorator
def greet():
    print("Hello")
```

Conceptually:

```python
def greet():
    print("Hello")

greet = decorator(greet)
```

This is one of the most important mental models for decorators.

Before decoration:

```text
greet
  ↓
original function
```

After decoration:

```text
greet
  ↓
wrapper function
  ↓
original function
```

This transformation is possible because functions are first-class objects.

---

# 19. Higher-Order Functions vs Normal Functions

Normal function:

```python
def add(a, b):
    return a + b
```

It operates on data:

```text
a + b
```

Higher-order function:

```python
def execute(func):
    return func()
```

It operates on behavior:

```text
function → function execution
```

The distinction isn't that one is "better".

Higher-order functions are useful when the behavior itself needs to be configurable.

---

# 20. Function Composition

Higher-order functions allow us to compose behavior.

Example:

```python
def double(x):
    return x * 2


def square(x):
    return x * x
```

We can create:

```python
def compose(f, g):

    def result(x):
        return f(g(x))

    return result
```

Then:

```python
double_square = compose(double, square)
```

Calling:

```python
double_square(3)
```

produces:

```text
3
 ↓
square(3)
 ↓
9
 ↓
double(9)
 ↓
18
```

This is function composition.

---

# 21. Execution Flow of Function Composition

For:

```python
compose(double, square)
```

the flow is:

```text
compose()
   │
   ├── f = double
   └── g = square
          │
          ▼
     create result()
          │
          ▼
     return result
```

Then:

```python
double_square(3)
```

becomes:

```text
result(3)
   ↓
g(3)
   ↓
square(3)
   ↓
9
   ↓
f(9)
   ↓
double(9)
   ↓
18
```

This is another example of:

* Higher-order function
* Nested function
* Closure

---

# 22. Common Mistakes

## Mistake 1 — Confusing function and function call

Incorrect mental model:

```python
execute(greet())
```

when you want to pass `greet`.

Correct:

```python
execute(greet)
```

---

## Mistake 2 — Thinking every function receiving arguments is higher-order

This:

```python
def add(a, b):
    return a + b
```

is not higher-order merely because it receives arguments.

A higher-order function receives or returns **functions**.

---

## Mistake 3 — Thinking higher-order functions require `lambda`

They don't.

This is completely valid:

```python
def square(x):
    return x * x


def apply(func, value):
    return func(value)
```

---

## Mistake 4 — Confusing first-class functions and higher-order functions

Remember:

```text
First-class function
→ capability of treating functions as objects

Higher-order function
→ function that accepts/returns functions
```

---

## Mistake 5 — Calling the function too early

Wrong:

```python
execute(greet())
```

Correct:

```python
execute(greet)
```

The first executes `greet()` immediately.

The second passes the function object.

---

# 23. Production Perspective

Higher-order functions appear in many production patterns:

* Decorators
* Middleware
* Callbacks
* Event handlers
* Retry wrappers
* Authentication wrappers
* Logging wrappers
* Validation
* Function factories
* Dependency injection
* Strategy patterns
* Data processing pipelines

For example, instead of:

```python
if operation == "add":
    ...


elif operation == "multiply":
    ...
```

you can sometimes use a mapping:

```python
operations = {
    "add": add,
    "multiply": multiply
}
```

and dynamically select behavior.

This can make systems more extensible when designed carefully.

---

# 24. Higher-Order Functions and Strategy Pattern

A simple strategy-style design:

```python
def process(data, strategy):
    return strategy(data)
```

Different strategies:

```python
def fast_strategy(data):
    ...


def accurate_strategy(data):
    ...
```

Then:

```python
process(data, fast_strategy)
process(data, accurate_strategy)
```

The processing function doesn't need to know the implementation details of each strategy.

This is essentially **behavior supplied as an object**.

---

# 25. Production Design Consideration

Higher-order functions are powerful, but abstraction should serve the problem.

Don't create:

```python
process(data, strategy, callback, transformer, validator)
```

just because Python allows it.

Too much function passing can make control flow difficult to understand.

Senior engineering asks:

> Does passing behavior improve flexibility and separation of concerns, or does it make the code harder to follow?

Use abstraction intentionally.

---

# 26. Higher-Order Functions and Dependency Injection

Consider:

```python
def service(repository):
    return repository.get_users()
```

The service doesn't create its repository internally.

The behavior/dependency is supplied from outside.

Conceptually:

```text
service
  │
  ▼
repository
  │
  ▼
get_users()
```

This makes testing easier because a different implementation can be supplied.

For example:

```python
service(mock_repository)
```

This is related to the broader idea of **dependency injection**.

---

# 27. Higher-Order Functions and Testability

Suppose:

```python
def process(data, save):
    result = transform(data)
    save(result)
```

Testing can provide a fake `save` function:

```python
def fake_save(data):
    print("Test save:", data)
```

Then:

```python
process(data, fake_save)
```

The processing logic doesn't need to know whether the real database or a test implementation is being used.

This is one practical advantage of passing behavior explicitly.

---

# 28. Higher-Order Functions and Callbacks in Systems

A common production flow is:

```text
Operation starts
     ↓
Do work
     ↓
Success?
     │
     ├── Yes → success callback
     │
     └── No  → failure callback
```

Example:

```python
def process(success, failure):

    try:
        result = perform_operation()
        success(result)

    except Exception as e:
        failure(e)
```

The function controls the execution flow while callers supply the behavior.

---

# 29. Senior-Level Mental Model

Don't think:

> "Higher-order function means a function inside another function."

That describes a nested function, not necessarily a higher-order function.

Instead think:

> **A higher-order function treats behavior as data by accepting functions, returning functions, or both.**

The key distinction:

```text
                 FUNCTION
                    │
                    ▼
             Function Object
                    │
          ┌─────────┴─────────┐
          │                   │
       passed              returned
          │                   │
          └─────────┬─────────┘
                    ▼
          Higher-Order Function
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
       Callback           Function Factory
          │                   │
          └─────────┬─────────┘
                    ▼
                 Closure
                    │
                    ▼
                Decorator
```

---

# 30. Important Relationships

The topics you have learned are now connected:

```text
FUNCTION
   │
   ▼
Functions are objects
   │
   ▼
First-class functions
   │
   ├───────────────┐
   ▼               ▼
Pass functions    Return functions
   │               │
   └───────┬───────┘
           ▼
   Higher-Order Functions
           │
           ├───────────────┐
           ▼               ▼
      Callbacks       Function Factory
                           │
                           ▼
                    Nested Function
                           │
                           ▼
                    Enclosing Scope
                           │
                           ▼
                        Closure
                           │
                           ▼
                       Decorator
```

This relationship is more important than memorizing the definition.

---

# 31. Common Interview Confusion

### Question:

> Is every function that returns a value a higher-order function?

No.

```python
def add(a, b):
    return a + b
```

returns a value, but that value isn't a function.

A higher-order function must accept or return a **function**.

---

### Question:

> Is every nested function a higher-order function?

No.

```python
def outer():

    def inner():
        print("Hello")

    inner()
```

`outer()` contains a nested function, but it doesn't accept or return a function.

If:

```python
return inner
```

then `outer()` becomes a higher-order function.

---

# 32. Senior Interview Questions

Before moving to the next topic, you should be able to answer:

1. What is a higher-order function?
2. Why are functions called first-class objects in Python?
3. What is the difference between a first-class function and a higher-order function?
4. How can a function be passed as an argument?
5. How can a function return another function?
6. What is a callback?
7. Why are callbacks useful?
8. What is behavior injection?
9. What is function composition?
10. How can higher-order functions help implement the Strategy pattern?
11. How are higher-order functions related to closures?
12. How are higher-order functions related to decorators?
13. Why does `execute(greet)` differ from `execute(greet())`?
14. Can a nested function be a higher-order function?
15. Is every nested function a higher-order function?
16. Is every function that accepts arguments a higher-order function?
17. How are `*args` and `**kwargs` useful with higher-order functions?
18. How can higher-order functions improve testability?
19. What are the disadvantages of excessive function passing?
20. Explain this complete chain:

```text
Functions
→ First-Class Functions
→ Higher-Order Functions
→ Nested Functions
→ Closures
→ Decorators
```

---

# 33. Final Revision

| Concept               | Mental Model                                    |
| --------------------- | ----------------------------------------------- |
| First-class function  | Function can be treated as an object            |
| Higher-order function | Function accepting/returning functions          |
| Callback              | Function passed for later execution             |
| Behavior injection    | Supplying behavior from outside                 |
| Function factory      | Function that creates/returns functions         |
| Function composition  | Combining functions to create new behavior      |
| Closure               | Function retaining enclosing state              |
| Decorator             | Function that transforms/wraps another callable |
| Strategy              | Supplying interchangeable behavior              |
| Argument forwarding   | Passing received arguments to another function  |

---

# 34. Final Mental Model

```text
                    PYTHON FUNCTION
                           │
                           ▼
                   Function Object
                           │
                           ▼
                 First-Class Function
                           │
              ┌────────────┴────────────┐
              │                         │
              ▼                         ▼
       Pass as argument           Return function
              │                         │
              ▼                         ▼
          Callback               Function Factory
              │                         │
              └────────────┬────────────┘
                           ▼
                 Higher-Order Function
                           │
                           ▼
                   Nested Function
                           │
                           ▼
                    Enclosing Scope
                           │
                           ▼
                        Closure
                           │
                           ▼
                       Decorator
```

> **Key takeaway:**
> **A higher-order function treats behavior as something that can be passed around just like data. Python's first-class function model makes this possible, and this capability becomes the foundation for callbacks, function factories, closures, and decorators.**
