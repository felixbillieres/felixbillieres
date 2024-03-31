# 🗺️ CrackMapExec

<figure><img src="../../.gitbook/assets/image (619).png" alt=""><figcaption></figcaption></figure>

CrackMapExec (CME) is a powerful penetration testing tool used by security professionals and red teamers to assess the security of Windows networks. It's designed to automate various tasks related to reconnaissance, exploitation, and post-exploitation on Windows networks, making it an essential tool in the arsenal of ethical hackers and security researchers.

{% embed url="https://cheatsheet.haax.fr/windows-systems/exploitation/crackmapexec/" %}

**Key Features:**

1. **SMB Enumeration**: CME can enumerate SMB (Server Message Block) shares, users, groups, and domains within a Windows network. This allows testers to identify potential attack vectors and assess the network's security posture.
2. **Credential Stuffing**: CME supports credential stuffing attacks, allowing testers to verify the validity of credentials obtained from various sources. It can test username and password combinations against multiple hosts simultaneously.
3. **Pass-the-Hash**: CME can perform Pass-the-Hash attacks by using NTLM hashes obtained from compromised systems to authenticate and execute commands on other systems within the network.
4. **Lateral Movement**: CME facilitates lateral movement by providing features to execute commands and scripts remotely on compromised systems. It supports various authentication methods, including password-based, hash-based, and Kerberos authentication.
5. **Post-Exploitation Modules**: CME includes post-exploitation modules to gather information, escalate privileges, and perform other actions on compromised systems. These modules can help testers maintain access and expand their foothold within the network.

**Installation:**

CME can be installed on Linux-based systems using Python's package manager, pip. Follow these steps to install CME:

1. Open a terminal window.
2.  Run the following command to install CME:

    ```
    pip install crackmapexec
    ```

**Basic Usage:**

Once installed, you can start using CME to assess Windows networks. Here are some basic commands to get you started:

1.  **Enumerate SMB Shares**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD --shares
    ```

    Replace `TARGET_IP`, `USERNAME`, and `PASSWORD` with the appropriate values for your target environment. This command will enumerate SMB shares on the target machine.
2.  **Credential Stuffing**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -P PASSWORD_LIST
    ```

    Replace `TARGET_IP`, `USERNAME`, and `PASSWORD_LIST` with the appropriate values for your target environment. This command will test username and password combinations from a password list against the target machine.
3.  **Pass-the-Hash**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -H NTLM_HASH
    ```

    Replace `TARGET_IP`, `USERNAME`, and `NTLM_HASH` with the appropriate values for your target environment. This command will perform a Pass-the-Hash attack using the specified NTLM hash.

Connection templates for CrackMapExec (CME) :

1.  **Basic Authentication**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD
    ```
2.  **NTLM Authentication**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -H HASH
    ```

    Replace `HASH` with the NTLM hash of the user's password.
3.  **Kerberos Authentication**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -k
    ```

    This command uses the current user's Kerberos ticket for authentication.
4.  **Password Prompt**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -P
    ```

    This will prompt you to enter the password securely without displaying it in the command line.
5.  **Specify Target Port**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD -p PORT
    ```

    Replace `PORT` with the specific port number where the SMB service is running if it's different from the default port (445).
6.  **Specify Domain**:

    ```
    crackmapexec smb TARGET_IP -u DOMAIN/USERNAME -p PASSWORD
    ```

    Replace `DOMAIN` with the domain name if you're targeting a domain user.
7.  **No SSL Certificate Validation**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD --no-ssl
    ```

    This command disables SSL certificate validation. Use with caution, as it can expose you to man-in-the-middle attacks.
8.  **Connection Timeout**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD --timeout TIMEOUT_SECONDS
    ```

    Replace `TIMEOUT_SECONDS` with the desired timeout duration in seconds.
9.  **Verbose Mode**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD -v
    ```

    This command enables verbose mode, providing more detailed output for troubleshooting.
10. **Enumerate Shares**:

    ```
    crackmapexec smb TARGET_IP -u USERNAME -p PASSWORD --shares
    ```

    This command will enumerate available shares on the target machine.

Remember to replace `TARGET_IP`, `USERNAME`, `PASSWORD`, `HASH`, `DOMAIN`, and `PORT` with the appropriate values for your target environment. Additionally, ensure that you have proper authorization and permission to perform penetration testing activities on the target system. Use these commands responsibly and ethically.

**Additional Resources:**

* [CME GitHub Repository](https://github.com/byt3bl33d3r/CrackMapExec)
* [CME Documentation](https://github.com/byt3bl33d3r/CrackMapExec/wiki)
* [CME Usage Examples](https://github.com/byt3bl33d3r/CrackMapExec/wiki/Usage-Examples)
