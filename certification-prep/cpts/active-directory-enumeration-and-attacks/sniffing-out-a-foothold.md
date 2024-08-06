# 👃 Sniffing out a Foothold

## LLMNR/NBT-NS Poisoning - from Linux

After enumerating on the domain, finding some usernames, we will work through two different techniques side-by-side: network poisoning and password spraying in order to find some cleartext credentials to gain foothold

We are going to see 2 techniques, a Man-in-the-Middle attack on Link-Local Multicast Name Resolution (LLMNR) and NetBIOS Name Service (NBT-NS) broadcasts.

[Link-Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) and [NetBIOS Name Service](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063\(v=technet.10\)?redirectedfrom=MSDN) (NBT-NS) are Microsoft Windows components that serve as alternate methods of host identification that can be used when DNS fails.

You can find more in my article about address resolution in active directory:

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/active-directory/ad-in-theory/questions-about-ad#address-resolution" %}

If a machine attempts to resolve a host but DNS resolution fails, typically, the machine will try to ask all other machines on the local network for the correct host address via LLMNR and If LLMNR fails, the NBT-NS will be used. NBT-NS identifies systems on a local network by their NetBIOS name.

We will user reponder.py to respond to the broadcast request&#x20;

### Timeline of a LLMNR/NBT-NS Poisoning

1. A host attempts to connect to the print server at \\\print01.inlanefreight.local, but accidentally types in \\\printer01.inlanefreight.local.
2. The DNS server responds, stating that this host is unknown.
3. The host then broadcasts out to the entire local network asking if anyone knows the location of \\\printer01.inlanefreight.local.
4. The attacker (us with `Responder` running) responds to the host stating that it is the \\\printer01.inlanefreight.local that the host is looking for.
5. The host believes this reply and sends an authentication request to the attacker with a username and NTLMv2 password hash.
6. This hash can then be cracked offline or used in an SMB Relay attack if the right conditions exist.

#### Using Responder

We first start responder with default settings ->

```
sudo responder -I ens224 
```

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Best practice is to let it run in background while we enumerate to maximize the number of hashes that we can obtain.

Once we got few hits, we can use hash cat with mode 5600 ->

{% embed url="https://hashcat.net/wiki/doku.php?id=example_hashes" %}

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This would be the syntax:

```
hashcat -m 5600 hashfile.txt /usr/share/wordlists/rockyou.txt 
```

And if everything works just fine, we should be able to gain foothold into the domain to begin further enumeration.

### Practical example

After pivoting in the target machine through ssh, I launch responder and wait for some hashes to pop ->

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We get a hit, now we copy and paste the hash in a file and hop on hashcat ->

Looking back at the questions, we had to crack the hash of an account that starts with letter B so I relaunched responder and waited for a bit and did just what I did before

```
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (1051).png" alt=""><figcaption></figcaption></figure>

## LLMNR/NBT-NS Poisoning - from Windows

So let's imagine we're on a Windows computer, we can also do this attack, we're going to see the process with the Inveigh tool, a tool like responder but written in PowerShell and C#

#### Start using Inveigh

```
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> (Get-Command Invoke-Inveigh).Parameters

Key                     Value
---                     -----
ADIDNSHostsIgnore       System.Management.Automation.ParameterMetadata
KerberosHostHeader      System.Management.Automation.ParameterMetadata
ProxyIgnore             System.Management.Automation.ParameterMetadata
PcapTCP                 System.Management.Automation.ParameterMetadata
[...]
```

Now we need to start Inveigh with LLMNR and NBNS spoofing, and output to the console and write to a file.

```
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

<figure><img src="../../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And we see some hits, the interface is very similar to Responder&#x20;

It's good to know that the Powershell version is not updated anymore, the author updates the C# version

to run the C# tool we can just:

```
PS C:\htb> .\Inveigh.exe
```

### Practical Example

I first connect through RDP:

```
xfreerdp /v:10.129.199.67 /u:htb-student /p:Academy_student_AD!
```

I find the Inveigh file:

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I run the exe file:

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And quickly get the hash of svc\_qualys:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I take it on my local machine, put it in a file and launch hashcat:

```
hashcat -m 5600 qualys /usr/share/wordlists/rockyou.txt
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
