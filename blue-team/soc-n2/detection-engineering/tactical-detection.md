# 🌸 Tactical Detection

Unique IOCs of previous intrusions are good examples of Threat Intel as they’re traces of the specific adversary that your environment has already faced. The inclusion of these IOCs in your detection mechanism will help spot re-intrusion of that specific adversary immediately

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we will transform these IOCs into detection rules in a vendor-agnostic format using Sigma.

A basic example of how it can be written into a functional Sigma rule is as follows.

```shell-session
title: Executable Download from Suspicious Domains
status: test
description: Detects download of executable types from hosts found in the IOC spreadsheet
author: Mokmokmok
date: 2022/08/23
modified: 2022/08/23
logsource:
  category: proxy
detection:
  selection:
    c-uri-extension:
      - 'exe'
    r-dns:
      - 'bad3xe69connection.io'
      - 'kind4bad.com'
      - 'nic3connection.io'
  condition: selection
fields:
  - c-uri
falsepositives:
  - Unkown
level: medium
```

there are a number of nice repositories that contain user-submitted Sigma rules that anyone can use. You can plug one directly into your SIEM for immediate value

**Change Sigma rule to be a valid rule for ElastAlert**

{% embed url="https://uncoder.io/" %}

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Setting up a Basic Tripwire:

Suppose there are ultra-sensitive files your organization intends to keep secret, such as a hidden treasure map. In that case, you can set alerts for instances that the said map has been accessed, edited, and deleted among other things, and then filter out the ones allowed access to make detections more actionable.

Tripwires are a great way to supplement the detection mechanisms that you already have in place. It’s a way to cover “unknown unknowns” and even study an adversary. The most common tripwires are Honeypots and Hidden Files and Folders.

_**Local Security Policy -> Security Settings → Local Policies → Audit Policy.**_

We have to specify in the actual file or folder that we intend to monitor&#x20;

on the desktop → New → Text Document. name it "Secret Document". Right-click the document → Properties → Security → Advanced → Auditing.

This is where we will create and specify an audit entry for our Secret Document. Click on Add → Select a principal.

text box, write "Everyone", then press Enter.

we click on OK on our Secret Document properties.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Anyone who accesses it will be logged and its details will be recorded in the Security event log with an Event ID 4663. This Event ID, along with the other Object Access Event IDs, can then be filtered and furnished into alerts that would immediately tell your analysts of tripwires being activated
