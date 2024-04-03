# ☝️ Initial Enumeration

## Understanding Your Target

During a penetration test or security audit, gaining a solid understanding of the target system is crucial. Initial enumeration involves collecting valuable information about the system, users, network, and potential vulnerabilities. Here are some beginner-friendly commands to assist you in this phase:

### System Information

#### 1. `uname -a`

* Displays system information, including the kernel version.

#### 2. `hostname`

* Reveals the host or machine name.

#### 3. `cat /proc/version`

* Shows detailed kernel and version information.

#### 4. `cat /etc/issue`

* Provides distribution-specific information.

#### 5. `lscpu`

* Displays CPU information.

#### 6. `ps aux`

* Lists all running processes, providing insights into active applications.

#### `7. ss -lntp`

* list listening network sockets along with their associated processes on a Linux system.



### User Enumeration

#### 7. `whoami`

* Identifies the current user.

#### 8. `id`

* Provides information about the user and group.

#### 9. `sudo -l`

* Lists the commands the current user can execute with sudo privileges.

#### 10. `cat /etc/passwd`

* Displays information about user accounts.

#### 11. `cat /etc/shadow`

* Shows encrypted password information.

#### 12. `cat /etc/group`

* Lists information about groups.

#### 13. `history`

* Displays command history, revealing recent activities.

**Mindset Tip:** Understand who you are on the system, the system's architecture, your sudo privileges, and review command history for insights.

### Network Enumeration

#### 14. `ip a`

* Shows information about network interfaces.

#### 15. `ip route`

* Displays the routing table.

#### 16. `ip neigh`

* Lists ARP cache entries, revealing connected devices.

#### 17. `netstat -ano`

* Provides network statistics and active connections.

### Password Hunting

#### 18. `grep --color=auto -rnw '/' -ie "PASSWORD=" --color=always 2> /dev/null`

* Searches for sensitive information like passwords.

#### 19. `locate password | more`

* Locates files containing the term "password."

#### 20. `locate pass | more`

* Locates files containing the term "pass."

#### 21. `find / -name authorized_keys`

* Searches for SSH authorized keys.

#### 22. `find / -name id_rsa > /dev/null`

* Searches for private SSH keys.

Remember, initial enumeration sets the stage for deeper analysis and exploitation. Use these commands responsibly and ethically in alignment with your testing goals. Happy hunting! 🕵️‍♂️🔍
