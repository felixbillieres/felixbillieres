# 🥐 Simple Port Scanner

## Building a Simple Port Scanner in Python 🕵️‍♂️🔍

Let's delve into the world of cybersecurity by creating a basic but effective port scanner using Python. This script will help you identify open ports on a target machine. Ready? Let's begin!

```python
#!/bin/python3

import sys
import socket
from datetime import datetime

# Define our target
if len(sys.argv) == 2:
    target = socket.gethostbyname(sys.argv[1])  # Translate hostname to IPv4
else:
    print("Invalid amount of arguments.")
    print("Syntax: python3 scanner.py <target>")

# Add a pretty banner
print("-" * 50)
print("Scanning target " + target)
print("Time started: " + str(datetime.now()))
print("-" * 50)

try:
    for port in range(50, 85):
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        socket.setdefaulttimeout(1)
        result = s.connect_ex((target, port))  # returns an error indicator - if port is open it throws a 0, otherwise 1
        if result == 0:
            print("Port {} is open".format(port))
        s.close()

except KeyboardInterrupt:
    print("\nExiting program.")
    sys.exit()

except socket.gaierror:
    print("Hostname could not be resolved.")
    sys.exit()

except socket.error:
    print("Could not connect to the server.")
    sys.exit()
```

### Explanation:

1. **Imports:**
   * `sys`: Provides access to some variables used or maintained by the Python interpreter and to functions that interact with the interpreter.
   * `socket`: Provides low-level networking interface.
   * `datetime`: Supplies classes for working with dates and times.
2. **Target Definition:**
   * The target is either set as the IPv4 address of the provided hostname or as a command-line argument.
3. **Banner:**
   * A simple banner is displayed, indicating the target and the time the scan started.
4. **Port Scanning:**
   * The script attempts to connect to ports in the range from 50 to 84.
   * If the connection is successful (result == 0), the script prints that the port is open.
5. **Exception Handling:**
   * Handles potential exceptions like keyboard interrupt, hostname resolution failure, and socket errors.

Feel free to explore and modify this script as you continue your journey into cybersecurity.
