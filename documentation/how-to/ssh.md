# 🔑 SSH

## SSH Enumeration and Interaction

Secure Shell (SSH) is a widely used cryptographic network protocol for secure communication over an unsecured network. This page covers the basics of SSH enumeration, connecting to SSH, and important considerations.

<figure><img src="https://www.hostinger.com/tutorials/wp-content/uploads/sites/2/2017/07/symmetric-encryption-ssh-tutorial.jpg" alt=""><figcaption></figcaption></figure>

### **1. Enumerating SSH**

SSH enumeration involves identifying systems that have SSH services running and gathering information about them. Tools like `nmap` and `enum4linux` are commonly used for SSH enumeration.

#### **Using nmap:**

```bash
nmap -p 22 --script ssh-brute,ssh-hostkey,ssh-run <target_ip>
```

This nmap command performs SSH enumeration, including brute-force attempts and collecting host key information.

#### **Using enum4linux:**

```bash
enum4linux -a <target_ip>
```

The `enum4linux` tool provides additional information about the target, including details about SMB and SSH services.

### **2. Connecting to SSH**

Connecting to SSH allows you to establish a secure shell session with a remote system. The `ssh` command is the standard tool for connecting to SSH-enabled hosts.

#### **Using ssh:**

```bash
ssh username@target_ip
```

Replace "username" with the appropriate username and enter the password when prompted. You can also use key-based authentication for added security.

### **3. Interacting with SSH Sessions**

Once connected to an SSH session, you can interact with the remote system's command-line interface.

#### **Common SSH Interactions:**

* Run commands on the remote system.
* Transfer files using `scp` (secure copy).
* Configure and manage the remote server.

### **4. SSH Security Considerations**

* **Key-based Authentication**: Prefer key-based authentication over password-based authentication for enhanced security.
* **Disable Root Login**: Disable direct root login and use a regular user with sudo privileges.
* **Update and Patch**: Keep the SSH server software updated to patch known vulnerabilities.
* **Limit User Access**: Grant SSH access only to necessary users.
* **Monitor Logs**: Regularly monitor SSH logs for suspicious activities.
