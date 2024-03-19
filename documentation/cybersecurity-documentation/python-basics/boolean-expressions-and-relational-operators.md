# ⛔ Boolean Expressions and Relational Operators

## Boolean Expressions and Relational Operators in Python 🚀🔍

Boolean expressions in Python involve the use of relational operators to compare values and produce Boolean results (`True` or `False`). Let's dive into a documentation section:

### Relational Operators:

#### 1. Equality (`==`)

Checks if two values are equal.

**Example:**

```python
result = 5 == 5
print(result)  # Output: True

word1 = "hello"
word2 = "world"
result = word1 == word2
print(result)  # Output: False
```

#### 2. Inequality (`!=`)

Checks if two values are not equal.

**Example:**

```python
result = 5 != 10
print(result)  # Output: True

name1 = "Alice"
name2 = "Bob"
result = name1 != name2
print(result)  # Output: True
```

#### 3. Greater than (`>`)

Checks if the value on the left is greater than the value on the right.

**Example:**

```python
result = 10 > 5
print(result)  # Output: True

price1 = 25
price2 = 30
result = price1 > price2
print(result)  # Output: False
```

#### 4. Less than (`<`)

Checks if the value on the left is less than the value on the right.

**Example:**

```python
result = 15 < 20
print(result)  # Output: True

quantity1 = 50
quantity2 = 40
result = quantity1 < quantity2
print(result)  # Output: False
```

#### 5. Greater than or Equal to (`>=`)

Checks if the value on the left is greater than or equal to the value on the right.

**Example:**

```python
result = 8 >= 8
print(result)  # Output: True

marks1 = 90
marks2 = 85
result = marks1 >= marks2
print(result)  # Output: True
```

#### 6. Less than or Equal to (`<=`)

Checks if the value on the left is less than or equal to the value on the right.

**Example:**

```python
result = 25 <= 30
print(result)  # Output: True

age1 = 30
age2 = 35
result = age1 <= age2
print(result)  # Output: True
```

### Boolean Expressions:

Boolean expressions combine multiple relational expressions using logical operators (`and`, `or`, `not`) to form complex conditions.

#### Logical AND (`and`):

Combines two conditions. The result is `True` only if both conditions are `True`.

**Example:**

```python
x = 15
y = 20

result = (x > 10) and (y < 30)
print(result)  # Output: True
```

#### Logical OR (`or`):

Combines two conditions. The result is `True` if at least one condition is `True`.

**Example:**

```python
x = 15
y = 20

result = (x > 10) or (y > 30)
print(result)  # Output: True
```

#### Logical NOT (`not`):

Negates the result of a condition.

**Example:**

```python
x = 15

result = not (x == 20)
print(result)  # Output: True
```

