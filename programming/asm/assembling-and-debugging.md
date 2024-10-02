# 🐱 Assembling & Debugging

## Assembly File Structure

Here is the `Hello World!` Assembly code template

```nasm
         global  _start

         section .data
message: db      "Hello HTB Academy!"

         section .text
_start:
         mov     rax, 1
         mov     rdi, 1
         mov     rsi, message
         mov     rdx, 18
         syscall

         mov     rax, 60
         mov     rdi, 0
         syscall
```

Here are different parts of this output:

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="191">Section</th><th>Description</th></tr></thead><tbody><tr><td><code>global _start</code></td><td>This is a <code>directive</code> that directs the code to start executing at the <code>_start</code> label defined below.</td></tr><tr><td><code>section .data</code></td><td>This is the <code>data</code> section, which should contain all of the variables.</td></tr><tr><td><code>section .text</code></td><td>This is the <code>text</code> section containing all of the code to be executed.</td></tr></tbody></table>

### Directives

the file is processed line-by-line, executing the instruction of each line. We see at the first line a directive `global _start`, which instructs the machine to start processing the instructions after the `_start` label. So, the machine goes to the `_start` label and starts executing the instructions there, which will print the message on the screen.

### Variables

The .`data` section holds our variables to make it easier for us to define variables and reuse them without writing them multiple times. Once we run our program, all of our variables will be loaded into memory in the `data` segment.

We can define variables using `db` for a list of bytes, `dw` for a list of words, `dd` for a list of digits, and so on

<table data-header-hidden><thead><tr><th width="234"></th><th></th></tr></thead><tbody><tr><td><code>db 0x0a</code></td><td>Defines the byte <code>0x0a</code>, which is a new line.</td></tr><tr><td><code>message db 0x41, 0x42, 0x43, 0x0a</code></td><td>Defines the label <code>message => abc</code>.</td></tr><tr><td><code>message db "Hello World!", 0x0a</code></td><td>Defines the label <code>message => Hello World!</code>.</td></tr></tbody></table>

we can also use the `equ` instruction with the `$` token to evaluate an expression, like the length of a defined variable's string.

the following code defines a variable and then defines a constant for its length:

```nasm
section .data
    message db "Hello World!", 0x0a
    length  equ $-message
```

### Code

The `.text` section holds all of the assembly instructions and loads them to the `text` memory segment. Once all instructions are loaded into the `text` segment, the processor starts executing them one after another.

## Assembling & Disassembling

### Assembling

First, we will copy the code below into a file called `helloWorld.s` (use the `.s` or the `.asm` extensions)

`helloWorld.s` file:

```nasm
global _start

section .data
    message db "Hello HTB Academy!"
    length equ $-message

section .text
_start:
    mov rax, 1
    mov rdi, 1
    mov rsi, message
    mov rdx, length
    syscall

    mov rax, 60
    mov rdi, 0
    syscall
```

we will assemble the file using `nasm`, with the following command:

```shell-session
nasm -f elf64 helloWorld.s
```

{% hint style="info" %}
The `-f elf64` flag is used to note that we want to assemble a 64-bit assembly code. If we wanted to assemble a 32-bit code, we would use `-f elf`
{% endhint %}

This should output a `helloWorld.o` object file

### Linking

Now we need to to link our file using `ld` because the `helloWorld.o` object file, though assembled, still cannot be executed.

```shell-session
ld -o helloWorld helloWorld.o
```

```shell-session
ElFelixi0@htb[/htb]$ ./helloWorld
Hello HTB Academy!
```

Here is a simple bash script to make all of the above things easier ->

```bash
#!/bin/bash

fileName="${1%%.*}" # remove .s extension

nasm -f elf64 ${fileName}".s"
ld ${fileName}".o" -o ${fileName}
[ "$2" == "-g" ] && gdb -q ${fileName} || ./${fileName}
```

```shell-session
ElFelixi0@htb[/htb]$ ./assembler.sh helloWorld.s
Hello HTB Academy!
```

### Disassembling

we will use the `objdump` tool, which dumps machine code from a file and interprets the assembly instruction of each hex code.

```shell-session
ElFelixi0@htb[/htb]$ objdump -M intel -d helloWorld

helloWorld:     file format elf64-x86-64

Disassembly of section .text:

0000000000401000 <_start>:
  401000:	b8 01 00 00 00       	mov    eax,0x1
  401005:	bf 01 00 00 00       	mov    edi,0x1
  40100a:	48 be 00 20 40 00 00 	movabs rsi,0x402000
  401011:	00 00 00
  401014:	ba 12 00 00 00       	mov    edx,0x12
  401019:	0f 05                	syscall
  40101b:	b8 3c 00 00 00       	mov    eax,0x3c
  401020:	bf 00 00 00 00       	mov    edi,0x0
  401025:	0f 05                	syscall
```

If we wanted to only show the assembly code, without machine code or addresses, we could add the `--no-show-raw-insn --no-addresses` flags

The `-d` flag will only disassemble the `.text` section of our code. To dump any strings, we can use the `-s` flag, and add `-j .data` to only examine the `.data` section. This means that we also do not need to add `-M intel`

```shell-session
ElFelixi0@htb[/htb]$ objdump -sj .data helloWorld

helloWorld:     file format elf64-x86-64

Contents of section .data:
 402000 48656c6c 6f204854 42204163 6164656d  Hello HTB Academ
 402010 7921                                 y!
```

_**Download the attached file and disassemble it to find the flag**_

```
objdump -sj .data disasm
```

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## GNU Debugger (GDB)

Debugging is a term used for finding and removing issues (i.e., bugs) from our code. we perform debugging by setting breakpoints and seeing how our program acts on each of them and how our input changes between them, which should give us a clear idea of what is causing the `bug`.

```shell-session
ElFelixi0@htb[/htb]$ sudo apt-get update
ElFelixi0@htb[/htb]$ sudo apt-get install gdb
```

GEF is a free and open-source GDB plugin that is built precisely for reverse engineering and binary exploitation. To install ->

```shell-session
ElFelixi0@htb[/htb]$ wget -O ~/.gdbinit-gef.py -q https://gef.blah.cat/py
ElFelixi0@htb[/htb]$ echo source ~/.gdbinit-gef.py >> ~/.gdbinit
```

we can run gdb to debug our `HelloWorld` binary

```shell-session
ElFelixi0@htb[/htb]$ gdb -q ./helloWorld
...SNIP...
gef➤
```

To assemble and link our assemble quickly, we can use the `assembler.sh` script we wrote in the previous section with the `-g` flag

```shell-session
ElFelixi0@htb[/htb]$ ./assembler.sh helloWorld.s -g
...SNIP...
gef➤
```

we will use the `info` command to check which `functions` are defined within the binary

```shell-session
gef➤  info functions

All defined functions:

Non-debugging symbols:
0x0000000000401000  _start
```

`info variables` command to view all available variables within the program

```shell-session
gef➤  info variables

All defined variables:

Non-debugging symbols:
0x0000000000402000  message
0x0000000000402012  __bss_start
0x0000000000402012  _edata
0x0000000000402018  _end
```

### Disassemble

To view the instructions within a specific function, we can use the `disassemble` or `disas` command

```shell-session
gef➤  disas _start

Dump of assembler code for function _start:
   0x0000000000401000 <+0>:	mov    eax,0x1
   0x0000000000401005 <+5>:	mov    edi,0x1
   0x000000000040100a <+10>:	movabs rsi,0x402000
   0x0000000000401014 <+20>:	mov    edx,0x12
   0x0000000000401019 <+25>:	syscall
   0x000000000040101b <+27>:	mov    eax,0x3c
   0x0000000000401020 <+32>:	mov    edi,0x0
   0x0000000000401025 <+37>:	syscall
End of assembler dump.
```

## Debugging with GDB

Here are the 4 steps of debugging:

<table><thead><tr><th width="130">Step</th><th>Description</th></tr></thead><tbody><tr><td><code>Break</code></td><td>Setting breakpoints at various points of interest</td></tr><tr><td><code>Examine</code></td><td>Running the program and examining the state of the program at these points</td></tr><tr><td><code>Step</code></td><td>Moving through the program to examine how it acts with each instruction and with user input</td></tr><tr><td><code>Modify</code></td><td>Modify values in specific registers or addresses at specific breakpoints, to study how it would affect the execution</td></tr></tbody></table>

### Break

The first step of debugging is setting `breakpoints` to stop the execution at a specific location or when a particular condition is met. This helps us in examining the state of the program and the value of registers at that point.

```shell-session
gef➤  b _start

Breakpoint 1 at 0x401000
```

we can use the `run` or `r` command

```nasm
gef➤  r
Starting program: ./helloWorld 

Breakpoint 1, 0x0000000000401000 in _start ()
[ Legend: Modified register | Code | Heap | Stack | String ]
───────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0               
$rbx   : 0x0               
$rcx   : 0x0               
$rdx   : 0x0               
$rsp   : 0x00007fffffffe310  →  0x0000000000000001
$rbp   : 0x0               
$rsi   : 0x0               
$rdi   : 0x0               
$rip   : 0x0000000000401000  →  <_start+0> mov eax, 0x1
...SNIP...
───────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007fffffffe310│+0x0000: 0x0000000000000001	 ← $rsp
0x00007fffffffe318│+0x0008: 0x00007fffffffe5a0  →  "./helloWorld"
...SNIP...
─────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x400ffa                  add    BYTE PTR [rax], al
     0x400ffc                  add    BYTE PTR [rax], al
     0x400ffe                  add    BYTE PTR [rax], al
 →   0x401000 <_start+0>       mov    eax, 0x1
     0x401005 <_start+5>       mov    edi, 0x1
     0x40100a <_start+10>      movabs rsi, 0x402000
     0x401014 <_start+20>      mov    edx, 0x12
     0x401019 <_start+25>      syscall 
     0x40101b <_start+27>      mov    eax, 0x3c
─────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "helloWorld", stopped 0x401000 in _start (), reason: BREAKPOINT
───────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x401000 → _start()
```

If we want to set a breakpoint at a certain address, like `_start+10`, we can either `b *_start+10` or `b *0x40100a`

### Examine
