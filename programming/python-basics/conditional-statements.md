# 🤚 Conditional Statements

## Conditional Statements in Python 🌐🔀

Conditional statements in Python allow you to control the flow of your program based on certain conditions. These statements include `if`, `elif` (else if), and `else`. Let's explore how to use them effectively:

### The `if` Statement:

The `if` statement is used to execute a block of code only if a specified condition is `True`.

```python
x = 10

if x > 5:
    print("x is greater than 5")
```

In this example, the indented block under the `if` statement is executed only if the condition `x > 5` is `True`.

### The `elif` Statement:

The `elif` statement allows you to check multiple conditions after the initial `if` statement. It is short for "else if."

```python
y = 20

if y > 25:
    print("y is greater than 25")
elif y > 15:
    print("y is greater than 15 but not 25")
else:
    print("y is 15 or less")
```

Here, the program checks conditions one by one. If the first condition is not met, it proceeds to the next `elif` condition, and finally, the `else` block is executed if none of the conditions is `True`.

### The `else` Statement:

The `else` statement is used to execute a block of code if none of the preceding conditions (in `if` and `elif`) is `True`.

```python
z = 5

if z > 10:
    print("z is greater than 10")
else:
    print("z is 10 or less")
```

In this case, if `z > 10` is not `True`, the code under the `else` block is executed.

### Combining Conditions:

You can use logical operators (`and`, `or`, `not`) to combine conditions.

```python
a = 15

if a > 10 and a < 20:
    print("a is between 10 and 20")
elif a >= 20 or a < 5:
    print("a is either greater than or equal to 20, or less than 5")
```

Here, the program checks if `a` is between 10 and 20 in the first `if` condition. If not, it checks the second `elif` condition.
