# 😑 Architecture

Assembly language is as a low-level language that can write direct instructions the processors can understand.

By using Assembly, developers can write human-readable machine instructions, which are then assembled into their machine code equivalent, so that the processor can directly run them. This is why some refer to Assembly language as symbolic machine code. For example, the Assembly code '`add rax, 1`' is much more intuitive and easier to remember than its equivalent machine shellcode '`4883C001`', and easier to remember than the equivalent binary machine code '`01001000 10000011 11000000 00000001'`

Here are Compilation stages ->

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

If we run this Python line, it would be essentially executing the following `C` code, then the `C` code uses the Linux `write` syscall, built-in for processes to write to the screen. The same syscall called in Assembly, then the Assembly code can be assembled into the following hex machine code and Finally, for the processor to execute the instructions linked to this machine, it would have to be translated into binary

## Computer Architecture

The architecture executes machine code to perform specific algorithms. It mainly consists of the following elements

* Central Processing Unit (CPU)
* Memory Unit
* Input/Output Devices
  * Mass Storage Unit
  * Keyboard
  * Display

Furthermore, the CPU itself consists of three main components:

* Control Unit (CU)
* Arithmetic/Logic Unit (ALU)
* Registers

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Memory

That's where the `temporary` data and instructions of currently running programs are located. It is the primary location the CPU uses to retrieve and process data.

There are two main types of memory:

1. `Cache`
2. `Random Access Memory (RAM)`

**Cache**

Cache memory is usually located within the CPU itself and hence is extremely fast compared to RAM, as it runs at the same clock speed as the CPU.

**RAM**

RAM is also located far away from the CPU cores and is much slower than cache memory. Accessing data from RAM addresses takes many more instructions.

When a program is run, all of its data and instructions are moved from the storage unit to the RAM to be accessed when needed by the CPU. This happens because accessing them from the storage unit is much slower and will increase data processing times.

the RAM is split into four main `segments`

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

<table><thead><tr><th width="105">Segment</th><th>Description</th></tr></thead><tbody><tr><td><code>Stack</code></td><td>Has a Last-in First-out (LIFO) design and is fixed in size. Data in it can only be accessed in a specific order by push-ing and pop-ing data.</td></tr><tr><td><code>Heap</code></td><td>Has a hierarchical design and is therefore much larger and more versatile in storing data, as data can be stored and retrieved in any order. However, this makes the heap slower than the Stack.</td></tr><tr><td><code>Data</code></td><td>Has two parts: <code>Data</code>, which is used to hold variables, and <code>.bss</code>, which is used to hold unassigned variables (i.e., buffer memory for later allocation).</td></tr><tr><td><code>Text</code></td><td>Main assembly instructions are loaded into this segment to be fetched and executed by the CPU.</td></tr></tbody></table>

### IO/Storage

the Input/Output devices, like the keyboard, the screen, or the long-term storage unit, also known as Secondary Memory. The processor can access and control IO devices using `Bus Interfaces`, which act as 'highways' to transfer data and addresses, using electrical charges for binary data.
