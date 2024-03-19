# ➰ Scanning & Enumeration (Kioptrix)

## Scanning and Enumerating the Kioptrix Machine

VulnHub is a platform that provides various vulnerable virtual machines (VMs) for cybersecurity enthusiasts and professionals to practice and enhance their penetration testing skills. One of the popular series of VMs available on VulnHub is Kioptrix, created by a security professional named "Kioptrix." These VMs are intentionally designed with security vulnerabilities, allowing users to identify, exploit, and patch them in a controlled environment.

### Kioptrix VM Information

* **Download Link:** [Kioptrix VM](https://tcm-sec.com/kioptrix)
* **Credentials:**
  * Username: john
  * Password: TwoCows2

### Target IP Address

To determine the IP address of the Kioptrix machine, we can perform a ping to a known IP, such as 8.8.8.8, and observe the response. In this case, let's assume the IP address is `192.168.179.129`.

```bash
ping 8.8.8.8
```

Response:

```
192.168.179.129
```

### Network Discovery with Netdiscover

Use the `netdiscover` tool to scan the local network and identify hosts.

```bash
netdiscover -r 192.168.179.0/24
```

Result:

```
IP               MAC Address         Vendor
----------------------------------------------
192.168.179.1    00:50:56:c0:00:08   VMware, Inc.
192.168.179.2    00:50:56:f9:11:f4   VMware, Inc.
192.168.179.129  00:0c:29:a5:da:ef   VMware, Inc.
192.168.179.254  00:50:56:fe:96:dd   VMware, Inc.
```

The Kioptrix machine is likely at IP `192.168.179.129`.

### Enumerating Open Ports

Perform an initial port scan using Nmap to identify open ports and services.

```bash
nmap -sC -sV 192.168.179.129
```

Results:

```
PORT      STATE SERVICE     VERSION
22/tcp    open  ssh         OpenSSH 2.9p2
80/tcp    open  http        Apache httpd 1.3.20 (Unix) mod_ssl/2.8.4 OpenSSL/0.9.6b
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
```

### Web Server Enumeration

Use Nikto to perform a web server scan and gather information about potential vulnerabilities.

```bash
nikto -h http://192.168.179.129
```

Review the Nikto scan results for information on the web server and identified vulnerabilities.

### Directory Brute-Forcing

Use DirBuster or another directory brute-forcing tool to discover hidden directories and files on the web server.

```bash
dirbuster&
```

Explore the discovered directories for potential entry points and vulnerabilities.

### SMB Enumeration

Enumerate SMB shares using the `smbclient` tool to gather information about the Samba server.

```bash
smbclient -L \\192.168.179.129
```

Explore the available shares, and attempt to connect to them for further investigation.

### SSH Version

Identify the version of the SSH service using the following command:

```bash
ssh -V 192.168.179.129
```

Note the SSH version for potential research on vulnerabilities associated with that version.

### Potential Vulnerabilities

Research potential vulnerabilities associated with the identified versions of services, including:

* Apache httpd 1.3.20 with mod\_ssl/2.8.4 OpenSSL/0.9.6b
* Samba smbd 2.2.1a
* OpenSSH 2.9p2

Explore known exploits and vulnerabilities associated with these versions to plan further penetration testing activities.

Proceed with caution, and always ensure you have the necessary permissions to perform penetration testing on the target system.
