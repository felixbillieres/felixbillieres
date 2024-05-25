# 😷 LLMNR Poisoning

## 🌐 **What is LLMNR Poisoning?**

LLMNR Poisoning is a cyber attack that exploits vulnerabilities in the LLMNR (Link-Local Multicast Name Resolution) protocol. LLMNR is used in networks to resolve the names of neighboring devices when traditional DNS (Domain Name System) fails. LLMNR Poisoning occurs when an attacker manipulates or poisons LLMNR responses to redirect network traffic and capture sensitive information.

<figure><img src="https://www.aptive.co.uk/wp-content/uploads/2017/02/responder-llmnr-netbios-name-server-spoofing.png" alt=""><figcaption></figcaption></figure>

## 🤔 **How Does LLMNR Poisoning Work?**

1. **Name Resolution Failures:**
   * When a device in a network tries to communicate with another device by name, it may use LLMNR if traditional DNS fails to resolve the name.
2. **Attacker's Deceptive Response:**
   * The attacker, leveraging a Man-in-the-Middle (MITM) position, intercepts LLMNR queries. Instead of letting the legitimate device respond, the attacker sends deceptive responses.
3. **Redirecting Traffic:**
   * The deceptive response points the requesting device to the attacker's machine, pretending to be the intended target. This redirection enables the attacker to capture sensitive information exchanged during the communication.
4. **Credential Capture:**
   * As the communication is now passing through the attacker's machine, they can capture sensitive data, such as usernames and password hashes. This is particularly effective in scenarios where devices are attempting to authenticate with each other.

## 🚦 **Common Tools Used:**

* **Responder:**
  * A popular tool for LLMNR and NBT-NS poisoning. It listens for LLMNR/NBT-NS requests and responds with deceptive answers.
* **Mimikatz:**
  * While not exclusively for LLMNR Poisoning, Mimikatz is often used to extract credentials obtained through these attacks.

## Simulation:

**LLMNR Poisoning Simulation with Responder**

LLMNR (Link-Local Multicast Name Resolution) poisoning is a network attack that exploits the LLMNR protocol to capture hashed credentials. Below is a step-by-step guide on simulating this attack using Responder on Kali Linux.

#### Step 1: Launch Responder on Kali Linux

Open a terminal on your Kali machine and run Responder with the following command:

```bash
sudo responder -I eth0 -dwPv
```

* `-I eth0`: Specifies the network interface (replace with your actual interface, e.g., eth0).
* `-dwPv`: Enables various options:
  * `-d`: Enables HTTP and SMB capture.
  * `-w`: Writes captured hashes to an NTLMv2 format.
  * `-P`: Prepares Poisoner to capture plaintext credentials.
  * `-v`: Enables verbose mode for detailed output.

#### Step 2: Identify Kali's IP Address

Check the IP address assigned to your Kali machine on the specified network interface (eth0). This IP address will be used by the Windows machine to simulate the event.

#### Step 3: Simulate Event on Windows

On your Windows machine:

1. Open File Explorer.
2.  In the path bar, type the following and press Enter:

    ```plaintext
    \\ip_of_kali
    ```

    Replace `ip_of_kali` with the actual IP address of your Kali machine.7

<figure><img src="../../../../.gitbook/assets/Capture d’écran du 2023-12-16 17-48-50.png" alt=""><figcaption></figcaption></figure>

#### Step 4: Observe Responder Output

Go back to the terminal where Responder is running on Kali. You should see captured hashes in the output. Responder captures NTLMv2-SSP hashes, which are commonly used for Windows authentication.

<figure><img src="../../../../.gitbook/assets/Screenshot 2023-12-16 at 17-52-22 LLMNR Poisoning - Félix Billières.png" alt=""><figcaption><p>hashes captured succesfully</p></figcaption></figure>

#### Understanding the Commands and Processes:

* **Responder Command Explanation:**
  * `sudo responder -I eth0 -dwPv`: Launches Responder with specific options for capturing hashes.
    * `-I eth0`: Specifies the network interface.
    * `-dwPv`: Enables various options for capturing HTTP, SMB, plaintext, and enabling verbose mode.
* **Windows Event Simulation:**
  * Navigating to `\\ip_of_kali` simulates a network event where the Windows machine attempts to connect to a network resource using the specified IP address.
* **Responder Output:**
  * Responder captures hashed credentials exchanged during the simulated event. NTLMv2-SSP hashes are commonly used for Windows authentication

## Disable LLMNR:

[https://www.blackhillsinfosec.com/how-to-disable-llmnr-why-you-want-to/](https://www.blackhillsinfosec.com/how-to-disable-llmnr-why-you-want-to/)

