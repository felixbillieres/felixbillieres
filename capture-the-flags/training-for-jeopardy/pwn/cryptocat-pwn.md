---
description: https://github.com/Crypto-Cat/CTF/tree/main/pwn/binary_exploitation_101
---

# 🐈 Cryptocat PWN

### 00-intro\_setup\_basics

{% embed url="https://avinetworks.com/glossary/buffer-overflow/" %}

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

You can obviously find thousands of definitions about what is a buffer overflow on internet but, here is my definition and how i understand it:

{% embed url="https://en.wikipedia.org/wiki/Data_buffer" %}

A data buffer is a region of the memory who stores data temporarily. There is a size allocated to a buffer's memory and buffer overflow or buffer overrun is when you write past that allocated memory and overwrite other zones in the memory you were not supposed to overwite. The goal is to write into areas known to hold [executable code](https://en.wikipedia.org/wiki/Execution\_\(computing\)) and replace it with [malicious code](https://en.wikipedia.org/wiki/Malicious\_code), or to selectively overwrite data pertaining to the program's state, therefore causing behavior that was not intended by the original programmer.

I hope that's not too far from the overall definition.

Now we will see the tools that we will need to persue all those exercises, first we'll need&#x20;

* A disassembler ->
