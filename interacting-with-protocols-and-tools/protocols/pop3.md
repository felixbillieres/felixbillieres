# 3️⃣ POP3

#### POP3 (Post Office Protocol 3) Enumeration and Interaction

Post Office Protocol 3 (POP3) is a widely used protocol for email retrieval. Enumeration and interaction with POP3 servers are essential steps in assessing email security. Here's a guide on POP3 enumeration and interaction:

<figure><img src="https://lh3.googleusercontent.com/AyHpAH4P6A03XVYtL_IwYGEJrzTDH9EMgIlXn10eHYqaH2ap7nSLIXNZGpI0vXqWqDEz09lYHdBao-w8GSh5dnSRNuF_LKrQX6J_Y8kB914xRkMw0OAZJPHPE9P-KFP6YpPVDeVzuqLB33vrWqPdMSc" alt=""><figcaption></figcaption></figure>

**1. POP3 Enumeration:**

* **Port Scanning:**
  * Identify systems running a POP3 server by conducting port scans. POP3 typically uses port 110 for unencrypted communication and port 995 for encrypted communication (POP3S).
* **Banner Grabbing:**
  * Capture banners from the POP3 service to gather information about the server and its software version. Tools like Netcat or Telnet can be used for banner grabbing.
* **User Enumeration:**
  * Attempt to enumerate valid usernames by using techniques like user enumeration commands or brute force attacks. This helps in identifying valid email accounts on the server.
* **Password Policy Assessment:**
  * Check for any password policy in place, such as minimum length requirements or account lockout policies. Understanding password policies is crucial for effective brute force attacks.

**2. Interacting with POP3:**

* **Telnet Connection:**
  *   Establish a Telnet connection to the POP3 server (if unencrypted) to interact with it manually. This can be done using the following command:

      ```bash
      telnet <pop3_server_ip> 110
      ```
* **Authentication:**
  *   Authenticate to the POP3 server using valid credentials. The `USER` and `PASS` commands are used for this purpose. For example:

      ```
      USER username
      PASS password
      ```
* **Retrieve Email Headers:**
  *   Use the `RETR` command to retrieve email headers. This command allows you to view the content of a specific email. Example:

      ```
      RETR 1
      ```
* **Delete Emails (Optional):**
  * If allowed, use the `DELE` command to delete emails from the server. Be cautious with this action, as it permanently removes emails.
* **List Emails:**
  * Use the `LIST` command to get a list of emails and their sizes. This provides an overview of the emails available on the server.

**3. Tools for POP3 Enumeration:**

* **Telnet:**
  * Telnet is a basic tool for manual interaction with POP3 servers. It allows you to send commands and receive responses directly.
* **Netcat:**
  * Netcat can be used for banner grabbing and basic interaction with POP3 servers. It provides a command-line interface for sending and receiving data.
* **Enumeration Tools:**
  * Tools like Metasploit or custom scripts can automate user enumeration and password attacks against POP3 servers.
