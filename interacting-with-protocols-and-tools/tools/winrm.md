# 🕺 WinRM

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

**Introduction to WinRM:**

Windows Remote Management (WinRM) is a management protocol introduced by Microsoft to enable remote management of Windows systems over HTTP(S). It allows administrators to remotely execute commands, scripts, and PowerShell cmdlets, as well as access management information on remote Windows machines. WinRM operates using the SOAP (Simple Object Access Protocol) over HTTP(S) and relies on the Windows Remote Management Service.

**Introduction to Evil-WinRM:**

Evil-WinRM is a tool designed for penetration testing and red teaming purposes. It leverages WinRM to provide an interactive shell-like interface to interact with Windows machines remotely. Evil-WinRM is particularly useful for gaining foothold access to Windows systems during penetration testing engagements or security assessments.

**Interacting with Evil-WinRM:**

```
evil-winrm -i target_ip -u username -p password
```

more connection templates for Evil-WinRM that you can use:

1.  **Basic Authentication**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD
    ```
2.  **NTLM Authentication**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -H HASH
    ```

    Replace `HASH` with the NTLM hash of the user's password.
3.  **Kerberos Authentication**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -k
    ```

    This command uses the current user's Kerberos ticket for authentication.
4.  **Password Prompt**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -P
    ```

    This will prompt you to enter the password securely without displaying it in the command line.
5.  **Use SSL**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -s
    ```

    This command establishes an encrypted SSL connection.
6.  **Specify Target Port**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -P PORT
    ```

    Replace `PORT` with the specific port number where the WinRM service is running if it's different from the default port (5985).
7.  **Specify Domain**:

    ```
    evil-winrm -i TARGET_IP -u DOMAIN\USERNAME -p PASSWORD
    ```

    Replace `DOMAIN` with the domain name if you're targeting a domain user.
8.  **No SSL Certificate Validation**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -S
    ```

    This command disables SSL certificate validation. Use with caution, as it can expose you to man-in-the-middle attacks.
9.  **Connection Timeout**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -t TIMEOUT_SECONDS
    ```

    Replace `TIMEOUT_SECONDS` with the desired timeout duration in seconds.
10. **Verbose Mode**:

    ```
    evil-winrm -i TARGET_IP -u USERNAME -p PASSWORD -v
    ```

    This command enables verbose mode, providing more detailed output for troubleshooting.

{% embed url="https://www.hackingarticles.in/a-detailed-guide-on-evil-winrm/" %}

{% embed url="https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-winrm" %}
