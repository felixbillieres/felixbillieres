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

## C Format Specifiers

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

### Change Variable Values

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

### Add Variables Together

Using the + operator:

```
int x = 5;
int y = 6;
int sum = x + y;
printf("%d", sum); 
```
