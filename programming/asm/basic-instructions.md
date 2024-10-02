---
icon: pi
---

# Basic Instructions

## Data Movement

We will frequently use data movement instructions for moving data between addresses, moving data between registers and memory addresses, and loading immediate data into registers or memory addresses.

<table><thead><tr><th width="120">Instruction</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td><code>mov</code></td><td>Move data or load immediate data</td><td><code>mov rax, 1</code> -> <code>rax = 1</code></td></tr><tr><td><code>lea</code></td><td>Load an address pointing to the value</td><td><code>lea rax, [rsp+5]</code> -> <code>rax = rsp+5</code></td></tr><tr><td><code>xchg</code></td><td>Swap data between two registers or addresses</td><td><code>xchg rax, rbx</code> -> <code>rax = rbx, rbx = rax</code></td></tr></tbody></table>

### Moving Data

Let's say `rax` and `rbx`, such that `rax = 0` and `rbx =1`

```nasm
global  _start

section .text
_start:
    mov rax, 0
    mov rbx, 1
```

And we can run ->

```nasm
$ ./assembler.sh fib.s -g
gef➤  b _start
Breakpoint 1 at 0x401000
gef➤  r
─────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
 →   0x401000 <_start+0>       mov    eax, 0x0
     0x401005 <_start+5>       mov    ebx, 0x1
───────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0
$rbx   : 0x0

...SNIP...

─────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x401000 <_start+0>       mov    eax, 0x0
 →   0x401005 <_start+5>       mov    ebx, 0x1
───────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0
$rbx   : 0x1
```

{% hint style="info" %}
we can consider `mov` as a `copy` function
{% endhint %}

We have to remember here that `the size of the loaded data depends on the size of the destination register`. For example, in the above `mov rax, 1` instruction, since we used the 64-bit register `rax`, it will be moving a 64-bit representation of the number `1` (i.e. `0x00000001`), which is not very efficient.

`This is why it is more efficient to use a register size that matches our data size`. For example, we will get the same result as the above example if we use `mov al, 1`, since we are moving 1-byte (`0x01`) into a 1-byte register (`al`), which is much more efficient.

```nasm
global  _start

section .text
_start:
    mov rax, 0
	mov rbx, 1
    mov bl, 1
```

Now let's assemble it and view its shellcode with `objdump`

```shell-session
ElFelixi0@htb[/htb]$ nasm -f elf64 fib.s && objdump -M intel -d fib.o
...SNIP...
0000000000000000 <_start>:
   0:	b8 00 00 00 00       	mov    eax,0x0
   5:	bb 01 00 00 00       	mov    ebx,0x1
   a:	b3 01                	mov    bl,0x1
```

We can see that the shellcode of the first instruction is more than double the size of the last instruction.

We are going to modify our code to use sub-registers to make it more efficient:

```nasm
global  _start

section .text
_start:
    mov al, 0
    mov bl, 1
```

The `xchg` instruction will swap the data between the two registers.

### Address Pointers

In many cases, we would see that the register or address we are using does not immediately contain the final value but contains another address that points to the final value. This is always the case with _pointer_ registers, like `rsp`, `rbp`, and `rip`,

For example, let's assemble and run `gdb` on our assembled `fib` binary, and check the `rsp` and `rip` registers:

```
gdb -q ./fib
gef➤  b _start
Breakpoint 1 at 0x401000
gef➤  r
...SNIP...
$rsp   : 0x00007fffffffe490  →  0x0000000000000001
$rip   : 0x0000000000401000  →  <_start+0> mov eax, 0x0
```

We see that both registers contain pointer addresses to other locations.

**Moving Pointer Values**





## Arithmetic Instructions

### Unary Instructions

The following are the main Unary Arithmetic Instructions (we will assume that `rax` starts as `1` for each instruction):

<table><thead><tr><th width="118">Instruction</th><th>Description</th><th>Example</th></tr></thead><tbody><tr><td><code>inc</code></td><td>Increment by 1</td><td><code>inc rax</code> -> <code>rax++</code> or <code>rax += 1</code> -> <code>rax = 2</code></td></tr><tr><td><code>dec</code></td><td>Decrement by 1</td><td><code>dec rax</code> -> <code>rax--</code> or <code>rax -= 1</code> -> <code>rax = 0</code></td></tr></tbody></table>
