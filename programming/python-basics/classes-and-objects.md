# 🏗️ Classes and Objects

## Dive into the World of Python Classes and Objects 🚀🐍

### Introduction:

Python, an object-oriented programming language, thrives on the concept of classes and objects. This paradigm provides a powerful way to structure and organize code, promoting reusability and modularity. Let's embark on a journey to understand classes and objects in Python!

### Classes:

A class is a blueprint for creating objects. It defines properties (attributes) and behaviors (methods) that the objects of the class will have. Here's a basic class definition:

```python
# Class Definition
class Car:
    # Class Attribute
    category = "Automobile"

    # Constructor (Initializer) Method
    def __init__(self, brand, model):
        # Instance Attributes
        self.brand = brand
        self.model = model

    # Instance Method
    def display_info(self):
        print(f"{self.brand} {self.model} - {self.category}")
```

In this example, `Car` is a class with a class attribute (`category`) and an initializer method (`__init__`) to set instance attributes (`brand` and `model`). The `display_info` method is an instance method.

### Objects:

An object is an instance of a class. It represents a real-world entity based on the class's blueprint. Let's create and use a `Car` object:

```python
# Creating Objects
car1 = Car("Toyota", "Camry")
car2 = Car("Tesla", "Model 3")

# Accessing Attributes
print(car1.brand)  # Output: Toyota

# Calling Methods
car1.display_info()  # Output: Toyota Camry - Automobile
```

Here, `car1` and `car2` are instances (objects) of the `Car` class. You can access their attributes (`brand`) and call their methods (`display_info`).

### Inheritance:

Inheritance allows a class (subclass) to inherit attributes and methods from another class (superclass). It fosters code reuse and hierarchy. Let's explore inheritance:

```python
# Inheritance
class ElectricCar(Car):
    # Additional Attribute
    battery_capacity = "60 kWh"

    # Overriding Method
    def display_info(self):
        print(f"{self.brand} {self.model} (Electric) - {self.category}")
        print(f"Battery Capacity: {self.battery_capacity}")
```

`ElectricCar` is a subclass of `Car` that inherits attributes and methods. It introduces a new attribute (`battery_capacity`) and overrides the `display_info` method.

### Encapsulation:

Encapsulation restricts access to some of an object's components. It helps prevent unintended interference with internal state. Attributes and methods can be public, protected, or private.

* Public: Accessible from anywhere (default).
* Protected: Accessible within the class and its subclasses (denoted by a single underscore `_`).
* Private: Accessible only within the class (denoted by two underscores `__`).

```python
# Encapsulation Example
class Person:
    def __init__(self, name, age):
        self._name = name  # Protected Attribute
        self.__age = age   # Private Attribute

    def get_age(self):
        return self.__age  # Getter Method

    def set_age(self, new_age):
        if new_age > 0:
            self.__age = new_age  # Setter Method
```

In this example, `name` is public, `_name` is protected, and `__age` is private. Getter and setter methods allow controlled access to private attributes.
