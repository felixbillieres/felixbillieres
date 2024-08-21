# 💃 Pivoting, Tunneling, and Port Forwarding

If a host has more than one network adapter, we can likely use it to move to a different network segment. Pivoting is essentially the idea of `moving to other networks through a compromised host to find more targets on different network segments`.

Pivoting's primary use is to defeat segmentation (both physically and virtually) to access an isolated network.

Whether assigned `dynamically` or `statically`, the IP address is assigned to a `Network Interface Controller` (`NIC`). A computer can have multiple NICs (physical and virtual), meaning it can have multiple IP addresses assigned, allowing it to communicate on various networks. It's important to check for additional NICs using commands like `ifconfig` (in macOS and Linux) and `ipconfig` (in Windows).
