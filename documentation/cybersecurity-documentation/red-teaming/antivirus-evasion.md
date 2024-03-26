# 🎪 Antivirus Evasion

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

When it comes to AV evasion we have two primary types available:

* On-Disk evasion
* In-Memory evasion

On-Disk evasion is when we try to get a file (be it a tool, script, or otherwise) saved on the target, then executed. This is very common when working with executable (`.exe`) files.

In-Memory evasion is when we try to import a script directly into memory and execute it there. For example, this could mean downloading a PowerShell module from the internet or our own device and directly importing it without ever saving it to the disk.

***

Generally speaking, detection methods can be classified into one of two categories:

* Static Detection
* Dynamic / Heuristic / Behavioural Detection

**Static** detection methods usually involve some kind of signature detection. A very rudimentary system, for example, would be taking the hashsum of the suspicious file and comparing it against a database of known malware hashsums.

The other form of static detection is called **Byte matching** and is another form of signature detection which works by searching through the program looking to match sequences of bytes against a known database of bad byte sequences.&#x20;

Where **static** virus malware detection methods look at the file itself, **dynamic** methods look at how the file _acts._

The AV lookes at pre-defined rules about what type of action is malicious (e.g. is the program reaching out to a known bad website, or messing with values in the registry that it shouldn't be?), the AV can see how the program intends to act

The suspicious software can outright be executed inside a sandbox environment under close supervision from the AV software. If the program acts maliciously then it is quarantined and flagged as malware
