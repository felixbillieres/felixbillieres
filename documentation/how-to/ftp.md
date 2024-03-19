# 🥎 FTP

#### **FTP Enumeration and Interaction**

File Transfer Protocol (FTP) is a standard network protocol used to transfer files between a client and a server on a computer network. In the context of penetration testing and ethical hacking, enumerating and interacting with FTP servers can provide valuable information. Here's a guide on FTP enumeration and interaction:

<figure><img src="https://kinsta.com/wp-content/uploads/2023/01/How-FTP-works.png" alt=""><figcaption></figcaption></figure>

**\*\*1. Enumeration:**

* **Identify FTP Service:**
  *   Determine if an FTP service is running on the target system. You can use tools like Nmap to perform a port scan:

      ```bash
      nmap -p 21 <target_ip>
      ```
* **Anonymous FTP Login:**
  *   Check if anonymous FTP login is allowed. Some FTP servers permit login with a generic username and password. Use an FTP client or command-line tool like `ftp`:

      ```bash
      ftp <target_ip>
      ```

      Use "anonymous" as the username and provide any string (e.g., your email address) as the password.
* **Banner Grabbing:**
  *   Retrieve the FTP banner to identify the software and version running on the server. This can be done using telnet or Netcat:

      ```bash
      telnet <target_ip> 21
      ```

      Once connected, observe the banner information.

**\*\*2. Connecting to FTP:**

* **Using FTP Client:**
  *   Connect to the FTP server using an FTP client like FileZilla, WinSCP, or the built-in `ftp` command:

      ```bash
      ftp <target_ip>
      ```

      Enter the username and password when prompted.
* **Anonymous Login:**
  *   If anonymous login is allowed, use the following command:

      ```bash
      ftp <target_ip>
      ```

      Username: anonymous Password: \<your\_email\_address>
* **Interactive FTP:**
  * Once connected, you can navigate directories, list files, upload, and download files interactively.

**\*\*3. FTP Commands:**

* **List Files:**
  *   Use the following command to list files on the FTP server:

      ```bash
      ls
      ```
* **Change Directory:**
  *   Change the current working directory:

      ```bash
      cd <directory_name>
      ```
* **Download File:**
  *   Download a file from the FTP server:

      ```bash
      get <file_name>
      ```
* **Upload File:**
  *   Upload a file to the FTP server:

      ```bash
      put <local_file_path>
      ```
* **Quit FTP Session:**
  *   Terminate the FTP session:

      ```bash
      bye
      ```

**\*\*4. FTP Brute Force:**

* **Hydra for FTP Brute Force:**
  *   Use Hydra to perform brute force attacks on the FTP login. Replace `<user_list>` and `<password_list>` with the respective lists:

      ```bash
      hydra -t 4 -vV -l <user_list> -P <password_list> ftp://<target_ip>
      ```

**\*\*5. Notes:**

* Be cautious when conducting FTP brute force attacks, as they can lead to account lockouts or other security measures.
* Always ensure you have proper authorization before attempting to access or manipulate files on an FTP server.
