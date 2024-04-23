# ➕ Math

## Python Math Basics 🐍➕➖➗✖️

### Arithmetic Operations:

Python supports standard arithmetic operations:

* Addition (`+`)
* Subtraction (`-`)
* Multiplication (`*`)
* Division (`/`)
* Exponentiation (`**`)
* Modulo (`%`) - Returns the remainder of the division.

### Examples:

```python
# Addition
sum_result = 5 + 3
print("Addition:", sum_result)

# Subtraction
difference = 8 - 3
print("Subtraction:", difference)

# Multiplication
product = 4 * 6
print("Multiplication:", product)

# Division
quotient = 10 / 2
print("Division:", quotient)

# Exponentiation
power_result = 2 ** 3
print("Exponentiation:", power_result)

# Modulo
remainder = 15 % 7
print("Modulo:", remainder)
```

### Math Functions:

Python provides built-in math functions through the `math` module:

```python
import math

# Square root
sqrt_result = math.sqrt(25)
print("Square Root:", sqrt_result)

# Trigonometric functions (sin, cos, tan)
angle_in_radians = math.radians(45)
sin_value = math.sin(angle_in_radians)
print("Sine Value:", sin_value)
```

### Random Numbers:

Generate random numbers with the `random` module:

```python
import random

# Random float between 0 and 1
random_float = random.random()
print("Random Float:", random_float)

# Random integer within a range
random_integer = random.randint(1, 10)
print("Random Integer:", random_integer)
```

### Math Functions:

Python provides built-in math functions through the `math` module:

```python
import math

# Square root
sqrt_result = math.sqrt(25)
print("Square Root:", sqrt_result)

# Trigonometric functions (sin, cos, tan)
angle_in_radians = math.radians(45)
sin_value = math.sin(angle_in_radians)
print("Sine Value:", sin_value)

# Additional Math Functions
power_value = math.pow(2, 5)
print("Power Value:", power_value)

exp_result = math.exp(2)
print("Exponential Value:", exp_result)

log_value = math.log(10)
print("Natural Logarithm:", log_value)

log10_value = math.log10(100)
print("Logarithm (Base 10):", log10_value)

# ... and more

```

### Practical Example: Calculating the Hypotenuse of a Right Triangle:

```python
# Calculate the hypotenuse of a right-angled triangle using the Pythagorean theorem
a = 3
b = 4

hypotenuse = math.sqrt(a**2 + b**2)
print("Hypotenuse:", hypotenuse)
```
