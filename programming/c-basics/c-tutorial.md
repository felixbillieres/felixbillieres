# 🆘 C Tutorial

The main difference between C and C++ is that C++ support classes and objects, while C does not

<figure><img src="../../.gitbook/assets/image (852).png" alt=""><figcaption></figcaption></figure>

Quick syntax review of a hello world:

```
#include <stdio.h>

int main(){
    printf("hello world");
    return 0;
}
```

`#include <stdio.h>` is a **header file library** that lets us work with input and output functions, such as `printf()`

Another thing that always appear in a C program is `main()`. This is called a **function**. Any code inside its curly brackets `{}` will be executed

`printf()` is a **function** used to output/print text to the screen.

**Note that:** Every C statement ends with a semicolon `;`

To insert a new line, you can use the **\n** character:

```
#include <stdio.h>

int main() {
  printf("Hello World!\nI am learning C.\nAnd it is awesome!");
  return 0;
} 
```

Single-line comments start with two forward slashes (`//`).

Multi-line comments start with `/*` and ends with `*/`.

## C Variables

**different types of variables in C:**

* `int` - stores integers (whole numbers), without decimals, such as `123` or `-123`
* `float` - stores floating point numbers, with decimals, such as `19.99` or `-19.99`
* `char` - stores single characters, such as `'a'` or `'B'`. Characters are surrounded by **single quotes**

To create a variable, we must specify the **type** and assign it a **value**:

```
 type variableName = value; 
```

Variable called theVarInt, type int, and it's value is 24:

```
int theVarInt = 24;
```

or we can assign value later:

```
//declare var:
int theVarInt;

//assign value:
theVarInt = 24;
```

if we wanted to print out the value assigned to _theVarInt_ it would not be a `printf(theVarInt);`

#### C Format Specifiers

Format specifiers are used together with the `printf()` function to tell the compiler what type of data the variable is storing.

int would use the format specifier `%d` surrounded by double quotes (`""`), inside the `printf()` function:

```
int theVarInt = 24;
printf("%d", theVarInt); 
```

(`%c` for `char` and `%f` for `float)`

<figure><img src="../../.gitbook/assets/image (854).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (855).png" alt=""><figcaption></figcaption></figure>

To combine both text and a variable, separate them with a comma inside the `printf()` function:

```
printf("i was born on june: %d", theInt); // June 24 ;)
```

And to print different type in a same printf:

```
printf("i was born on june %d and pi is equal to %f ", theInt, theFloat);
```

#### Change Variable Values

If you assign a new value to an existing variable, it will **overwrite** the previous value:

```
int myNum = 15;  // myNum is 15
myNum = 10;  // Now myNum is now 10 
```

you can also assign a value using another variable:

```
int myNum = 15;

int myOtherNum = 23;

// Assign the value of myOtherNum (23) to myNum
myNum = myOtherNum;
```

#### Add Variables Together

Using the + operator:

```
int x = 5;
int y = 6;
int sum = x + y;
printf("%d", sum); 
```

#### Declare Multiple Variables

if we want to declare multiple (same type) variable, we can do it in one line with comma separated values:

```
int x = 5, y = 6, z = 50;
printf("%d", x + y + z); 
```

&#x20;or same value:

```
int x, y, z;
x = y = z = 50;
printf("%d", x + y + z); 
```

**Real life example: calculate area of rectangle:**

```
#include <stdio.h>

int main(){
  int a = 2;
  int b = 13;
  float area;
  
  area = a * b;
  printf("the area is: %f", area );
  return 0;
}

//the area is: 26.000000
```

#### C Character Data Types

```
char myGrade = 'A';
printf("%c", myGrade); 
```

But we can also use ASCII to print out char values:

```
char a = 65;
printf("%c", a)
//prints A
```

{% embed url="https://theasciicode.com.ar/" %}

To store multiple characters or words, we need to use strings:

```
char myText[] = "Hello";
printf("%s", myText); 
```

#### Numeric Types

Use `int` when you need to store a whole number without decimals, like 35 or 1000, and `float` or `double` when you need a floating point number

The precision of `float` is six or seven decimal digits, while `double` variables have a precision of about 15 digits. Therefore, it is often safer to use `double` for most calculations - but note that it takes up twice as much [memory](https://www.w3schools.com/c/c\_data\_types\_sizeof.php) as `float` (8 bytes vs. 4 bytes).

If we want to be more precise with our outputs, we can use a dot (`.`) followed by a number that specifies how many digits that should be shown after the decimal point:

```
float myFloatNum = 3.5;

printf("%f\n", myFloatNum);   // Default will show 6 digits after the decimal point
printf("%.1f\n", myFloatNum); // Only show 1 digit
printf("%.2f\n", myFloatNum); // Only show 2 digits
printf("%.4f", myFloatNum);   // Only show 4 digits 
```

#### Get the Memory Size

If we want to print out the memory size of a var → we need to use the `sizeof` operator.

it's better to use the `%lu` format specifier to print the result:

```
printf("%lu\n", sizeof(myInt));
printf("%lu\n", sizeof(myFloat));
printf("%lu\n", sizeof(myDouble));
printf("%lu\n", sizeof(myChar)); 
```

real-life example of using different data types, to calculate and output the total cost of a number of items:

```
#include <stdio.h>

int main(){
  int a = 50;
  float b = 5.5;
  float total = a * b;

  printf("the cost of the a item is %d\n", a);
  printf("the cost of the whole stock is %f\n", total);
  return 0;
}
```

#### Type Conversion

if we need to use 2 int but the result is a float (5/2=2.5), we can use implicit conversion:

if you assign an `int` value to a `float` type:

```
// Automatic conversion: int to float
float myFloat = 9;

printf("%f", myFloat); // 9.000000 
```

and the other way around:

```
// Automatic conversion: float to int
int myInt = 9.99;

printf("%d", myInt); // 9 
```

The conversion is tricky:

```
float sum = 5 / 2;

printf("%f", sum); // 2.000000 
```

the result is 2 and not 2.5 because we assigned the value of sum with 2 int, we need to manually convert the integer values to floating-point values

```
// Manual conversion: int to float
float sum = (float) 5 / 2;

printf("%f", sum); // 2.500000 
```

or

```
int num1 = 5;
int num2 = 2;
float sum = (float) num1 / num2;

printf("%f", sum); // 2.500000 
//and if we wanted to make the output even cleaner:
printf("%.1f", sum); //2.5
```

**Real life example**

create a program to calculate the percentage of a user's score in relation to the maximum score in a game:

```
#include <stdio.h>

int main(){
  int maxScore = 1000;
  float userScore= 756;

  float pourcentScore= (userScore/maxScore)*100;
  printf("le pourcentage est de %.2f\n", pourcentScore);
  return 0;
}  //le pourcentage est de 75.60
```

### C Constants

If we do not want anyone to be able to change a variable we can use the `const` keyword.

It will make the variable **unchangeable** and **read-only**:

<figure><img src="../../.gitbook/assets/image (859).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
When you declare a constant variable, it must be assigned with a value
{% endhint %}

```
//YES:
const int minutesPerHour = 60;

//NO
const int minutesPerHour;
minutesPerHour = 60; // error 
```

### C Operators

Operators are used to perform operations on variables and values.

you can have arithmetic operators:

<figure><img src="../../.gitbook/assets/image (860).png" alt=""><figcaption></figcaption></figure>

assignment operators:

<figure><img src="../../.gitbook/assets/image (861).png" alt=""><figcaption></figcaption></figure>

Comparisons operators:

<figure><img src="../../.gitbook/assets/image (862).png" alt=""><figcaption></figcaption></figure>

Logical operators:

<figure><img src="../../.gitbook/assets/image (863).png" alt=""><figcaption></figcaption></figure>

### C Booleans

Good to know we must import booleans:

```
#include <stdbool.h>
```

A boolean variable is declared with the `bool` keyword and can only take the values `true` or `false`

It's good to know that boolean values are returned as **integers** (1=true, 0=false)

So to print it we must use `%d`

You can use `bool` to compare values:

```
printf("%d", 10 > 9);  // Returns 1 (true) because 10 is greater than 9
printf("%d", 10 == 10); // Returns 1 (true), because 10 is equal to 10
```

or 2 bool variables:

```
//if both variables are true
printf("%d", isHamburgerTasty == isPizzaTasty); 
//will return 1
```

**Real life example**:

Output "Old enough to vote!" if `myAge` is **greater than or equal to** `18`. Otherwise output "Not old enough to vote.":

```
#include <stdio.h>

int main(){
  int myAge = 21;
  const int minAge= 25;

  if(myAge>=minAge){
    printf("You are old enough to vote");
  }else{
    printf("You are not old enough to vote, you are only %d years old", myAge);
  }
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (864).png" alt=""><figcaption></figcaption></figure>

### C If ... Else

Combined to the conditions we just saw, we can add conditional statements:

<figure><img src="../../.gitbook/assets/image (865).png" alt=""><figcaption></figcaption></figure>

#### Short Hand If...Else (Ternary Operator)

As a one-liner enjoyer it's good to know that this exists:

to replace simple if else statements, we can use this syntax:

```
 variable = (condition) ? expressionTrue : expressionFalse;
```

Example:

```
int time = 20;
if (time < 18) {
  printf("Good day.");
} else {
  printf("Good evening.");
} 
```

turns into:

```
int time = 20;
(time < 18) ? printf("Good day.") : printf("Good evening."); 
```

**Real life example:**

use `if..else` to "open a door" if the user enters the correct code

```
#include <stdio.h>

int main(){
  int code = 5678;

  if(code !=5678){
    printf("Access Denied");
  }else{
    printf("Access Granted\n");
  }
  //or
  (code == 5679) ? printf("Access granted") : printf("Access denied");
  return 0;
} 
/*Access Granted
Access denied*/
```

### C Switch

The `switch` statement selects one of many code blocks to be executed:

```
switch (expression) {
  case x:
    // code block
    break;
  case y:
    // code block
    break;
  default:
    // code block
}
```

Quick little switch condition to see the mood today:

```
#include <stdio.h>

int main(){
  int mood = 3;
  
  switch(mood){
    case 1:
      printf("I am happy");
      break;
    case 2:
      printf("I am sad");
      break;
    case 3:
      printf("I am grinding");
  }
  return 0;
}
// Output "I am grinding"
```

if the mood variable has some chances of not being in our switch case, we can add a default case that specifies some code to run if there is no case match:

<pre><code><strong>...
</strong><strong>case 2:
</strong>      printf("I am sad");
      break;
    case 3:
      printf("I am grinding");
    default:
      printf("none of them");
  }
  return 0;
}
// Output for mood > 3 --> "none of them"
</code></pre>

### C While Loop

Loops can execute a block of code as long as a specified condition is reached.

```
#include <stdio.h>

int main(){
  int mood = 12;
  int i = 0;
  
  while(i<mood){
    printf("%d is not yet equal to %d\n", i, mood);
    i++;
  }
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Another variable of the while loop is the do/while loop:

```
#include <stdio.h>

int main(){
  int mood = 12;
  int i = 0;
  
  do{
    printf("%d is not yet equal to %d\n", i, mood);
    i++;
  }
    while(i < mood);
  return 0;
}
```

### C For Loop

If we know how many occurrences we want to have, we can use the for loop that takes 3 expressions

**Expression 1** is executed (one time) before the execution of the code block.

**Expression 2** defines the condition for executing the code block.

**Expression 3** is executed (every time) after the code block has been executed.

```
#include <stdio.h>

int main(){
  int mood = 4;
  
  for(int i = 0; i < mood; i++){
    printf("is %d equal to %d?\n", i, mood);
  }
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

We can also play with nested loops, but it will multiply by the number of occurrences of the first loop:

```
#include <stdio.h>

int main(){
  int mood = 4;
  int nested_loop=2;
  
  for(int i = 0; i < mood; i++){
    printf("is %d equal to %d?\n", i, mood);
    for(int b =0; b < nested_loop; b++){
      printf("the nested number %d is not equal to %d\n", b, nested_loop);
    }
  }
  return 0;
}
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

## C Break and Continue

The `break` statement can also be used to jump out of a **loop**.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

The `continue` statement breaks one iteration (in the loop), if a specified condition occurs, and continues with the next iteration in the loop.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## C Arrays
