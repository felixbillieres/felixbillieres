# 👷 Functions

## Python Functions 🐍🔧

In Python, a **function** is a reusable block of code designed to perform a specific task. Functions are crucial for organizing code into logical and modular units, enhancing readability, maintainability, and reusability. Here's an overview of functions in Python:

### Function Definition:

A function is defined using the `def` keyword, followed by the function name, parentheses, and a colon. Optional components include parameters and a return statement. Below is an example of a simple function definition:

```python
def greet():
    print("Hello, World!")
```

### Function Call:

To execute a function, call it by its name followed by parentheses. For instance, calling the `greet()` function is done as follows:

```python
greet()
```

### Function Parameters:

Functions can accept parameters, which are variables holding values passed during the function call. Parameters customize the function's behavior based on provided values. Here's an example:

```python
def greet(name):
    print("Hello, " + name + "!")

greet("Alice")
```

In this example, the `greet()` function accepts a parameter named `name`.

### Return Statement:

Functions can return values using the `return` statement. The returned value can be assigned to a variable or used directly. Example:

```python
def add_numbers(a, b):
    return a + b

result = add_numbers(3, 4)
print(result)  # Output: 7
```

Here, the `add_numbers()` function returns the sum of its two parameters.

### Functions Example:

```python
# Functions
print("Here is an example function:")

def who_am_i():  # Function without parameters
    name = "Felix"
    age = 30  # Local variable
    print("My name is " + name + " and I am " + str(age) + " years old.")

who_am_i()

# Adding parameters
def add_one_hundred(num):
    print(num + 100)

add_one_hundred(100)

# Multiple parameters
def add(x, y):
    print(x + y)

add(7, 7)

def multiply(x, y):
    return x * y

multiply_result = multiply(7, 7)
print(multiply_result)

def square_root(x):
    print(x ** 0.5)

square_root(64)

def nl():
    print('\n')

nl()
```
