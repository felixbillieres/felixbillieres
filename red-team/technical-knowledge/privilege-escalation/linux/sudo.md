# 🏃‍♂️ Sudo

### Sudo Shell Escaping

Sudo Shell Escaping refers to a situation where an attacker, after gaining access to a system with limited privileges, leverages the `sudo` command to run a shell with elevated privileges. The `sudo` command allows a permitted user to execute a command as the superuser (or another user), as specified by the security policy configured in the `/etc/sudoers` file.

When you run `sudo -l`, it lists the allowed (or forbidden) commands for the invoking user on the current host. This information is valuable for an attacker because it provides insights into potential paths for privilege escalation.

If the output of `sudo -l` reveals that you have permission to run a specific command as a superuser, you might be able to exploit this privilege to gain a root shell. One common approach is to look for opportunities to escape the restricted environment and execute arbitrary commands.

Here's a step-by-step process:

1.  **Run `sudo -l`**: Identify the commands you are allowed to run with elevated privileges.

    <figure><img src="../../../../../.gitbook/assets/image (311).png" alt=""><figcaption><p>let's take vim as an example</p></figcaption></figure>
2. **Check gtfobins**: Visit [gtfobins.github.io](https://gtfobins.github.io/) to find techniques and examples of exploiting specific commands to escalate privileges.
3.  **Select appropriate technique**: Look for the command you have sudo access to on gtfobins. The site provides detailed explanations and commands to exploit these commands for privilege escalation.

    <figure><img src="../../../../../.gitbook/assets/image (312).png" alt=""><figcaption></figcaption></figure>
4.  **Execute the exploit**: Use the provided commands on gtfobins to attempt privilege escalation. If successful, you may gain a shell with elevated privileges.

    <figure><img src="../../../../../.gitbook/assets/image (313).png" alt=""><figcaption><p>after typing the (a) </p></figcaption></figure>



Let's try with the 'awk' method, since we have sudo rights on it:

<figure><img src="../../../../../.gitbook/assets/image (315).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (314).png" alt=""><figcaption></figcaption></figure>

same for 'find'

<figure><img src="../../../../../.gitbook/assets/image (317).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/image (316).png" alt=""><figcaption></figcaption></figure>

### Intended Functionality <a href="#lecture_heading" id="lecture_heading"></a>

Escalation via Intended Functionality is a technique where an attacker exploits the legitimate features and functions of a system to gain unauthorized access or elevated privileges. It typically involves manipulating existing functionalities in a way that was not intended by the system designers. This method relies on understanding the system's architecture and features to identify opportunities for abuse.

Here are a few examples to illustrate Escalation via Intended Functionality:

1. **Insecure Defaults**: Some systems or applications come with default configurations that might be insecure. Attackers may exploit these defaults to gain unauthorized access. For example, a system might have default credentials that are not changed by the administrator.
2. **Misconfigured Permissions**: If a system has misconfigured permissions, an attacker may be able to escalate privileges. For instance, a web application might store sensitive files in a directory with lax permissions, allowing an attacker to read or modify those files.
3. **Abusing Trusted Services**: Sometimes, systems rely on trusted services or components. If an attacker can manipulate or abuse these trusted components, it may lead to privilege escalation. For example, abusing trust relationships between different systems or services.
4. **Command Injection**: Applications that allow user input to be directly passed to system commands can be vulnerable to command injection. An attacker may provide malicious input that tricks the system into executing unintended commands with elevated privileges.
5. **File Upload Vulnerabilities**: Web applications that allow users to upload files may have vulnerabilities. An attacker could upload malicious files containing code that, when executed by the server, grants unauthorized access.
6. **Race Conditions**: Certain race conditions might occur when multiple processes or threads are racing to access shared resources. Exploiting these conditions can lead to unintended consequences, such as privilege escalation.
7. **Business Logic Flaws**: Understanding the business logic of an application or system can reveal opportunities for exploitation. For example, an e-commerce site may have flaws in its checkout process that allow an attacker to manipulate the transaction flow.
8. **Feature Abuse**: Some features of a system may have unintended consequences when used in a certain way. Attackers may abuse features to manipulate or bypass security controls.

#### Here's a step-by-step process:

It's pretty much like the first part with the `sudo -l` command

<figure><img src="../../../../../.gitbook/assets/image (318).png" alt=""><figcaption></figcaption></figure>

We see the apache2 sudo parameter

Let's quickly google that "sudo apache2 privesc"

{% embed url="https://touhidshaikh.com/blog/2018/04/abusing-sudo-linux-privilege-escalation/" %}

<figure><img src="../../../../../.gitbook/assets/image (319).png" alt=""><figcaption></figcaption></figure>

we were not able to gain a root shell but if we can get the hash of root it's already very good:

<figure><img src="../../../../../.gitbook/assets/image (320).png" alt=""><figcaption></figcaption></figure>

### LD\_PRELOAD <a href="#lecture_heading" id="lecture_heading"></a>

**Exploiting LD\_PRELOAD for Privilege Escalation**

`LD_PRELOAD` is an environment variable in Linux that specifies a list of shared libraries to be loaded before all others. This feature is often used to override or extend the functionality of certain functions in a program. However, if misconfigured, it can be exploited by attackers for privilege escalation.

#### How LD\_PRELOAD Works:

When a program starts, the dynamic linker/loader (ld.so) is responsible for loading shared libraries. The `LD_PRELOAD` variable allows users to specify additional shared libraries to be loaded, effectively intercepting calls to functions. This can be powerful for legitimate use cases, such as implementing debugging tools or modifying program behavior. However, if an attacker gains control over `LD_PRELOAD`, they can load a malicious library to compromise the system.

#### Abusing LD\_PRELOAD for Privilege Escalation:

1. **Create a Malicious Shared Library:**
   * Craft a shared library containing code that grants elevated privileges or performs malicious actions.
   * For example, a shared library could override the `open()` function to allow the attacker to read or write to sensitive files.
2.  **Set LD\_PRELOAD:**

    * Set the `LD_PRELOAD` environment variable to point to the location of the malicious shared library.
    * This can be done in various ways, such as exporting the variable before executing a command or modifying startup files.

    ```bash
    export LD_PRELOAD=/path/to/malicious_library.so
    ```
3. **Execute the Target Program:**
   * Run a program that uses the manipulated shared library.
   * The target program will load the attacker's library, and the overridden functions will execute the malicious code.

#### Example Scenario:

Consider a system where a setuid binary (binary with elevated privileges) is vulnerable to exploitation. An attacker could create a shared library that overrides a function called by the setuid binary. By setting `LD_PRELOAD` to load the malicious library, the attacker can manipulate the behavior of the setuid binary and potentially gain unauthorized access.

#### Here's a step-by-step process:

```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>

void _init(){
 unsetenv("LD_PRELOAD");
 setgid(0);
 setuid(0);
 system("/bin/bash");
}
```

<figure><img src="../../../../../.gitbook/assets/image (321).png" alt=""><figcaption><p>we compile and run the library </p></figcaption></figure>

Explanation:

1. **`unsetenv("LD_PRELOAD");`**: This line removes the `LD_PRELOAD` environment variable, ensuring that no other shared libraries are preloaded before this one. This helps prevent interference with the library's intended actions.
2. **`setgid(0);`**: This line sets the effective group ID (GID) to 0, which is the root group. This is a privilege escalation step, granting the process elevated group privileges.
3. **`setuid(0);`**: This line sets the effective user ID (UID) to 0, which is the root user. This is another privilege escalation step, granting the process elevated user privileges.
4. **`system("/bin/bash");`**: Finally, this line uses the `system()` function to execute the `/bin/bash` command, spawning a Bash shell with root privileges. This effectively provides the user with a command prompt as the root user.

This code, when compiled into a shared library and loaded, would potentially allow an attacker to gain unauthorized root access to a system. The use of `setgid(0)` and `setuid(0)` is a classic privilege escalation technique. It's important to note that such code should be treated with extreme caution, as it can lead to serious security vulnerabilities if misused. In a secure environment, preventing the loading of unauthorized shared libraries and closely monitoring setuid/setgid binaries are essential measures.
