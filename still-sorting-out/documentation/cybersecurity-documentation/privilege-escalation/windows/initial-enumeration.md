# 📻 Initial Enumeration

### System Enumeration <a href="#lecture_heading" id="lecture_heading"></a>

once you get a shell, you can `systeminfo` to learn more about your target:

<figure><img src="../../../../../.gitbook/assets/image (341).png" alt=""><figcaption></figcaption></figure>

If we want to get only certain fields, we can grep out the information like this:

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

You can also do this to find computer ID or hostname:

<figure><img src="../../../../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

To see the patches your computer had recently:

```
wmic qfe list
```

This command will display a list of installed hotfixes on the system, including their names, description, installed by, and other relevant information. You can also use additional parameters with the `wmic qfe` command to filter the results or format the output as needed.

Now we got

```
wmic logicaldisk get caption,description,providername
```

This command will display information about each logical disk on the system, including its caption (drive letter), description, and provider name (applicable to network drives). The output will be a table with columns for each of these properties, showing details for each logical disk present on the system.

### User Enumeration <a href="#lecture_heading" id="lecture_heading"></a>

In a Windows privilege escalation (privesc) context, these commands are often used to gather information about the current user's privileges and group memberships, as well as to explore the users and groups on the system. Let's break down each command:

1. **`whoami`**
   * **Purpose:** Displays the username, domain, and groups of the currently logged-in user.
   *   **Usage Example:**

       ```bash
       whoami
       ```
2. **`whoami /priv`**
   * **Purpose:** Shows the privileges held by the current user. It displays information about the user's security privileges, which may include elevated privileges.
   *   **Usage Example:**

       ```bash
       whoami /priv
       ```
3. **`whoami /groups`**
   * **Purpose:** Lists the group memberships of the current user.
   *   **Usage Example:**

       ```bash
       whoami /groups
       ```
4. **`net user`**
   * **Purpose:** Displays a list of user accounts on the system.
   *   **Usage Example:**

       ```bash
       net user
       ```
5. **`net user username`**
   * **Purpose:** Shows detailed information about a specific user account, including group memberships and account status.
   *   **Usage Example:**

       ```bash
       net user john_doe
       ```
6. **`net user administrator`**
   * **Purpose:** Displays detailed information about the "Administrator" user account.
   *   **Usage Example:**

       ```bash
       net user administrator
       ```
7. **`net localgroup`**
   * **Purpose:** Lists local groups on the system.
   *   **Usage Example:**

       ```bash
       net localgroup
       ```
8. **`net localgroup administrators`**
   * **Purpose:** Displays members of the "Administrators" local group. This can reveal who has administrative privileges on the system.
   *   **Usage Example:**

       ```bash
       net localgroup administrators
       ```

### Network Enumeration <a href="#lecture_heading" id="lecture_heading"></a>

In a network enumeration and Windows privilege escalation context, these commands are used to gather information about the network configuration, active connections, and associated processes on a Windows system. Here's an explanation of each command:

1. **`ipconfig`**
   * **Purpose:** Displays the IP configuration information for all network interfaces on the system.
   *   **Usage Example:**

       ```bash
       ipconfig
       ```
   * **Relevance for Privesc:** The output provides information about the IP addresses, subnet masks, default gateways, and DNS servers. It can be useful to understand the network environment, identify potential targets, and find network-related vulnerabilities.
2. **`ipconfig /all`**
   * **Purpose:** Provides detailed information about the IP configuration, including additional details like DHCP lease information, DNS server addresses, and more.
   *   **Usage Example:**

       ```bash
       ipconfig /all
       ```
   * **Relevance for Privesc:** Offers a more comprehensive view of the network configuration, which can be valuable for understanding the network infrastructure and identifying potential points of entry.
3. **`arp -a`**
   * **Purpose:** Displays the ARP (Address Resolution Protocol) cache, showing the mapping between IP addresses and corresponding physical MAC addresses.
   *   **Usage Example:**

       ```bash
       arp -a
       ```
   * **Relevance for Privesc:** Helps identify devices on the local network and may reveal previously accessed devices, aiding in reconnaissance and potential lateral movement.
4. **`route print`**
   * **Purpose:** Prints the IP routing table, which shows the routing information used by the operating system to determine how to reach different networks.
   *   **Usage Example:**

       ```bash
       route print
       ```
   * **Relevance for Privesc:** Understanding the routing table can provide insights into how network traffic is directed, potentially revealing additional paths or points of interest for an attacker.
5. **`netstat -ano`**
   * **Purpose:** Displays active network connections, listening ports, and associated process IDs.
   *   **Usage Example:**

       ```bash
       netstat -ano
       ```
   * **Relevance for Privesc:** Identifies active connections, open ports, and associated processes. This information can be crucial for identifying services, potential vulnerabilities, or unauthorized processes running on the system.

### Password Hunting <a href="#lecture_heading" id="lecture_heading"></a>

1. **`findstr /si password *.txt`**
   * **Purpose:** Searches for the string "password" case-insensitively (`/i`) in all text files (`*.txt`) in the current directory and its subdirectories (`/s`).
   * **Relevance:** This command is useful for locating text files that may contain instances of the word "password."
2. **`cd C:\ & findstr /SI /M "password" *.xml *.ini *.txt`**
   * **Purpose:** Changes the current directory to the root of the C: drive (`cd C:\`) and then searches for the string "password" case-insensitively (`/SI`) in XML, INI, and TXT files (`*.xml *.ini *.txt`) in the entire system (`/M` for machine-readable output).
   * **Relevance:** It searches for the specified string in various types of configuration files.
3. **`findstr /si password *.xml *.ini *.txt *.config 2>nul >> results.txt`**
   * **Purpose:** Searches for the string "password" case-insensitively (`/si`) in XML, INI, TXT, and CONFIG files (`*.xml *.ini *.txt *.config`). The results are redirected to a file named "results.txt" (`>> results.txt`), and any error messages are redirected to `nul` (`2>nul`).
   * **Relevance:** This command collects occurrences of "password" in various file types and appends the results to a text file.
4. **`findstr /spin "password" *.*`**
   * **Purpose:** Searches for the string "password" in all types of files (`*.*`) in the current directory and its subdirectories (`/s`). The `/p` option suppresses file names in the output.
   * **Relevance:** This command is similar to the first one but includes searching in non-text files.

{% embed url="https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/windows-privilege-escalation/#default-writeable-folders" %}

<figure><img src="../../../../../.gitbook/assets/image (339).png" alt=""><figcaption></figcaption></figure>

### AV Enumeration <a href="#lecture_heading" id="lecture_heading"></a>

Anti-Virus (AV) enumeration and firewall configuration on a Windows system. Let's break down each command:

1. **`sc query windefend`**
   * **Purpose:** Queries information about the Windows Defender service.
   * **Relevance:** This command provides details about the status of the Windows Defender service, including its state (running, stopped) and other information.
2. **`sc queryex type= service`**
   * **Purpose:** Queries information about all services on the system.
   * **Relevance:** This command lists information about all services installed on the system. It can be used to identify the state and configuration of various services, including those related to antivirus.
3. **`netsh advfirewall dump`**
   * **Purpose:** Dumps the configuration of the Windows Firewall.
   * **Relevance:** This command provides a detailed configuration dump of the Windows Firewall settings, including rules, groups, and other firewall-related configurations.
4. **`netsh firewall show state`**
   * **Purpose:** Displays the operational state of the Windows Firewall.
   * **Relevance:** Shows whether the Windows Firewall is enabled or disabled, along with other operational details.
5. **`netsh firewall show config`**
   * **Purpose:** Displays detailed configuration information for the Windows Firewall.
   * **Relevance:** This command shows the specific configuration settings of the Windows Firewall, including allowed and blocked ports, settings for different profiles (domain, private, public), etc.

### Automated Tool <a href="#lecture_heading" id="lecture_heading"></a>

Here's a quick explanation for each of the tools and resources you mentioned:

1. **WinPEAS:**
   * **Description:** WinPEAS (Windows Privilege Escalation Awesome Scripts Suite) is a collection of scripts that automate the process of Windows privilege escalation. It checks for common misconfigurations and vulnerabilities that can be exploited for privilege escalation.
   * **GitHub Repository:** [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS)
2. **Windows PrivEsc Checklist:**
   * **Description:** The Windows Privilege Escalation Checklist is a comprehensive guide that provides a checklist of common techniques and misconfigurations used for Windows privilege escalation. It covers various aspects of the Windows operating system.
   * **Online Resource:** [Windows PrivEsc Checklist](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation)
3. **Sherlock:**
   * **Description:** Sherlock is a PowerShell script that checks a Windows system for the presence of known privilege escalation vulnerabilities, including missing patches and misconfigurations.
   * **GitHub Repository:** [Sherlock](https://github.com/rasta-mouse/Sherlock)
4. **Watson:**
   * **Description:** Watson is another PowerShell script designed for Windows privilege escalation. It focuses on finding missing patches and misconfigurations that can be exploited.
   * **GitHub Repository:** [Watson](https://github.com/rasta-mouse/Watson)
5. **PowerUp:**
   * **Description:** PowerUp is a PowerShell script from the PowerSploit project. It includes a variety of functions to assist in Windows privilege escalation, including finding misconfigurations and potential vulnerabilities.
   * **GitHub Repository:** [PowerUp](https://github.com/PowerShellMafia/PowerSploit/tree/master/Privesc)
6. **JAWS:**
   * **Description:** JAWS (Just Another Windows (Enum) Script) is a PowerShell script designed to automate the enumeration of Windows systems. It checks for common misconfigurations and potential privilege escalation vectors.
   * **GitHub Repository:** [JAWS](https://github.com/411Hall/JAWS)
7. **Windows Exploit Suggester:**
   * **Description:** The Windows Exploit Suggester is a tool that compares the patch level of a system with the Microsoft vulnerability database. It suggests potential exploits for missing patches.
   * **GitHub Repository:** [Windows Exploit Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
8. **Metasploit Local Exploit Suggester:**
   * **Description:** This Metasploit module is used to suggest local exploits based on the information collected from a compromised system. It can help identify potential privilege escalation paths.
   * **Metasploit Blog Post:** [Metasploit Local Exploit Suggester](https://blog.rapid7.com/2015/08/11/metasploit-local-exploit-suggester-do-less-get-more/)
9. **Seatbelt:**
   * **Description:** Seatbelt is a collection of PowerShell scripts that perform various security-oriented enumeration tasks on Windows systems. It covers a wide range of information, including potential privilege escalation vectors.
   * **GitHub Repository:** [Seatbelt](https://github.com/GhostPack/Seatbelt)
10. **SharpUp:**
    * **Description:** SharpUp is another tool from the GhostPack project. It is designed to assist in Windows privilege escalation by identifying potential opportunities based on system configuration.
    * **GitHub Repository:** [SharpUp](https://github.com/GhostPack/SharpUp)

{% embed url="https://book.hacktricks.xyz/windows-hardening/checklist-windows-privilege-escalation" %}

<figure><img src="../../../../../.gitbook/assets/image (340).png" alt=""><figcaption></figcaption></figure>

install pip one liner:

```
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py; python get-pip.py
```
