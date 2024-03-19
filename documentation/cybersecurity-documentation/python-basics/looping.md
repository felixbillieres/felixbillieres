# 🔂 Looping

## Looping in Python: Unleash the Power of Repetition! 🔄💪

### For Loop: Conquering Sequences 🚀

The **for loop** in Python is your go-to tool for iterating over sequences like lists, tuples, or strings. It helps you perform the same action for each item in the sequence.

#### Basic Syntax:

```python
fruits = ["apple", "banana", "orange"]
for fruit in fruits:
    print(fruit)
```

In this example, the loop iterates through each fruit in the list and prints it.

#### Range Function:

```python
for number in range(1, 5):
    print(number)
```

The `range()` function generates a sequence of numbers, and the loop iterates over them.

### While Loop: Mastering Conditions 🔄🕰️

The **while loop** is your companion when you need to repeat a block of code as long as a certain condition is true.

#### Basic Syntax:

```python
counter = 0
while counter < 3:
    print("Counting:", counter)
    counter += 1
```

This loop counts from 0 to 2 based on the condition `counter < 3`.

#### Infinite Loop Caution:

```python
while True:
    print("This is an infinite loop!")
```

Be cautious with `while` loops to avoid unintentional infinite loops.

### Loop Control Statements ⚔️

#### Break Statement:

```python
for number in range(1, 10):
    if number == 5:
        break
    print(number)
```

`break` terminates the loop prematurely when a condition is met.

#### Continue Statement:

```python
for number in range(1, 6):
    if number == 3:
        continue
    print(number)
```

`continue` skips the rest of the loop's code for a specific iteration.

### Nested Loops: The Inception 🔄🔄

```python
for i in range(3):
    for j in range(2):
        print(f"({i}, {j})")
```

You can nest loops to iterate through multiple dimensions.
