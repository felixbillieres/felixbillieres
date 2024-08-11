---
description: https://tryhackme.com/r/room/volatility
---

# 🏐 Volatility

<figure><img src="../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Five different plugins within Volatility allow you to dump processes and network connections:

* To list processes, we can use `pslist` The output from this plugin will include all current processes and terminated processes with their exit times.
  * Syntax: `python3 vol.py -f <file> windows.pslist`
* Usually malwares want to hide their running processes and will unlike them, making it invisible from the `pslist` command, we can use `psscan`;this technique of listing processes will locate processes by finding data structures that match `_EPROCESS`
  * Syntax: `python3 vol.py -f <file> windows.psscan`
* To list processes based on their process ID, therefore it can be useful to make a timeline for the occurring events, we can use the `pstree` command
  * Syntax: `python3 vol.py -f <file> windows.pstree`
* To identify network connections, we can simply use `netstat`, it will attempt to identify all memory structures with a network connection.
  * Syntax: `python3 vol.py -f <file> windows.netstat`
* And lastly, `dlllist` will list all DLLs associated with processes at the time of extraction
  * Syntax: `python3 vol.py -f <file> windows.dlllist`

#### Threat hunting

to identify injected processes and their PIDs along with the offset address and a Hex, Ascii, and Disassembly view of the infected area, we can use the very useful `malfind` command

Syntax: `python3 vol.py -f <file> windows.malfind`

Volatility also offers the capability to compare the memory file against YARA rules. `yarascan` will search for strings, patterns, and compound rules against a rule set. You can either use a YARA file as an argument or list rules within the command line.

Syntax: `python3 vol.py -f <file> windows.yarascan`

#### Practical Investigations

_**What is the build version of the host machine in Case 001?**_

```
python3 vol.py -f /Scenarios/Investigations/Investigation- 1.vmem windows.info
```

_**At what time was the memory file acquired in Case 001?**_

```
python3 vol.py -f /Scenarios/Investigations/Investigation- 1.vmem windows.info
```

_**What process can be considered suspicious in Case 001?**_

```
python3 vol.py -f /Scenarios/Investigations/Investigation- 1.vmem windows.psscan
```

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What is the parent process of the suspicious process in Case 001?**_

<figure><img src="../../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
python3 vol.py -f /Scenarios/Investigations/Investigation-1.vmem -o /home/thmanalyst windows.memmap.Memmap — pid 1640 — dump
```

_**What user-agent was employed by the adversary in Case 001?**_

```
python3 vol.py -f /Scenarios/Investigations/Investigation-1.vmem -o /home/thmanalyst windows.memmap.Memmap --pid 1640 --dump
```

And then&#x20;

```
strings /home/thmanalyst/*.dmp | grep -i "user-agent"
```

<figure><img src="../../../../.gitbook/assets/image (14) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Was Chase Bank one of the suspicious bank domains found in Case 001? (Y/N)**_

```
strings /home/thmanalyst/*.dmp | grep -i "chase"
```

<figure><img src="../../../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What suspicious process is running at PID 740 in Case 002?**_

```
 python3 vol.py -f /Scenarios/Investigations/Investigation-2.raw -o /home/thmanalyst windows.psscan
```

<figure><img src="../../../../.gitbook/assets/image (16) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What is the full path of the suspicious binary in PID 740 in Case 002?**_

```
 python3 vol.py -f /Scenarios/Investigations/Investigation-2.raw -o /home/thmanalyst windows.dlllist | grep 740
```

<figure><img src="../../../../.gitbook/assets/image (17) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What is the suspicious parent process PID connected to the decryptor in Case 002?**_

```
python3 vol.py -f /Scenarios/Investigations/Investigation-2.raw -o /home/thmanalyst windows.pstree
```

<figure><img src="../../../../.gitbook/assets/image (18) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What DLL is loaded by the decryptor used for socket creation in Case 002?**_

{% embed url="https://medium.com/@codingkarma/wannacry-analysis-and-cracking-6175b8cd47d4" %}

<figure><img src="../../../../.gitbook/assets/image (19) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**What mutex can be found that is a known indicator of the malware in question in Case 002?**_

```
 python3 vol.py -f /Scenarios/Investigations/Investigation-2.raw windows.handles | grep 1940
```

<figure><img src="../../../../.gitbook/assets/image (20) (1) (1).png" alt=""><figcaption></figcaption></figure>
