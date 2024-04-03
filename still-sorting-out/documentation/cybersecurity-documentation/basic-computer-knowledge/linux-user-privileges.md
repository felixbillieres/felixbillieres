# 🗝️ Linux User Privileges

## Beginner's Guide to Linux User Privileges

### Introduction:

Understanding and managing user privileges in Linux is essential for effective system administration. This guide will walk you through the basics of permissions, ownership, and important files related to user management.

#### 1. **File Permissions:**

In Linux, every file and directory has associated permissions that control who can read, write, and execute them. The `chmod` command is used to change these permissions.

**Symbolic Representation:**

* **`+`**: Add permission.
* **`-`**: Remove permission.
* **`r`**: Read permission.
* **`w`**: Write permission.
* **`x`**: Execute permission.

**Example:**

```bash
chmod +rwx test.txt
```

This grants read, write, and execute permissions to the file owner. Others have no permissions.

**Numeric Representation:**

<figure><img src="https://miro.medium.com/v2/resize:fit:1400/1*Qd9k5fOi4crDc33l0VveaQ.png" alt=""><figcaption></figcaption></figure>

**Example:**

```bash
chmod 777 test.txt
```

This gives full permissions (read, write, execute) to the file owner, group, and others.

#### 2. **Important Files:**

**- `/etc/passwd`:**

Contains information about user accounts, including usernames and user IDs (UIDs).

**- `/etc/shadow`:**

Stores password hashes for user accounts. Accessible only by privileged users.

**- `/etc/sudoers`:**

Controls user permissions for executing commands with `sudo`.

**- `/etc/group`:**

Contains information about groups, including group names and group IDs (GIDs).

#### 3. **Checking Sudo Access:**

```bash
grep 'sudo' /etc/group
```

This command checks for users who have sudo access, as users in the "sudo" group usually have elevated privileges.

***

#### 4. **User and Group Ownership:**

*   **Every file has an owner and group:** Use `ls -l` to view file details. The output shows the owner, group, and permissions.

    ```bash
    ls -l test.txt
    ```
*   **Change ownership with `chown`:**

    ```bash
    chown newowner:newgroup test.txt
    ```

#### 5. **User Groups:**

* **Linux uses groups to organize users:** A user can belong to multiple groups.
*   **Check groups a user belongs to:**

    ```bash
    groups username
    ```
*   **Add a user to a group:**

    ```bash
    usermod -aG groupname username
    ```

#### 6. **umask:**

*   **Defines default permissions for new files:** It subtracts the umask value from the maximum permissions (666 for files, 777 for directories).

    ```bash
    umask
    ```

#### 7. **Setuid, Setgid, and Sticky Bit:**

*   **Setuid (SUID):** Allows a user to execute a file with the permissions of the file owner.

    ```bash
    chmod u+s filename
    ```
*   **Setgid (SGID):** Allows a user to inherit the group ownership of the file group.

    ```bash
    chmod g+s dirname
    ```
*   **Sticky Bit:** Protects directories from deletion by users other than the file owner.

    ```bash
    chmod +t dirname
    ```

#### 8. **Access Control Lists (ACLs):**

*   **Provide more fine-grained control:** Allows specifying permissions for multiple users and groups.

    ```bash
    getfacl filename
    ```

#### 9. **su and sudo:**

*   **`su` (Switch User):** Allows switching to another user with the specified shell.

    ```bash
    su username
    ```
*   **`sudo` (Superuser Do):** Allows authorized users to execute commands as the superuser.

    ```bash
    sudo command
    ```

#### 10. **PAM (Pluggable Authentication Modules):**

*   **Manages authentication tasks:** Controls access, sets session limits, and more.

    ```bash
    cat /etc/pam.d/login
    ```

#### 11. **Secure File Permissions:**

* **Be cautious with `chmod 777`:** Giving all permissions is risky; only use it when absolutely necessary.
* **Keep sensitive files secure:** Limit access to critical system files and directories.

#### 12. **Monitor Logs:**

*   **Check system logs:** Review `/var/log/auth.log` for authentication-related information.

    ```bash
    tail -f /var/log/auth.log
    ```

### Conclusion:

Understanding and managing Linux user privileges is crucial for maintaining a secure and organized system. With the knowledge of file permissions, ownership, and key configuration files, you can navigate user management more effectively. As you explore further, you'll gain confidence in handling permissions and ensuring a well-configured Linux environment.
