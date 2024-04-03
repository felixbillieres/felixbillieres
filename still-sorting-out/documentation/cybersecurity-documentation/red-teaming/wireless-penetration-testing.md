# 🐡 Wireless Penetration Testing

Wireless Penetration Testing, also known as wireless pentesting, is the systematic examination of the security of a wireless network to identify vulnerabilities and potential risks. This process involves evaluating the strength of security mechanisms, reviewing nearby networks, assessing guest networks, and checking network access controls.

<figure><img src="../../../../.gitbook/assets/image (386).png" alt="" width="375"><figcaption></figcaption></figure>

### Types of Wireless Networks

1. **WPA PSK (Pre-Shared Key):**
   * Basic security commonly used in home environments.
   * Involves a shared passphrase or key between the user and the network.
2. **WPA2 Enterprise:**
   * More advanced security commonly found in small businesses.
   * Utilizes a centralized authentication server, such as RADIUS (Remote Authentication Dial-In User Service).

### Activities Performed in Wireless Pentesting

1. **Evaluating Strength of PSK:**
   * Analyzing the complexity and robustness of the Pre-Shared Key.
2. **Reviewing Nearby Networks:**
   * Identifying neighboring wireless networks and their security configurations.
3. **Assessing Guest Networks:**
   * Examining the security of networks designated for guest access.
4. **Checking Network Access:**
   * Verifying the effectiveness of access controls and permissions.

### Tools Used in Wireless Pentesting

1. **Wireless Card:**
   * Hardware component allowing the computer to communicate with wireless networks.
2. **Router:**
   * Networking device responsible for transmitting data between the local network and the internet.
3. **Laptop:**
   * Portable computer used for running pentesting tools and analyzing results.

### The Hacking Process (WPA2 PSK)

1. **Place:**
   * Physically position yourself within the range of the target wireless network.
2. **Discover:**
   * Identify available wireless networks in the vicinity.
3. **Select:**
   * Choose the target network based on the assessment criteria.
4. **Perform:**
   * Execute penetration testing tools and techniques to exploit vulnerabilities.
5. **Capture:**
   * Collect data packets transmitted over the network for analysis.
6. **Attempt:**
   * Make various attempts to crack the WPA2 PSK, utilizing methods such as brute force attacks or dictionary attacks.

## **Cracking WPA2 PSK** Practical case:

### **Wireless Pentesting Setup and Network Identification**

Before diving into the wireless penetration testing process, ensure your wireless card is connected to your Kali VM. Follow these professional steps to set up and initiate the process:

#### Step 1: Connect Your Wireless Card to Kali VM

Connect your wireless card to the Kali VM to enable communication with wireless networks.

#### Step 2: Enable Monitor Mode

Open a terminal and execute the following command to place your wireless card in monitor mode, allowing you to monitor all incoming traffic and capture a handshake:

<figure><img src="../../../../.gitbook/assets/image (387).png" alt=""><figcaption></figcaption></figure>

Next, use the `airmon-ng` tool to check and terminate interfering processes:

```
airmon-ng check kill
```

it checks any process that's running and kills it if it interferes with what we are going to do

Initiate monitor mode with the following command:

```
airmon-ng start wlan0
```

<figure><img src="../../../../.gitbook/assets/image (388).png" alt=""><figcaption></figcaption></figure>

Verify that monitor mode is active by running:

```bash
iwconfig
```

Now, you should observe `wlan0mon` indicating that your wireless card is in monitor mode.

Now that the setup is complete, it's time to identify the target network. If you have a client, gather the SSID to define the scope. Alternatively, scan the networks around you using:

```
airodump-ng wlan0mon
```

<figure><img src="../../../../.gitbook/assets/image (389).png" alt=""><figcaption></figcaption></figure>

Explanation of the displayed information:

* **BSSID:** MAC address of the access point.
* **PWR:** Signal strength.
* **Beacons:** Number of beacons sent by the access point.
* **CH:** Channel on which the network operates.
* **MB:** Maximum speed supported by the access point.
* **ENC:** Encryption type.
* **CIPHER:** Cipher used for encryption.
* **AUTH:** Authentication method.
* **ESSID:** Extended Service Set Identifier (SSID), the name of the network.

Once you've identified the target network, narrow down the information you need by executing:

```
airodump-ng -c 11 --bssid EO:22:04:C4:95:2A -w capture wlan0mon
```

Explanation of variables:

* **-c 11:** Specifies the channel of the target network.
* **--bssid EO:22:04:C4:95:2A:** Specifies the BSSID (MAC address) of the target network.
* **-w capture:** Names the output file for capturing data.

The -c and --bssid are those variables:

<figure><img src="../../../../.gitbook/assets/image (390).png" alt=""><figcaption></figcaption></figure>

Upon executing the specified command, transition into listener mode, attentively awaiting a critical parameter to materialize: the notification indicating "handshake captured." This parameter serves as confirmation that the WPA handshake has been successfully obtained, a pivotal step in the wireless penetration testing process.

However, an alternative and more proactive approach exists for capturing the WPA handshake. This involves inducing the target to perform a handshake by initiating a deauthentication process. By strategically deauthenticating the target, it is compelled to re-establish a connection, subsequently generating the coveted WPA handshake.

Open another terminal while listener is on

```
aireplay-ng -0 1 -a [mac adress of target] -c [Station of target] wlan0mon
```

* `-0 1`: Send a single deauthentication packet to the target, prompting it to re-authenticate and generate a WPA handshake.
* `-a [mac address]`: Specify the MAC address of the target access point.
* `-c [station address]`: Specify the MAC address of a connected client (station).
* `wlan0mon`: Replace this with the name of your wireless interface in monitor mode.

<figure><img src="../../../../.gitbook/assets/image (391).png" alt=""><figcaption><p>sending death packets to a target</p></figcaption></figure>

Monitor your listener for the WPA handshake. This might take a few attempts, as it depends on the target network's activity. Once successful, you'll see a notification indicating the captured handshake.

Now, you can attempt to crack the captured WPA handshake using aircrack-ng and a wordlist. Run the following command:

```
aircrack-ng -w wordlist.txt -b [mac adress of target] [capturefile.cap]
```

* `-w wordlist.txt`: Specify the path to your wordlist file containing potential passwords.
* `-b [mac address]`: Provide the MAC address of the target access point.
* `[capturefile.cap]`: Replace this with the name of the file where you saved the captured WPA handshake.

And if the word list is successful, you will see a "current passphrase _: password\_of\_target"_
