---
description: https://tryhackme.com/r/room/reverselfiles
---

# ⏪ Reversing ELF

### Crackme2 <a href="#id-5c50" id="id-5c50"></a>

Ok, so I start by doing:

```
strings crackme2
```

I find the secret password:&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and with a quick&#x20;

```
chmod 777 Crackme2
./Crackme2 super_secret_password
```

You find the flag

### Crackme3

for this one, i just ran `strings crackme3` and found something that popped ->

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This looks like a base64 encoded message →

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Indeed, it is :smile:

### Crackme4

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

With a quick strings crackme4 we find the usage so ./crackme4 password&#x20;

and we see something about strcmp function, so we might need to use gdb debugger ->

I open up the file using gdb with `gdb crackme4` command and then look at the different functions using `info functions`

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So there we get the name of the functions, we just need to spot the interesting ones

we have 2/3 funcs that seem interesting, but we'll only look at the `strcmp@plt` one since it's mentioned in the strings ->

**What is `strcmp@plt`?**

* **PLT (Procedure Linkage Table)**:
  * The PLT is a mechanism used in ELF binaries for dynamic linking. When a program uses shared libraries, function calls to these libraries are initially routed through the PLT.
  * The entry in the PLT for `strcmp` is used to resolve the actual address of `strcmp` at runtime. This allows the program to dynamically link to the correct function in the shared library.

We then set a breakpoint at the memory address of the function

{% hint style="info" %}
In software development, a **breakpoint** is an intentional stopping or pausing place in a **program**, put in place for debugging purposes and to help acquire knowledge about a **program** during its execution.
{% endhint %}

{% embed url="https://users.pja.edu.pl/~jms/qnx/help/watcom/wd/brkpt.html" %}

```
b *0x0000000000400520
```

So here i set a breakpoint to be able to run it with a test value to see it run UNTIL it hits the breakpoint

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **Breakpoint Set**: You set a breakpoint at the PLT entry for `strcmp`.
* **Program Start**: You run the program with `test` as an argument.
* **Breakpoint Hit**: Execution stops at `0x400520`, the PLT entry for `strcmp`.

```
(gdb) b *0x0000000000400520
(gdb) run test
```

* This command (b \*0x0000000000400520) sets a breakpoint at the address `0x400520`, which corresponds to the `strcmp@plt` entry.
* This means the program will pause execution when it reaches the PLT entry for the `strcmp` function.
* and the run test command starts running the program `/home/felix/Bureau/assembyAndReverse/crackme4` with `test` as the command-line argument.

Then we try a new command to go and check out the registers ->

```
(gdb) info registers
```

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

if we run the following command we will be able to access various memory parts of the system:

```
(gdb) x/s 0x7fffffffe36a
```

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* **`x`**: The `x` command in GDB is short for "examine". It allows you to view the contents of memory at a specified address.
* **`/s`**: The `/s` modifier specifies the format in which to display the memory contents. The `s` stands for "string". When you use `/s`, GDB interprets the memory at the specified address as a null-terminated string (a sequence of characters ending with a `\0`).
* **`0x7fffffffe36a`**: This is the address in memory that you want to examine. In this case, `0x7fffffffe36a` is a specific memory address in the program's address space.

And after displaying a few addresses that are not valuable, I find the one that we are looking for:&#x20;

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Crackme5 <a href="#ab1c" id="ab1c"></a>

Ok so first we can see what the program does ->

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Using info functions, we find some interesting functions ->

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now we create a breakpoint with the memory of the strncmp@plt function  ->

```
b *0x0000000000400560
```

We then run test ->

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and when we show the registers and output, the values of Rex and rcx ->

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

`x/s` tells GDB to examine the memory at the specified address and interpret it as a null-terminated string

**Output:** `0x7fffffffdef0: "OfdlDSA|3tXb32~X3tX@sX`4tXtz\377\177"\`

* This output indicates that at memory address `0x7fffffffdef0`, GDB found a sequence of characters that form the string `"OfdlDSA|3tXb32~X3tX@sX`4tXtz\377\177"\`.
* This string includes printable ASCII characters, and it ends with non-printable characters represented by escape sequences: `\377` (octal for 255) and `\177` (octal for 127).

### Crackme6 <a href="#ab1c" id="ab1c"></a>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Ok so no important information without digging but since it says source, we might need ghidra ->&#x20;

<figure><img src="../../../.gitbook/assets/image (17) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>1337_pwd</p></figcaption></figure>

### Crackme7

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Crackme8

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://johnkavanagh.co.uk/articles/string-to-integer-atoi-decoding-strings-in-web-development/" %}

{% hint style="info" %}
The **atoi()** function is a function in the C programming language that **converts a string into an integer** numerical representation. I can convert the **-0x35010ff3** value to decimal
{% endhint %}

