---
description: https://guyinatuxedo.github.io/01-intro_assembly/assembly/index.html
---

# 0️⃣ Assembly -> Reverse

### **How do processors work?**

#### What are CPUs?

**Definition**: A Central Processing Unit (CPU) is the primary component of a computer that performs most of the processing inside the computer. It executes instructions from programs through basic arithmetic, logic, control, and input/output (I/O) operations specified by the instructions.

**Key Points**:

* Often referred to as the "brain" of the computer.
* Contains components like the Arithmetic Logic Unit (ALU) and Control Unit (CU).
* Executes a sequence of stored instructions called a program.
* Manages data processing, memory access, and task scheduling.

#### What are Registers?

**Definition**: Registers are small, fast storage locations within the CPU that hold data, addresses, or instructions temporarily. They are used by the CPU to quickly access and manipulate data during instruction execution.

**Key Points**:

* Faster than main memory (RAM).
* Include general-purpose registers (e.g., EAX, EBX, ECX, EDX in x86 architecture) for arithmetic and data storage.
* Special-purpose registers include the Instruction Pointer (EIP) which holds the address of the next instruction to execute, and the Stack Pointer (ESP) which points to the top of the stack.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

#### What is the Stack?

**Definition**: The stack is a data structure used in computer memory to store data temporarily. It operates on a Last In, First Out (LIFO) basis, meaning the last element added is the first to be removed.

**Key Points**:

* Used for function call management, storing return addresses, local variables, and passing parameters.
* Managed via the Stack Pointer (ESP in x86), which points to the top of the stack.
* Stack operations include `PUSH` (to add data to the stack) and `POP` (to remove data from the stack).
* Crucial for managing function calls and supporting recursion.

#### What is ELF?

**Definition**: ELF stands for Executable and Linkable Format. It is a standard file format for executables, object code, shared libraries, and core dumps used in Unix-like operating systems.

**Key Points**:

* **Purpose**: Facilitates the storage and loading of binary executables and shared libraries. It standardizes how these files are structured.

#### Assembly Code (`ex1.asm`)

```assembly
global _start 
_start:
        mov eax, 1
        mov ebx, 42
        int 0x80
```

**Line-by-Line Explanation**:

1. **`global _start`**:
   * This line declares the `_start` label as global, which means it can be used as an entry point when linking. This is necessary because the linker needs to know where the execution of the program should begin.
2. **`_start:`**:
   * This label marks the beginning of the program. The linker will set this as the entry point for the executable.
3. **`mov eax, 1`**:
   * This instruction moves the value `1` into the `eax` register. In Linux system calls, setting `eax` to `1` indicates that we want to call the `exit` system call.
4. **`mov ebx, 42`**:
   * This instruction moves the value `42` into the `ebx` register. For the `exit` system call, `ebx` holds the exit status code. Here, `42` is the exit code that the program will return.
5. **`int 0x80`**:
   * This instruction triggers a software interrupt, signaling the kernel to execute a system call. The `eax` register value determines which system call to perform, and `ebx` provides the argument for the system call.

#### Commands

1. **`nasm -f elf32 ex1.asm -o ex1.o`**:
   * This command assembles the assembly source file `ex1.asm` into an object file `ex1.o` using the Netwide Assembler (NASM).
   * `-f elf32`: Specifies the output format as 32-bit ELF (Executable and Linkable Format).
   * `-o ex1.o`: Specifies the output file name.
2. **`ld -m elf_i386 ex1.o -o ex1`**:
   * This command links the object file `ex1.o` to create an executable named `ex1`.
   * `-m elf_i386`: Specifies the target architecture as 32-bit Intel (i386).
   * `-o ex1`: Specifies the output file name for the executable.
3. **`./ex1`**:
   * This command runs the newly created executable `ex1`.
4. **`echo $?`**:
   * This command prints the exit status of the last executed command (`./ex1` in this case). The exit status is the value set in the `ebx` register when the `exit` system call was invoked.

#### Expected Output

When `./ex1` runs, the following happens:

* The program calls the `exit` system call with `42` as the exit status.
* The `echo $?` command then prints `42`, the exit status of `./ex1`.

**Play with the code ->**

```
global _start 
_start:
        mov eax, 1
        mov ebx, 42
        sub ebx, 29
        int 0x80
```

the sub ebx, 29 part will substract 29 to the ebx register wich is the output register, altering the output to 13 here

Here are some other assembly instructions:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://guyinatuxedo.github.io/01-intro_assembly/assembly/index.html#instructions" %}

* When you use the `mul` instruction with a register (e.g., `ebx`), it implicitly uses `eax` as the left operand.
* The right operand is the register or memory location you specify (e.g., `ebx`).
* The instruction `mul ebx` performs the following multiplication:

```
EAX = EAX * EBX
```

The result is stored in the `EDX:EAX` pair, where:

* `EAX` holds the lower 32 bits of the result.
* `EDX` holds the upper 32 bits of the result (if the result is larger than 32 bits).

```
mov eax, 10     ; Set EAX to 10
mov ebx, 20     ; Set EBX to 20
mul ebx         ; EAX = EAX * EBX (10 * 20)
                ; Result: EAX = 200, EDX = 0 (since 200 fits within 32 bits)
```

and for div

```
mov edx, 0      ; Set EDX to 0 (for a clean dividend)
mov eax, 200    ; Set EAX to 200 (dividend = 200)
mov ebx, 10     ; Set EBX to 10 (divisor = 10)
div ebx         ; EAX = (EDX:EAX) / EBX = 200 / 10
                ; EAX = 20, EDX = 0 (remainder 0)
```



### Hello world!

```
global _start

section .data
    msg db "Hello, world!", 0x0a
    len equ $ - msg
    
section .text
_start:
    mov eax, 4
    mov ebx, 1
    mov ecx, msg
    mov edx, len
    mov int, 0x80
    mov eax, 1
    mov ebx, 0
    int 0x80
```

**Line-by-Line Explanation**:

1. **`global _start`**:
   * This line declares the `_start` label as global, making it the entry point for the program. The linker needs this to know where the execution begins.
2. **`section .data`**:
   * This section is used to declare initialized data or constants. Data in this section does not change at runtime.
3. **`msg db "Hello, world!", 0x0a`**:
   * `msg` is a label for the data "Hello, world!" followed by a newline character (`0x0a` in hexadecimal, which is the ASCII code for a newline).
4. **`len equ $ - msg`**:
   * `len` is defined as the length of the string `msg`. `$` represents the current address, and `msg` is the address of the beginning of the string. The difference gives the length of the string.
5. **`section .text`**:
   * This section contains the code (instructions) of the program.
6. **`_start:`**:
   * This label marks the entry point of the program. The execution starts here.

#### System Call to Write

1. **`mov eax, 4`**:
   * Moves the value `4` into the `eax` register. In Linux, `4` is the syscall number for `sys_write`.
2. **`mov ebx, 1`**:
   * Moves the value `1` into the `ebx` register. `1` is the file descriptor number for standard output (stdout).
3. **`mov ecx, msg`**:
   * Moves the address of `msg` (the string "Hello, world!\n") into the `ecx` register. `ecx` will point to the data to be written.
4. **`mov edx, len`**:
   * Moves the value of `len` (the length of the string `msg`) into the `edx` register. `edx` specifies the number of bytes to write.
5. **`int 0x80`**:
   * Triggers a software interrupt to invoke the kernel. The kernel interprets the values in the registers and performs the `write` syscall, writing "Hello, world!\n" to the standard output.

#### System Call to Exit

1. **`mov eax, 1`**:
   * Moves the value `1` into the `eax` register. In Linux, `1` is the syscall number for `sys_exit`.
2. **`mov ebx, 0`**:
   * Moves the value `0` into the `ebx` register. `0` is the exit status code.
3. **`int 0x80`**:
   * Triggers a software interrupt to invoke the kernel. The kernel interprets the values in the registers and performs the `exit` syscall, terminating the program with a status code of `0`.
