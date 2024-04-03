# 🧑‍🏭 Variables and Methods

## Python Variables and Methods 🐍🔧

### Variables:

A **variable** is a named storage location used to store data or values in a program. It acts as a placeholder for data that can be accessed, modified, or used in calculations throughout the program. Variables in Python are dynamically typed, meaning their data type can change during program execution.

#### Example:

```python
# Variable assignment
x = 10
name = "John"
is_true = True

# Variable usage
y = x + 5
print("Hello, " + name)
if is_true:
    print("The condition is true")
```

In the example above, `x`, `name`, and `is_true` are variables assigned with different data types (integer, string, and boolean, respectively). They are used in calculations and print statements to perform operations and display values.

### Methods (Functions):

A **method** is a block of reusable code that performs a specific task or action. Methods are associated with objects or classes and are called upon to perform certain operations. In Python, methods are commonly referred to as functions. Built-in functions and user-defined functions both fall under the category of methods.

#### Example:

```python
# Built-in method example
numbers = [1, 2, 3, 4, 5]
length = len(numbers)
print("Length:", length)

# User-defined method example
def greet(name):
    print("Hello, " + name)

greet("Alice")
```

In the example above, `len()` is a built-in method that calculates the length of a list (`numbers` in this case). The user-defined method `greet()` takes a parameter `name` and prints a greeting message. It is called with the argument "Alice" to print "Hello, Alice" to the console.

#### Python Code Example:

```python
# Variables and Methods
quote = "All is fair in love and war."
print(quote)

print(quote.upper())  # Uppercase
print(quote.lower())  # Lowercase
print(quote.title())  # Title case
print(len(quote))  # Counts characters

name = "Heath"  # String
age = 33  # Int
gpa = 3.7  # Float - has a decimal

print(int(age))
print(int(30.1))
# Will it round? No!
print(int(30.9))

print("My name is " + name + " and I am " + str(age) + " years old.")

age += 1
print(age)

birthday = 1
age += birthday
print(age)
```

Variables and methods are essential components in Python programming. Variables store data, while methods encapsulate reusable blocks of code for specific tasks. Understanding their usage and relationship is crucial for building functional and efficient programs. 🚀📊
