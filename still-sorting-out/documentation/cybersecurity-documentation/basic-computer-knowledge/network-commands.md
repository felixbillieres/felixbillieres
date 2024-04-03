# ⚡ Network Commands

#### Network Commands Cheat Sheet:

1. **`ip a`:**
   * _Explanation:_ Displays the network interfaces and their associated IP addresses.
   * _Example:_ `ip a` would show information about network interfaces, including their IP addresses, MAC addresses, and other details.
2. **`ifconfig`:**
   * _Explanation:_ Displays the configuration and status of network interfaces.
   * _Example:_ Running `ifconfig` shows configuration details, including IP addresses, MAC addresses, and other information for active network interfaces.
3. **`iwconfig`:**
   * _Explanation:_ Displays the configuration and status of wireless network interfaces.
   * _Example:_ Running `iwconfig` shows configuration details like wireless signal strength, frequency, and encryption information for active wireless interfaces.
4. **`ip n`:**
   * _Explanation:_ Displays the Neighbor Table, containing IP-to-MAC address mappings for devices in the local network.
   * _Example:_ Running `ip n` shows IP and MAC addresses of devices that recently communicated with the current device.
5.  **`arp -a`:**

    * _Explanation:_ Displays the ARP (Address Resolution Protocol) cache, mapping IP addresses to MAC addresses.
    * _Example:_ Running `arp -a` shows IP and MAC addresses of devices recently resolved by the ARP protocol.

    <figure><img src="https://media.geeksforgeeks.org/wp-content/uploads/20190307221912/arp_deleting_host.png" alt=""><figcaption></figcaption></figure>
6. **`ip r`:**
   * _Explanation:_ Displays the routing table, containing information about network routes.
   * _Example:_ Running `ip r` shows the routing table, including destination networks, gateway IP addresses, and network interfaces.
7. **`route`:**
   * _Explanation:_ Displays or manipulates the IP routing table.
   * _Example:_ Running `route` displays the routing table, similar to the `ip r` command.
8. **`netstat`:**
   * _Explanation:_ Displays network connections, routing tables, interface statistics, masquerade connections, and multicast memberships.
   * _Example:_ Running `netstat -an` shows all active network connections and listening ports.
9. **`ss`:**
   * _Explanation:_ A command-line tool that provides information about sockets and socket statistics.
   * _Example:_ `ss -t` displays all TCP sockets.
10. **`ping`:**
    * _Explanation:_ Tests network connectivity to a specific IP address or domain.
    * _Example:_ `ping google.com` checks the connectivity to Google's servers.
11. **`traceroute` or `traceroute6`:**
    * _Explanation:_ Shows the route that packets take to reach a destination, indicating the number of hops.
    * _Example:_ `traceroute google.com` traces the route to Google's servers.
12. **`nslookup` or `dig`:**
    * _Explanation:_ Performs DNS queries to obtain domain name or IP address information.
    * _Example:_ `nslookup example.com` retrieves DNS information for the domain.
13. **`tcpdump`:**
    * _Explanation:_ A packet analyzer that allows real-time packet capturing and display.
    * _Example:_ `tcpdump -i eth0` captures and displays packets on the eth0 interface.
14. **`nmap`:**
    * _Explanation:_ A powerful network scanning tool used for discovering hosts and services on a computer network.
    * _Example:_ `nmap -p 1-1000 targetIP` scans the first 1000 ports on the specified target IP.
15. **`telnet`:**
    * _Explanation:_ A network protocol used to establish a connection to a remote server or device.
    * _Example:_ `telnet example.com 80` connects to port 80 on example.com using the telnet protocol.
16. **`wget` or `curl`:**
    * _Explanation:_ Downloads files from the internet using HTTP, HTTPS, or FTP protocols.
    * _Example:_ `wget http://example.com/file.txt` downloads a file from the specified URL.
17. **`iptraf` or `iftop`:**
    * _Explanation:_ Real-time console-based network bandwidth monitoring tools.
    * _Example:_ `iftop` displays a list of network connections and bandwidth usage.
18. **`lsof`:**
    * _Explanation:_ Lists open files, including network connections.
    * _Example:_ `lsof -i` shows all network connections.

\
\
