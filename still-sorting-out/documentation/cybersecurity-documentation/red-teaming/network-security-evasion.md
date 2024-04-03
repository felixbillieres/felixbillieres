# 🏈 Network Security Evasion

An Intrusion Detection System (IDS) is a system that detects network or system intrusions. He can spot a theft, but he cannot stop it by himself.

An Intrusion Detection and Prevention System (IDPS) or simply Intrusion Prevention System (IPS) is a system that can detect and prevent intrusions.

Snort can be set up as an IDS or an IPS. For Snort to function as an IPS, it needs some mechanism to block (`drop`) offending connections. This capability requires Snort to be set up as `inline` and to bridge two or more network cards.

As a signature-based network IDS, Snort is shown in the figure below.

<figure><img src="https://tryhackme-images.s3.amazonaws.com/user-uploads/5f04259cf9bf5b57aed2c476/room-content/112f5abb83ffd40a8ce514980242ce60.png" alt=""><figcaption></figcaption></figure>

The following figure shows how Snort can be configured as an IPS if set up inline.

<figure><img src="https://tryhackme-images.s3.amazonaws.com/user-uploads/5f04259cf9bf5b57aed2c476/room-content/0925b08163ea115da03213b6fd846296.png" alt=""><figcaption></figcaption></figure>

IDS setups can be divided based on their location in the network into:

1. Host-based IDS (HIDS)
2. Network-based IDS (NIDS)

The host-based IDS (HIDS) is installed on an OS along with the other running applications. This setup will give the HIDS the ability to monitor the traffic going in and out of the host; moreover, it can monitor the processes running on the host.

The network-based IDS (NIDS) is a dedicated appliance or server to monitor the network traffic. The NIDS should be connected so that it can monitor all the network traffic of the network or VLANs we want to protect. This can be achieved by connecting the NIDS to a monitor port on the switch. The NIDS will process the network traffic to detect malicious traffic.

In the figure below, we use two red circles to show the difference in the coverage of a HIDS versus a NIDS.

<figure><img src="https://tryhackme-images.s3.amazonaws.com/user-uploads/5f04259cf9bf5b57aed2c476/room-content/61e14978e7f97e0de5d467babc3cfbff.png" alt=""><figcaption></figcaption></figure>

### IDS Engine Types

We can classify network traffic into:

1. Benign traffic: This is the usual traffic that we expect to have and don’t want the IDS to alert us about.
2. Malicious traffic: This is abnormal traffic that we don’t expect to see under normal conditions and consequently want the IDS to detect it.

Signature-based IDS recognizes malicious traffic, so everything that is not malicious is considered benign (normal). This approach is commonly found in antivirus software, which has a database of known virus signatures. Anything that matches a signature is detected as a virus.

An anomaly-based IDS recognize normal traffic, so anything that deviates from normal is considered malicious.

### IDS/IPS Rule Triggering

Snort uses the following format for its rules: `Rule Header (Rule Options)`, where **Rule Header** constitutes:

1. Action: Examples of action include `alert`, `log`, `pass`, `drop`, and `reject`.
2. Protocol: `TCP`, `UDP`, `ICMP`, or `IP`.
3. Source IP/Source Port: `!10.10.0.0/16 any` refers to everything not in the class B subnet `10.10.0.0/16`.
4. Direction of Flow: `->` indicates left (source) to right (destination), while `<>` indicates bi-directional traffic.
5. Destination IP/Destination Port: `10.10.0.0/16 any` to refer to class B subnet `10.10.0.0/16`.

Below is an example rule to `drop` all ICMP traffic passing through Snort IPS:

`drop icmp any any -> any any (msg: "ICMP Ping Scan"; dsize:0; sid:1000020; rev: 1;)`

Snort rule that detects the term `ncat` in the payload of the traffic exchanged with our webserver to learn how people exploit this vulnerability.

`alert tcp any any <> any 80 (msg: "Netcat Exploitation"; content:"ncat"; sid: 1000030; rev:1;)`

The rule above inspects the content of the packets exchanged with port 80 for the string `ncat`. Alternatively, you can choose to write the content that Snort will scan for in hexadecimal format. `ncat` in ASCII is written as `6e 63 61 74` in hexadecimal and it is encapsulated as a string by 2 pipe characters `|`.

`alert tcp any any <> any 80 (msg: "Netcat Exploitation"; content:"|6e 63 61 74|"; sid: 1000031; rev:1;)`

## Network Infrastructure

Once arriving onto an unknown network, our first goal is to identify where we are and what we can get to. During the red team engagement, we need to understand what target system we are dealing with, what service the machine provides, what kind of network we are in.

### Internal Networks

Internal Networks are subnetworks that are segmented and separated based on the importance of the internal device or the importance of the accessibility of its data. The main purpose of the internal network(s) is to share information, faster and easier communications, collaboration tools, operational systems, and network services within an organization.

<figure><img src="../../../../.gitbook/assets/image (374).png" alt=""><figcaption><p>network segmentation as the network is divided into two networks.</p></figcaption></figure>

first one is for employee workstations and personal devices. The second is for private and internal network devices that provide internal services such as DNS, internal web, email services, etc.

### A Demilitarized Zone (DMZ)

A DMZ Network is an edge network that protects and adds an extra security layer to a corporation's internal local-area network from untrusted traffic.

a company provides public services such as a website, DNS, FTP, Proxy, VPN, etc. In that case, they may design a DMZ network to isolate and enable access control on the public network traffic, untrusted traffic.

<figure><img src="../../../../.gitbook/assets/image (375).png" alt=""><figcaption></figcaption></figure>

Network traffic to the DMZ network in red color, which is untrusted (comes directly from the internet). The green network traffic between the internal network is the controlled traffic that may go through one or more than one network security device(s).

### Network Enumeration

Let's start checking the target machine's TCP and UDP open ports.

```
PS C:\Users\thm> netstat -na

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:88             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:389            0.0.0.0:0              LISTENING
```

The output reveals the open ports as well as the established connections.

**TCP 0.0.0.0:80 0.0.0.0:0 LISTENING:**

* **Local Address (`0.0.0.0:80`):** This indicates that the system is listening on all available network interfaces (`0.0.0.0`) on port 80.
* The "Foreign Address" (`0.0.0.0:0`) in the listening state typically means that the system is willing to accept connections from any IP address.

"Foreign Address" refers to the address of the remote system or device with which your local system is communicating. It represents the endpoint of the connection from the perspective of your system.

Next, let's list the ARP table, which contains the IP address and the physical address of the computers that communicated with the target machines within the network. This could be helpful to see the communications within the network to scan the other machines for open ports and vulnerabilities.

```
PS C:\Users\thm> arp -a

Interface: 10.10.141.51 --- 0xa
  Internet Address      Physical Address      Type
  10.10.0.1             02-c8-85-b5-5a-aa     dynamic
  10.10.255.255         ff-ff-ff-ff-ff-ff     static
```

1. **Interface: 10.10.141.51 --- 0xa:**
   * This section provides information about the network interface with the IP address 10.10.141.51.
2. **ARP Table Entries:**
   * **Internet Address (`10.10.0.1`):** This is an entry in the ARP cache. It indicates that the IP address 10.10.0.1 is associated with the following MAC (Physical) address: `02-c8-85-b5-5a-aa`. The "dynamic" type indicates that this mapping was dynamically learned and may change over time.
   * **Internet Address (`10.10.255.255`):** Another entry in the ARP cache. It associates the broadcast IP address 10.10.255.255 with the MAC address `ff-ff-ff-ff-ff-ff`. The "static" type indicates that this mapping is static and unlikely to change frequently. This entry is often used for broadcast communication within the local network.

### Active Directory (AD) environment

<figure><img src="../../../../.gitbook/assets/image (376).png" alt=""><figcaption></figcaption></figure>

The diagram is one possible example of how Active Directory can be designed. The AD controller is placed in a subnet for servers (shown above as server network), and then the AD clients are on a separate network where they can join the domain and use the AD services via the firewall.

Terms to know:&#x20;

* Domain Controllers
* Organizational Units
* AD objects
* AD Domains
* Forest
* AD Service Accounts: Built-in local users, Domain users, Managed service accounts
* Domain Administrators

**A Domain Controller** is a Windows server that provides Active Directory services and controls the entire domain. It is a form of centralized user management that provides encryption of user data as well as controlling access to a network, including users, groups, policies, and computers. It also enables resource access and sharing.

<figure><img src="../../../../.gitbook/assets/image (377).png" alt=""><figcaption></figcaption></figure>

**Organizational Units (OU's)** are containers within the AD domain with a hierarchical structure.

Active Directory **Objects** can be a single user or a group, or a hardware component, such as a computer or printer. Each domain holds a database that contains object identity information that creates an AD environment, including:

* Users - A security principal that is allowed to authenticate to machines in the domain
* Computers - A special type of user accounts
* GPOs - Collections of policies that are applied to other AD objects

**AD domains** are a collection of Microsoft components within an AD network.&#x20;

**AD Forest** is a collection of domains that trust each other.&#x20;

<figure><img src="../../../../.gitbook/assets/image (378).png" alt=""><figcaption></figcaption></figure>

`systeminfo` provides information about the machine, including the operating system name and version, hostname, and other hardware information as well as the AD domain.

```
C:\Users\kkidd>systeminfo | findstr Domain
OS Configuration:          Primary Domain Controller
Domain:                    thmredteam.com
```

### Active Directory (AD) Enum

The following PowerShell command is to get all active directory user accounts. Note that we need to use -Filter argument.

```
PS C:\Users\thm> Get-ADUser  -Filter *
DistinguishedName : CN=Administrator,CN=Users,DC=thmredteam,DC=com
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 4094d220-fb71-4de1-b5b2-ba18f6583c65
SamAccountName    : Administrator
SID               : S-1-5-21-1966530601-3185510712-10604624-500
Surname           :
UserPrincipalName :
```

The Distinguished Name (DN) is a collection of comma-separated key and value pairs used to identify unique records within the directory. The DN consists of Domain Component (DC), OrganizationalUnitName (OU), Common Name (CN), and others. The following "CN=User1,CN=Users,DC=thmredteam,DC=com" is an example of DN, which can be visualized as follow:

<figure><img src="../../../../.gitbook/assets/image (380).png" alt=""><figcaption></figcaption></figure>

Using the SearchBase option, we specify a specific Common-Name CN in the active directory. For example, we can specify to list any user(s) that part of Users.

```
PS C:\Users\thm> Get-ADUser -Filter * -SearchBase "CN=Users,DC=THMREDTEAM,DC=COM"


DistinguishedName : CN=Administrator,CN=Users,DC=thmredteam,DC=com
Enabled           : True
GivenName         :
Name              : Administrator
ObjectClass       : user
ObjectGUID        : 4094d220-fb71-4de1-b5b2-ba18f6583c65
SamAccountName    : Administrator
SID               : S-1-5-21-1966530601-3185510712-10604624-500
Surname           :
UserPrincipalName :
```

Note that the result may contain more than one user depending on the configuration of the CN. Try the command to find all users within the THM OU and answer question 1 below.

## Host Security Solution

### Antivirus Software (AV)

```
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

, est une commande PowerShell utilisée pour interagir avec le service WMI (Windows Management Instrumentation) afin d'obtenir des informations sur les produits antivirus installés sur un système Windows.

```
PS C:\Users\thm> Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct


displayName              : Bitdefender Antivirus
instanceGuid             : {BAF124F4-FA00-8560-3FDE-6C380446AEFB}
pathToSignedProductExe   : C:\Program Files\Bitdefender\Bitdefender Security\wscfix.exe
pathToSignedReportingExe : C:\Program Files\Bitdefender\Bitdefender Security\bdservicehost.exe
productState             : 266240
timestamp                : Wed, 15 Dec 2021 12:40:10 GMT
PSComputerName           :

displayName              : Windows Defender
instanceGuid             : {D58FFC3A-813B-4fae-9E44-DA132C9FAA36}
pathToSignedProductExe   : windowsdefender://
pathToSignedReportingExe : %ProgramFiles%\Windows Defender\MsMpeng.exe
productState             : 393472
timestamp                : Fri, 15 Oct 2021 22:32:01 GMT
PSComputerName           :
```

There is a third-party antivirus (Bitdefender Antivirus) and Windows Defender installed on the computer.

Active mode is used where the MS Defender runs as the primary antivirus software on the machine where provides protection and remediation. Passive mode is run when a 3rd party antivirus software is installed. Therefore, it works as secondary antivirus software where it scans files and detects threats but does not provide remediation. Finally, Disable mode is when the MS Defender is disabled or uninstalled from the system.

```
PS C:\Users\kkidd> Get-Service WinDefend

Status   Name               DisplayName
------   ----               -----------
Running  WinDefend          Windows Defender Antivirus Service
```

### Host-based Firewall

```
PS C:\Users\thm> Get-NetFirewallProfile | Format-Table Name, Enabled

Name    Enabled
----    -------
Domain     True
Private    True
Public     True
```

If we have admin privileges on the current user we logged in with, then we try to disable one or more than one firewall profile using the Set-NetFirewallProfile cmdlet.

```
PS C:\Windows\system32> Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled False
PS C:\Windows\system32> Get-NetFirewallProfile | Format-Table Name, Enabled
---- -------
Domain False
Private False
Public False
```

### Security Event Logging and Monitoring

We can get a list of available event logs on the local machine using the Get-EventLog cmdlet.

```
PS C:\Users\thm> Get-EventLog -List

  Max(K) Retain OverflowAction        Entries Log
  ------ ------ --------------        ------- ---
     512      7 OverwriteOlder             59 Active Directory Web Services
  20,480      0 OverwriteAsNeeded         512 Application
     512      0 OverwriteAsNeeded         170 Directory Service
 102,400      0 OverwriteAsNeeded          67 DNS Server
  20,480      0 OverwriteAsNeeded       4,345 System
  15,360      0 OverwriteAsNeeded       1,692 Windows PowerShell
```

Sometimes, the list of available event logs gives you an insight into what applications and services are installed on the machine

### Endpoint Detection and Response (EDR)

The EDR is a cybersecurity solution that defends against malware and other threats. EDRs can look for malicious files, monitor endpoint, system, and network events, and record them in a database for further analysis, detection, and investigation. EDRs are the next generation of antivirus and detect malicious activities on the host in real-time.

### Installed Applications

First, we start enumerating the system for installed applications by checking the application's name and version. As a red teamer, this information will benefit us. We may find vulnerable software installed to exploit and escalate our system privileges. Also, we may find some information, such as plain-text credentials,

```
PS C:\Users\thm> wmic product get name,version
Name                                                            Version
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.28.29910     14.28.29910
AWS Tools for Windows                                           3.15.1248
Amazon SSM Agent                                                3.0.529.0
aws-cfn-bootstrap                                               2.0.5
AWS PV Drivers                                                  8.3.4
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29910  14.28.29910
```
