---
description: https://www.w3schools.com/c/c_functions.php
---

# 👾 C Functions

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

## C Functions

A function is a block of code that i can call whenever i want and runs only when it's called.

Rather than writing some code multiple times, you write the function once and call it multiple times

I first create this simple function called myFunction:

```
void myFunction() {
  // code to be executed
}
```

`void` means that the function does not have a return value.

and afterward you can call the function once it has a purpose:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

## C Function Parameters

Now, if we wanted to define params and args ->

```
returnType functionName(parameter1, parameter2, parameter3) {
  // code to be executed
}
```

and define the variable once we call our function:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

We can also have multiple args with different value types:

<figure><img src="../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

You can also use arrays through functions:

```
void myFunction(int myNumbers[5]) {
  for(int i = 0; i<5; i++){
  	printf("the number is %d \n", myNumbers[i]);
    }
}

int main() {
  int myNumbers[5] = {1,2,3,40,50};
  myFunction(myNumbers);
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (984).png" alt=""><figcaption></figcaption></figure>

**Functions & return types:**

```
int myFunction(int addNumber) {
  return addNumber + 3;
}

int main() {
  printf("test out the add function: %d\n", myFunction(4));
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (985).png" alt=""><figcaption></figcaption></figure>

And with multiple arguments as well:

```
int multipleArgs(int a, int b) {
  return a *b;
}
```

<figure><img src="../../.gitbook/assets/image (986).png" alt=""><figcaption></figcaption></figure>

or use it to store the result in a variable:

<figure><img src="../../.gitbook/assets/image (987).png" alt=""><figcaption></figcaption></figure>

program that converts a value from Fahrenheit to Celsius:

<figure><img src="../../.gitbook/assets/image (988).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (989).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (990).png" alt=""><figcaption></figcaption></figure>

I did an easy one but the w3school example is pretty nice:

```c
// Function to convert Fahrenheit to Celsius
float toCelsius(float fahrenheit) {
  return (5.0 / 9.0) * (fahrenheit - 32.0);
}

int main() {
  // Set a fahrenheit value
  float f_value = 98.8;

  // Call the function with the fahrenheit value
  float result = toCelsius(f_value);

  // Print the fahrenheit value
  printf("Fahrenheit: %.2f\n", f_value);

  // Print the result
  printf("Convert Fahrenheit to Celsius: %.2f\n", result);

  return 0;
}
```

## C Function Declaration and Definition

A function consist of two parts:

* **Declaration:** the function's name, return type, and parameters (if any)
* **Definition:** the body of the function (code to be executed)

```c
void myFunction() { // declaration
  // the body of the function (definition)
}
```

So the following is not very clear to me. I don't really grasp why I would need to do the following, but it appears that there's a good practice in C that is to declare your function before the main func:

```c
// Function declaration
void myFunction();

// The main method
int main() {
  myFunction();  // call the function
  return 0;
}

// Function definition
void myFunction() {
  printf("I just got executed!");
}
```

## C Recursion

Recursion is the technique of making a function call itself.

<figure><img src="../../.gitbook/assets/image (991).png" alt=""><figcaption></figcaption></figure>

```c
#include <stdio.h>

// Define a function named recursive that takes an integer parameter 'a'
int recursive(int a) {
  // Check if 'a' is less than 10
  if (a < 10) {
    // If 'a' is less than 10, print its value
    printf("the var a is equal to %d\n", a);
    // Call the recursive function with 'a+1' and add its result to 'a'
    return a + recursive(a + 1);
  } else {
    // If 'a' is not less than 10, return 0
    return 0;
  }
}

// Main function
int main() {
  // Call the recursive function with initial value 0 and store the result in 'result'
  int result = recursive(0);
  // Print the final result
  printf("%d", result);
  // Return the result
  return result;
  // This line will never be executed because the previous line already returned from the function
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (992).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.jesuisundev.com/comprendre-la-recursivite-en-7-min/" %}

## C Math Functions

If we write the following header file:

```
#include <math.h>
```

We will be able to access a variety of list of math functions

**square root:**

```
printf("%f", sqrt(16));
```

**Round a number**

`ceil()` function rounds a number upwards to its nearest integer, and the `floor()` method rounds a number downwards

```
printf("%f", ceil(1.4)); //2
printf("%f", floor(1.4)); //1
```

**Power func:**

`pow()` function returns the value of _x_ to the power of _y_ (_xy_):

```
printf("%f", pow(4, 3));
```

and so on:

<figure><img src="../../.gitbook/assets/image (993).png" alt=""><figcaption></figcaption></figure>
