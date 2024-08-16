---
description: https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
---

# 🐈 00-intro\_setup\_basics

###

{% embed url="https://avinetworks.com/glossary/buffer-overflow/" %}

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

You can obviously find thousands of definitions about what is a buffer overflow on internet but, here is my definition and how i understand it:

{% embed url="https://en.wikipedia.org/wiki/Data_buffer" %}

A data buffer is a region of the memory who stores data temporarily. There is a size allocated to a buffer's memory and buffer overflow or buffer overrun is when you write past that allocated memory and overwrite other zones in the memory you were not supposed to overwite. The goal is to write into areas known to hold [executable code](https://en.wikipedia.org/wiki/Execution\_\(computing\)) and replace it with [malicious code](https://en.wikipedia.org/wiki/Malicious\_code), or to selectively overwrite data pertaining to the program's state, therefore causing behavior that was not intended by the original programmer.

I hope that's not too far from the overall definition.

Now we will see the tools that we will need to persue all those exercises, first we'll need:

* Disassembler - Ghidra/IDA/Radare/BinaryNinja/Hopper: [https://gist.github.com/liba2k/d522b4...](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbEtKd3o4M0c2Ujk2SlpKb0NwWndHRHNoYU9Fd3xBQ3Jtc0tuSFlaNlRTSEVxdm5vbmRzN3pHZWZoWXhkcks2RmRlazdRV3hsWlRzdzdnNkxXdHBENWg2SE5WZTdYNTZHeTNOTU05aXROaGVDa3ZTZG1tMkpPMTcwdE5TTGF3djdyRE9HU2J3XzVKMHdXQ0FaSFVWcw\&q=https%3A%2F%2Fgist.github.com%2Fliba2k%2Fd522b4f20632c4581af728b286028f8f\&v=wa3sMSdLyHw)&#x20;
* Debugger - GDB-PwnDBG/GEF/PEDA: [https://infosecwriteups.com/pwndbg-ge...](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbTBQb2VLSXZHNTduZmFFc0dLREZiTGUteEx6Z3xBQ3Jtc0tuaVFZMm1pUlpLUFN2STV6dHdCMjhWd1NiRjVxTlNSU1laZTdlVF9fMGpmcUllMjd5YmtjZnZyNlJmeDItZzNJd2tXYUNVRFduVUdnbXphbGNKQWdTRkxfUkdQdktZeEVCWnBDTXlwVkxSb3c1YWtTcw\&q=https%3A%2F%2Finfosecwriteups.com%2Fpwndbg-gef-peda-one-for-all-and-all-for-one-714d71bf36b8\&v=wa3sMSdLyHw)&#x20;
* Checksec: [https://github.com/slimm609/checksec.sh](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbnVxdGJxUnNtcHpzSXNOYkVwSG1iR1R2SjIyZ3xBQ3Jtc0tsbU02SzNDWUM5U1FaOEQ0LWY0ZlBWS3FqZi0zMm5MNUpZNk1SVW1oZnlzQ1dpVDJCWXRCMjlDbEl3WlJ2ZGpSZkoyNTlsb1FXQnB2SHBwLWV5ZzNURy1EUENvX1dlU01uNHpnQ3Z4SmdCNUh2MU9zZw\&q=https%3A%2F%2Fgithub.com%2Fslimm609%2Fchecksec.sh\&v=wa3sMSdLyHw)&#x20;
* Ropper: [https://github.com/sashs/Ropper](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbkRPNjJUTm5VdHZxVXphSGpmTmhWNlprbFlId3xBQ3Jtc0tsdGtfaWEwcGNwSmJoNG81QUxkakpNemRMbjcwd3o1MnhhaXhPTGRCVkk3dENfLV9ILThWdm9wMXdUOVRVcjhTaTRMWDZrb2lwRFpuWVd5MnBrYVhPOFk1c2xmUVVkekNfZVhuSlNFb0ZaT1ZRYkl0WQ\&q=https%3A%2F%2Fgithub.com%2Fsashs%2FRopper\&v=wa3sMSdLyHw)

Here are some useful ressources:

* Ghidra: [https://ghidra-sre.org/CheatSheet.html](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbUhaYnV0NGFGQ0RtWXNVLWljUVNLT2NmYU5GUXxBQ3Jtc0tscFl1TFhRRWd2bUhRZkgxZmpvM2lMNFBzZWgtWWNjbmlpcW1NX2lkSlJVM3JoYTMwSldSNHdCNmFRUVEyaGNIYVBZa3pYV3FTejZPcVVfM3RLYWxJdHVweTdpU25COUZVMTBJc1RIbDdXRnY0UDRMMA\&q=https%3A%2F%2Fghidra-sre.org%2FCheatSheet.html\&v=wa3sMSdLyHw)&#x20;
* PwnTools: [https://github.com/Gallopsled/pwntool...](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbVY2Q1VVZVh3ek42NjkwNE5oRkk2V0M5aWNnQXxBQ3Jtc0tsV3p6S3E0SkJhVk5hcWppd090VUtuRjNncTBWUXd2d1NLcFNpTlNvRC1rQ1pIZ2RoN1FPcWI3TERpME9pX0gybDlqajFkMjg2ejNnNUhyeUp5LS1vOFRlUktsU2NLYjFxOEt4RXhxYWJrWjFtNU5xQQ\&q=https%3A%2F%2Fgithub.com%2FGallopsled%2Fpwntools-tutorial\&v=wa3sMSdLyHw)&#x20;
* CyberChef: [https://gchq.github.io/CyberChef](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbXduTHIzYnM5QnMzNVlab2JLVlVTLW5xcmNOQXxBQ3Jtc0ttb1FhSi1udkpBeWlyeWQwcndXVlFuS3VaRFg5RnBTRk5Vck5LUjZNZ3NvT3FZR2V2V0w4amtvOUttSW5YdER1OV9za1RtYndpS2tKWk9neE1uU1U2NS1rM1prOUlqNVZnNjJYUGdqbWMtRVNMbjE2QQ\&q=https%3A%2F%2Fgchq.github.io%2FCyberChef\&v=wa3sMSdLyHw)&#x20;
* HackTricks: [https://book.hacktricks.xyz/exploitin...](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbHBqUzZpSW5oREFXa3pIakJfUElDd2ZiUzJrZ3xBQ3Jtc0trTXB1czdPX1RpMnZQWDZGU0ZSWW5XRzNyRDRJR09ENjVQNENZeTZvdG9SZFVmUno1Yk1iYmN5ZW5sUW9LSVY0QS1aMW5VcVZMenBXMHVTN19WSHdWdFVjOFdqQmZtUmItOV9Ta0xhbExmaHY2UTZiTQ\&q=https%3A%2F%2Fbook.hacktricks.xyz%2Fexploiting%2Flinux-exploiting-basic-esp\&v=wa3sMSdLyHw)&#x20;
* GTFOBins: [https://gtfobins.github.io](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbXdNRnlyYWUtM1BHU0pRVVExZGcta3BzTEVqQXxBQ3Jtc0tteExvMWFuMGZEZm5GMHdNMTF3T3prcmthdkVnNUlUVFpZTVVUMFBQaWRNOW1hQktLLTZPTVlvWEhGdXExOFdRQ2pKaGt3ZmNWUS1fU1E1RDh2RExhUjk0RWpfOEhTZVJTdnhrcmxZaHdpYXdLNmF3dw\&q=https%3A%2F%2Fgtfobins.github.io%2F\&v=wa3sMSdLyHw)&#x20;
* Decompile Code: [https://www.decompiler.com](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqazBpRWEyOER5NGU5b0N3a3RXblhIVmtYX3Jrd3xBQ3Jtc0trSFRoX1FSYTlPQ2ppTlpNZVZVR2swUlpYYkJ1akhpdUVqVzJPdFdVaTlHNzB0b2lBT19kWEozcThPYmpKLURwaDZOckQ3OWc3OUJZakFhRHlsd1lGbGU3S1l2dkNmZm50YWhLSjlRcGhmY2MyU3pxRQ\&q=https%3A%2F%2Fwww.decompiler.com%2F\&v=wa3sMSdLyHw)&#x20;
* Run Code: [https://tio.run](https://www.youtube.com/redirect?event=video\_description\&redir\_token=QUFFLUhqbUFOVUNlUjBhZlhNZ29OVzRaQVVtOUc2WXNOUXxBQ3Jtc0tsTmF3aWVfNlU1M3hYX2pDckxHbUZzRmhidVRiUnUzYXh2SExZUlBHZlZVaUFubE91Sm8zSHhXYURQU0ZvMV9RMDRsdnZMai1KTnczLXRqZF9PNWwxbk9lVkNqMkxVN0F1eWE5NTdYdlI2MnhubXRUQQ\&q=https%3A%2F%2Ftio.run%2F\&v=wa3sMSdLyHw)

### Here is a python script that will be make it possible to create a ghidra project very quickly:

{% embed url="https://gist.github.com/liba2k/d522b4f20632c4581af728b286028f8f" %}

Let's get on the practical side of things ->

this is our vulnerable code

```c
#include <stdio.h>
#include <string.h>

int main(void)
{
    char buffer[16];

    printf("Give me data plz: \n");
    gets(buffer);
    
    return 0;
}
```

Ok so let's start by getting some infos  about the file ->

let's remember we have 2 files: `vuln` & `vuln.c`

```
file vuln
```

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's explain:

* **ELF**: This stands for Executable and Linkable Format, which is a common file format for executables, object code, shared libraries, and core dumps in Unix-like systems.
* **32-bit**: Indicates that this executable is compiled for a 32-bit architecture, meaning it can access 2^32 memory addresses.
* **LSB**: Stands for Least Significant Byte. This indicates that the file uses little-endian byte order, where the least significant byte is stored at the smallest address.
* **dynamically linked**: This means that the executable is linked to shared libraries at runtime, rather than having all code statically included in the executable.
* **not stripped**: This means that the executable still contains its symbol table and debugging information. Stripped executables have this information removed to reduce file size.

Here it is already compiled but here is the command if we would want to do it:

```
gcc vuln.c -o vuln -fstack-protector-all
```

And we get a warning:

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

this gives us a hint on what to do next

Let's try to do our buffer overflow ->

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The trick here if i understand it well is that there is a buffer size of 16 allocated to the buffer and gets is very unsecure unlike fgets because it does not look at the size inputed so if we enter an input greater than 16 it will crash

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now if we try to open the file with gdb ->

```
gdb vuln
```

We can get some infos about the file and what it's doing:

<figure><img src="../../../../.gitbook/assets/image (1174).png" alt=""><figcaption></figcaption></figure>

What is interesting is obviously the main function so let's disassemble it:

```
disassemble main
```

<figure><img src="../../../../.gitbook/assets/image (1175).png" alt=""><figcaption></figcaption></figure>

We could jump to different functions inside the debugger or set up a breakpoint

{% embed url="https://learn.microsoft.com/en-us/visualstudio/debugger/using-breakpoints?view=vs-2022" %}

Let's set up a breakpoint at main:

```
break main
run
```

<figure><img src="../../../../.gitbook/assets/image (1176).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1177).png" alt=""><figcaption></figcaption></figure>

If we want an info about the stack:

```
info stack
```

<figure><img src="../../../../.gitbook/assets/image (1178).png" alt=""><figcaption></figcaption></figure>

now let's just type c (continue), this will technically run the program and we will me able to see how it reacts ->

<figure><img src="../../../../.gitbook/assets/image (1179).png" alt=""><figcaption><p>no buffer overflow</p></figcaption></figure>

don't forget to do a little `delete breakpoints` si it does not hit many times on it



### Ghidra methodology

First create a new project and import the file you want:

<figure><img src="../../../../.gitbook/assets/image (1183).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1184).png" alt=""><figcaption></figcaption></figure>

click on yes to analyze and then you're in:

<figure><img src="../../../../.gitbook/assets/image (1185).png" alt=""><figcaption></figcaption></figure>

We can start by looking at the main function and quickly see in the decompiler that this looks a lot like vuln.c program we had earlier:

<figure><img src="../../../../.gitbook/assets/image (1186).png" alt=""><figcaption></figcaption></figure>

if we select a function name and type L, we can globally modify it's value:

<figure><img src="../../../../.gitbook/assets/image (1187).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1188).png" alt=""><figcaption></figcaption></figure>
