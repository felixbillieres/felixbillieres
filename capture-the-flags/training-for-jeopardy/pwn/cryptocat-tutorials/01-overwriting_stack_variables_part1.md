# ✍️ 01-overwriting\_stack\_variables\_part1

Here is our C program:

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char password[6];
    int authorised = 0;

    printf("Enter admin password: \n");
    gets(password);

    if(strcmp(password, "pass") == 0)
    {
        printf("Correct Password!\n");
        authorised = 1;
    }
    else
    {
        printf("Incorrect Password!\n");
    }

    if(authorised)
    {
        printf("Successfully logged in as Admin (authorised=%d) :)\n", authorised);
    }else{
		printf("Failed to log in as Admin (authorised=%d) :(\n", authorised);
	}

    return 0;
}#
```

So off the bat we can look at what the binary is doing ->

<figure><img src="../../../../.gitbook/assets/image (1189).png" alt=""><figcaption></figcaption></figure>

we are able to bypass login pretty much by accident but let's dig deeper ->

Now we can try using ltrace to see what the program does:

<figure><img src="../../../../.gitbook/assets/image (1191).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1190).png" alt=""><figcaption></figcaption></figure>

we can see it compares our test input to "pass"

so technically we can legit bypass auth with this:

<figure><img src="../../../../.gitbook/assets/image (1192).png" alt=""><figcaption></figcaption></figure>

If we go and look deeper we can see that we had a segmentation fault when we put plenty of random chars so let's try to fix that:

<figure><img src="../../../../.gitbook/assets/image (1193).png" alt=""><figcaption></figcaption></figure>

So when the password is valid we get "authorized=1", when we put 6 chars random chars we get password incorrect but when we put 7 chars we bypass login, no segmentation fault with "authorized=97". Why 97? just because it's the ascii value of a, if we inputed 7 b's it would've been 98

Let's review the code to understand better what's happening:

<figure><img src="../../../../.gitbook/assets/image (1194).png" alt=""><figcaption></figcaption></figure>

so our input is passed in as "password" variable with a wanted length of 6 but we fetch the variable with gets who is highly unsecure and will not check the length, thus making a buffer overflow very easily

<figure><img src="../../../../.gitbook/assets/image (1195).png" alt=""><figcaption></figcaption></figure>

Here we can see that the if(authorised) function is valid if authorised is not equal to 0 and with pretty much every ascii > 0, it's very easy to check this function

Let's hop on ghidra to look at all of that:

<figure><img src="../../../../.gitbook/assets/image (1196).png" alt=""><figcaption></figcaption></figure>

after decompiling main we can see some code that's really similare to what we have

to get better understanding of the code we can start by changing the variable names with the select+L tip:

<figure><img src="../../../../.gitbook/assets/image (1197).png" alt=""><figcaption></figcaption></figure>

Now let's look at the comparaison:

<figure><img src="../../../../.gitbook/assets/image (1198).png" alt=""><figcaption></figcaption></figure>

so we go and look in the assembly the parti where it checks if authorised is equal to 0, we can see that if the result is matching there is a jump to `LAB_0804923b`&#x20;

Let's go and take a look at this with gdb ->

we can start by using the info functions command and later on disassemble main

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when we disassemble main we can find the compare function that we saw earlier in ghidra,

now let's set up a breakpoint at the compare function ->

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>break *0x0804921ew</p></figcaption></figure>

Now that we set the breakpoint let's run the program and input test as our input ->

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

when it hits the breakpoint it's going to show us some stuff:

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So here we can see the line that's being currently ran, its comparing to see if what's in ebp minux 0cx (12 in decimal)  is equal 0 and if it is, we can see in the next line that it will jump to main+169

let's print out that value:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>x $ebp - 0xc</p></figcaption></figure>

We can modify this value but instead of `ebp = 0xc =...` , let's  modify the value of  `0xffa6683c` because `ebp = 0xc` holds a pointer to the location of the authorised variable:

<figure><img src="../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
pwndbg> set *0xffa6683c = 1
pwndbg> x $ebp - 0xc
0xffa6683c:     0x00000001
```

We can see the value changed to 1 now that we want to print out `ebp = 0xc`

So now we can hit continue and get the login without any errors since value is = to 1:

<figure><img src="../../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now let's show another POV, let's run the binary and input 6 a's

If we print out the value using `x $ebp - 0xc` look at what we see ->

<figure><img src="../../../../.gitbook/assets/image (9) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The value is 61
