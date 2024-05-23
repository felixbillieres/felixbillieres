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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

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

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## C Break and Continue

The `break` statement can also be used to jump out of a **loop**.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The `continue` statement breaks one iteration (in the loop), if a specified condition occurs, and continues with the next iteration in the loop.

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## C Arrays



## C Strings

here is the syntax for a string in C:

```
 char greetings[] = "Hello World!";
```

for the printf output we need the format specifier `%s` for strings and `%c` format specifier to print a **single character**.

If we wanted to modify a single character in a string, we would refer to the index like in an array since strings are quite litterally char arrays, and we would use single quotes for the assignment:

```
char greetings[] = "Hello World!";
greetings[0] = 'J';
printf("%s", greetings);
// Outputs Jello World! instead of Hello World!
```

You can use the for loop just like any other way we learnt earlier:

```
char felix[] = "Felix";

  for(int i = 0; i < 5; i++){
    printf("%c is the %dth letter of my name\n", felix[i], i+1);
  }
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

another interestig way of printing out strings is to use {} and single quotes:

```
  char felix[] = {'f','e','l','i','x'};
  printf("%s is my name", felix);
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

and to finish the size of a string is pretty straightforward:

```
char felix[] = {'f','e','l','i','x'};

printf("%s is my name\n", felix);
printf("the number of chars of my name is %d", sizeof(felix));
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we want to print out " or ' or \ (special chars)in our strings without it breaking the function we need to put escape characters:

```
char txt[] = "We are the so-called "Vikings" from the north.";
//error
char txt[] = "We are the so-called \"Vikings\" from the north.";
//no error
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

**String functions:**

We can perform some interesting stuff on strings if we include `<string.h>` header file in our program ->

first we need to know the difference between strlen() and sizeof()

`sizeof` and `strlen` behaves differently, as `sizeof` also includes the `\0` character when counting:

```
char alphabet[] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
printf("%d", strlen(alphabet));   // 26
printf("%d", sizeof(alphabet));   // 27
```

and sizeof() will return the memory size in bytes and not the length as we understand it:

```
char alphabet[50] = "ABCDEFGHIJKLMNOPQRSTUVWXYZ";
printf("%d", strlen(alphabet));   // 26
printf("%d", sizeof(alphabet));   // 50
```

If we want to concatenate strings, we can use the strcat() function but we need to be aware of buffer overflow:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (5) (1) (2).png" alt=""><figcaption></figcaption></figure>

In C, when you declare an array of characters like `char string1[12]`, you're allocating space for 12 characters, including the null terminator `'\0'`. However, when you initialize `string1` with `"hello"`, it already consumes 6 characters (5 for "hello" and 1 for `'\0'`). So, `string1` has only 6 characters left before reaching its maximum capacity.

**Copy strings:**

we need to use the strcpy() function if we want to assign the value of one string to another

```
char string1[6] = "hello";
char string2[6];
  
strcpy(string2, string1);
printf("%s", string2);
```

{% hint style="info" %}
notice how while using strcpy we put the var that we want to assign something to in first
{% endhint %}

**Compare strings:**

If we want to compare 2 strings, we need to use the strcmp() function:

the output is either ->

0 == equal

!0 == not equal

```
char string1[6] = "hello";
  char string2[6] = "hello";
  char string3[6] = "hel";
  
  printf("%d\n", strcmp(string1, string2)); //0
  printf("%d", strcmp(string1, string3)); //108
```

<figure><img src="../../.gitbook/assets/image (6) (1) (2).png" alt=""><figcaption></figcaption></figure>

## C User Input

To capture user input, we need to use the scanf() function:

```
int myNum;

// Ask the user to type a number
printf("Type a number: \n");

// Get and save the number the user types
scanf("%d", &myNum);

// Output the number the user typed
printf("Your number is: %d", myNum);
```

Like this:

```
int myage;
printf("enter your age:\n");
scanf("%d", &myage);
printf("your age is %d", myage);
```

and for multiple inputs:

```
  int myage;
  char myname[10];
  printf("enter your age:\n");
  printf("enter your name:\n");

  scanf("%d", &myage);
  scanf("%s", &myname);
  printf("your age is %d and your name is %s", myage, myname);
```

{% hint style="info" %}
scanf() considers space chars as escape chars so if you were to write your name "John Doe" it would ouput only John
{% endhint %}

Use the `scanf()` function to get a single word as input, and use `fgets()` for multiple words.&#x20;

fgets() takes the following parameters: the name of the string variable, `sizeof`(_string\_name_), and `stdin`:

```
char fullName[30];

printf("Type your full name: \n");
fgets(fullName, sizeof(fullName), stdin);

printf("Hello %s", fullName);
```

## C Memory Address

To access memory addresses in C you have to put a & before the variable:

```
int myAge = 43;
printf("%p", &myAge); // Outputs 0x7ffe5367e044
```

the memory address is in hexadecimal form

## C Pointers

A **pointer** is a variable that **stores** the **memory address** of another variable as its value.

A **pointer variable** **points** to a **data type** (like `int`) of the same type, and is created with the `*` operator.

```
int myAge = 43;     // An int variable
int* ptr = &myAge;  // A pointer variable, with the name ptr, that stores the address of myAge

// Output the value of myAge (43)
printf("%d\n", myAge);

// Output the memory address of myAge (0x7ffe5367e044)
printf("%p\n", &myAge);

// Output the memory address of myAge with the pointer (0x7ffe5367e044)
printf("%p\n", ptr);
```

you can dereference a pointer by calling it with a \* character before ->

```
int myAge = 43;     // Variable declaration
int* ptr = &myAge;  // Pointer declaration

// Reference: Output the memory address of myAge with the pointer (0x7ffe5367e044)
printf("%p\n", ptr);

// Dereference: Output the value of myAge with the pointer (43)
printf("%d\n", *ptr);
```

You can also declare pointers thes 2 ways in C:

```
int* myNum;
int *myNum;
```

#### Pointers & Arrays

let's say we have the following array:

```
int myNumbers[4] = {25, 50, 75, 100};
```

<figure><img src="../../.gitbook/assets/image (964).png" alt=""><figcaption></figcaption></figure>

Note that the last number of each of the elements' memory address is different, with an addition of 4.

It is because the size of an `int` type is typically 4 bytes ->

<figure><img src="../../.gitbook/assets/image (965).png" alt=""><figcaption></figcaption></figure>

In c, the name of the array is a direct pointer to the first element of the array

The **memory address** of the **first element** is the same as the **name of the array**:

```
int myNumbers[4] = {25, 50, 75, 100};

// Get the memory address of the myNumbers array
printf("%p\n", myNumbers);

// Get the memory address of the first array element
printf("%p\n", &myNumbers[0]);

//0x7ffe70f9d8f0
//0x7ffe70f9d8f0
```

So if we want to access to an element of the array through a pointer, we have to take the array (which points to the first element) and add 1,2,3... ->

```
int myNumbers[4] = {25, 50, 75, 100};

// Get the value of the first element in myNumbers
printf("%d", *myNumbers); //25

// Get the value of the second element in myNumbers
printf("%d\n", *(myNumbers + 1)); //50

// Get the value of the third element in myNumbers
printf("%d", *(myNumbers + 2)); //75
```

<figure><img src="../../.gitbook/assets/image (966).png" alt=""><figcaption><p>or through a loop</p></figcaption></figure>

Arrays are used to store multiple values in a single variable, instead of declaring separate variables for each value.

We first define the data type (like `int`) and specify the name of the array followed by&#x20;

**square brackets \[]**

```
int myNumbers[] = {25, 50, 75, 100};
```

To access an array element, refer to its **index number:**

```
int myNumbers[] = {25, 50, 75, 100};
printf("%d", myNumbers[0]);

// Outputs 25
```

Now if we wanted to change the value assigned to a array value:

```
int myNumbers[] = {25, 50, 75, 100};
myNumbers[0] = 33;

printf("%d", myNumbers[0]);

// Now outputs 33 instead of 25
```

And finally, we could use the for loop to iterate through an array:

```
int myNumbers[] = {25, 50, 75, 100};
int i;

for (i = 0; i < 4; i++) {
  printf("%d\n", myNumbers[i]);
}
```

Now if we wanted to know the **size of** an array:

```
int myNumbers[] = {10, 25, 50, 75, 100};
printf("%lu", sizeof(myNumbers)); // Prints 20
```

the output is equal to 20 because the output refers to the size of bytes and since an int is usually equal to 4 bytes, 4x5=20

if we wanted to output the size as we think of it (the number of variables in the array):

```
int myNumbers[] = {10, 25, 50, 75, 100};
int length = sizeof(myNumbers) / sizeof(myNumbers[0]);

printf("%d", length);  // Prints 5
```

* `sizeof(myNumbers)` gives the total size in bytes of the `myNumbers` array. Since `myNumbers` is an array of integers (`int`), and there are five integers in the array, the total size would be `5 * sizeof(int)`.
* `sizeof(myNumbers[0])` gives the size in bytes of the first element of the `myNumbers` array, which is an integer (`int`).
* By dividing the total size of the array (`sizeof(myNumbers)`) by the size of a single element (`sizeof(myNumbers[0])`), we get the number of elements in the array.

We can make our for loop better by specifying the length of our array in the for loop:

```
int myNumbers[] = {25, 50, 75, 100};
int length = sizeof(myNumbers) / sizeof(myNumbers[0]);
int i;

for (i = 0; i < length; i++) {
  printf("%d\n", myNumbers[i]);
}
```

A real life example of a program that calculates the average of different users:

<pre><code>#include &#x3C;stdio.h>

int main(){
  int age[] = {22,23,15,90};

  int length = sizeof(age) / sizeof(age[0]);
  int value =0;
  for(int i = 0; i &#x3C; length; i++){
    value += age[i];
  }
  int finalMoyenne = value / length;
  printf("le total des ages reunis est %d\n", value);
  printf("La moyenne est de %d", finalMoyenne);
  return 0;
}
//le total des ages reunis est 150
<strong>//La moyenne est de 37
</strong></code></pre>

another useful thing good to know is multi dimensional arrays:

lets create a 2D array:

```
int matrix[2][3] = { {1, 4, 2}, {3, 6, 8} };
```

<figure><img src="../../.gitbook/assets/image (994).png" alt=""><figcaption></figcaption></figure>

To access the values of the array, you first specify the which row you want then which column:

```
int matrix[2][3] = { {1, 4, 2}, {3, 6, 8} };

printf("%d", matrix[0][2]);  // Outputs 2, row number 0 and element 2 in that row
```

if you want to change the value of an element in the array it's the same, you first choose the row, then the element in the row:

```
  int twoDarray[2][3] = {{1,2,3},{4,5,6}};
  printf("%d\n", twoDarray[0][2]);
  twoDarray[0][2] = 4;
  printf("%d\n", twoDarray[0][2]);
//3
//4
```
