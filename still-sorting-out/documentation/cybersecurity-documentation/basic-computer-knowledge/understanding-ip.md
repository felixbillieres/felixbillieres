# 0️⃣ Understanding IP

## Understanding IP Addresses and ifconfig Command

### Introduction:

IP addresses are crucial identifiers in computer networks, enabling communication between devices. The `ifconfig` command provides a snapshot of network configuration on a Unix-like system.

#### Executing `ifconfig`:

```bash
ifconfig
```

This command reveals detailed information about network interfaces, including IP addresses, MAC addresses, and more.

<figure><img src="https://www.wikihow.com/images_en/thumb/c/c8/Check-the-IP-Address-in-Linux-Step-6.jpg/v4-460px-Check-the-IP-Address-in-Linux-Step-6.jpg" alt=""><figcaption><p>The ipv4 of the machine you're on</p></figcaption></figure>

### IP Address Types:

#### IPv4 (inet):

* **IPv4 addresses** are the traditional numerical labels assigned to devices on a network.
* In the `ifconfig` output, IPv4 addresses are denoted by the keyword **inet**.

#### IPv6 (inet6):

* **IPv6 addresses** are a more modern and expansive system to accommodate the increasing number of devices.
* In the `ifconfig` output, IPv6 addresses are denoted by the keyword **inet6**.

<figure><img src="https://linuxteaching.com/storage/img/images_4/ifconfig_disable_ipv6.png" alt=""><figcaption></figcaption></figure>

### IP Address Classes:

IP addresses are categorized into classes, traditionally denoted as A, B, and C classes. These classes represent different ranges of IP addresses.

#### A, B, C Classes:

1. **Class A (0.0.0.0 to 126.255.255.255):**
   * Suitable for large networks.
   * First octet represents the network, and the remaining three octets represent hosts.
2. **Class B (128.0.0.0 to 191.255.255.255):**
   * Balances between network and host capacity.
   * First two octets represent the network, and the remaining two octets represent hosts.
3. **Class C (192.0.0.0 to 223.255.255.255):**
   * Suitable for smaller networks.
   * First three octets represent the network, and the last octet represents hosts.

### Additional Insights:

* **Network Mask:** The subnet mask specifies the network portion of an IP address.
* **MAC Address:** The hardware address uniquely identifying a network interface.
* **RX, TX:** Indicate received and transmitted data, respectively.

### Practical Use:

Understanding IP addresses is essential for network administrators, security professionals, and enthusiasts. Use `ifconfig` to inspect network configurations, diagnose issues, and gain insights into your system's connectivity.

For more in-depth exploration of IP addresses and networking concepts, consider referring to official documentation or online tutorials.
