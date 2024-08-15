# ⚡ DLL Hijacking

DLL hijacking is a method of injecting malicious code into a given service or application by loading an evil DLL, often replacing the original one, that will be executed when the service starts. This is possible due to the way some Windows applications search and load DLLs, more specifically, if the path to a service’s DLL isn’t already loaded or stored in the system, Windows will start looking for it in the environment path, an attacker can therefore place the malicious DLL in a directory that is part of it to trigger the malicious code.

<figure><img src="../../../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

When a Windows application is launched, it searches for DLL files that it needs to function.

Windows searches for these DLL files in a certain number of directories, typically in the following order: the application's directory, the Windows system directory, the Windows directory, and then in the PATH.

* Attackers can exploit this DLL loading process by placing a malicious DLL file in one of the directories where Windows searches for DLL files.
* If the legitimate DLL file is not present in these directories, Windows may load the malicious DLL file instead.
* The malicious DLL file can contain malicious code that can be executed with the privileges of the launched Windows application, allowing the attacker to gain elevated privileges.

<figure><img src="../../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

first, we go in our tools folder and run this Procmon file as administrator:

<figure><img src="../../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

It will open us an application, our goal now is to find all the dll's that are not found by our system:

<figure><img src="../../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

so we click on the filter option and change the query to `Result Is NAME NOT FOUND`

click on the include and then add button&#x20;

and start again but a filter for dll files:

<figure><img src="../../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

So we hit apply and then see the results:

<figure><img src="../../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

we can possibly overwrite the dll's if we can control the service and if the location is writable&#x20;

Let's go open a cmd and type in `sc start dllsvc`

<figure><img src="../../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

So in our previous app that we opened we saw some movement:

<figure><img src="../../../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

Program file or Temp file are possibly files that we can modify

we would generate a reverse shell with meterpreter and save it on our system as hijackme.dll but we are going to try something else ->

Let's move this windows\_dll.c file to our local machine

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There is a problem with the documentation, so escalation will not be possible, you already are able to spot any missing DLLs to escalate privilege so here is some documentation to pursue the attack:

{% embed url="https://juggernaut-sec.com/dll-hijacking/" %}

{% embed url="https://book.hacktricks.xyz/windows-hardening/windows-local-privilege-escalation/dll-hijacking" %}
