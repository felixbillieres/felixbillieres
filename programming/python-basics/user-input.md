# ☑️ User Input

## Mastering User Input in Python: Empowering Interactivity 🎉💻

### Introduction to User Input:

In Python, user input can transform your programs into dynamic, interactive experiences. Utilizing the `input()` function, Python allows you to collect user responses during runtime, enabling a wide array of applications.

### Gathering User Input:

The `input()` function reads a line from the user and returns it as a string. Let's explore some examples:

```python
# Basic User Input
user_name = input("Enter your name: ")
print("Hello, " + user_name + "!")

# Collecting Numerical Input
user_age = int(input("How old are you? "))
print("You are " + str(user_age) + " years old.")

# Accepting Multiple Inputs
coordinates = input("Enter x and y coordinates (comma-separated): ").split(',')
x, y = map(float, coordinates)
print("Coordinates: x =", x, ", y =", y)
```

### Validating User Input:

It's essential to handle user input carefully, especially when expecting specific data types. Let's see an example of handling potential errors:

```python
# Handling Invalid Input
while True:
    try:
        user_number = float(input("Enter a number: "))
        break  # Exit the loop if input is valid
    except ValueError:
        print("Invalid input. Please enter a valid number.")

print("You entered:", user_number)
```

### Practical Use: Interactive Calculator:

Let's build a simple interactive calculator where the user can perform basic arithmetic operations:

```python
while True:
    # Menu
    print("Select operation:")
    print("1. Add")
    print("2. Subtract")
    print("3. Multiply")
    print("4. Divide")
    print("5. Exit")

    choice = input("Enter choice (1/2/3/4/5): ")

    if choice == '5':
        print("Exiting the calculator. Goodbye!")
        break

    try:
        num1 = float(input("Enter first number: "))
        num2 = float(input("Enter second number: "))
    except ValueError:
        print("Invalid input. Please enter valid numbers.")
        continue

    if choice == '1':
        result = num1 + num2
    elif choice == '2':
        result = num1 - num2
    elif choice == '3':
        result = num1 * num2
    elif choice == '4':
        if num2 != 0:
            result = num1 / num2
        else:
            print("Error: Division by zero.")
            continue
    else:
        print("Invalid choice. Please enter a number between 1 and 5.")
        continue

    print("Result:", result)
```
