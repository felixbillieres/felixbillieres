# 🎷 RDP

## RDP Enumeration and Interaction

Remote Desktop Protocol (RDP) is a proprietary protocol developed by Microsoft, which provides a user with a graphical interface to connect to another computer over a network connection. This page covers the basics of RDP enumeration, connecting to RDP sessions, and important considerations.

<figure><img src="https://www.layerstack.com/img/docs/resources/rdpdiagram.jpg" alt=""><figcaption></figcaption></figure>

### **1. Enumerating RDP**

RDP enumeration involves identifying systems that have RDP services running and gathering information about them. Tools like `nmap` and `rdp-enum-encryption` are commonly used for RDP enumeration.

#### **Using nmap:**

```bash
nmap -p 3389 --script rdp-enum-encryption <target_ip>
```

This command uses nmap to perform RDP enumeration and identify potential security vulnerabilities.

#### **Using rdp-enum-encryption:**

```bash
rdp-enum-encryption <target_ip>
```

This tool is specifically designed for RDP enumeration, providing information about encryption levels supported by the target.

### **2. Connecting to RDP Sessions**

Connecting to RDP sessions allows you to interact with a remote system's desktop environment. You can use the built-in Remote Desktop Connection client on Windows or tools like `xfreerdp` on Linux.

#### **Using Remote Desktop Connection (Windows):**

1. Open Remote Desktop Connection.
2. Enter the target IP address and click "Connect."
3. Enter your credentials when prompted.

#### **Using xfreerdp (Linux):**

```bash
xfreerdp /v:<target_ip>
```

This command opens an RDP session to the specified target IP address.

### **3. Interacting with RDP Sessions**

Once connected to an RDP session, you can interact with the remote desktop as if you were physically present.

#### **Common RDP Interactions:**

* Use the mouse and keyboard to control the remote desktop.
* Transfer files between local and remote machines.
* Run commands and applications on the remote system.

### **4. RDP Security Considerations**

* **Network Security**: Ensure RDP traffic is encrypted and secured over the network.
* **Strong Authentication**: Use strong and unique credentials for RDP access.
* **Network Level Authentication (NLA)**: Enable NLA to add an additional layer of security.
* **Firewall Rules**: Adjust firewall rules to permit RDP traffic only from trusted sources.
* **Account Lockout Policies**: Implement account lockout policies to defend against brute force attacks.
