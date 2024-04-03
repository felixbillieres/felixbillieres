# 🍯 Honeypots

A honeypot is a deliberately vulnerable security tool designed to attract attackers and record the actions of adversaries. Honeypots can be used in a defensive role to alert administrators of potential breaches and to distract attackers away from real infrastructure.

<figure><img src="../../../.gitbook/assets/image (623).png" alt=""><figcaption></figcaption></figure>

### Types of Honeypots

**Low-Interaction honeypots** offer little interactivity to the adversary and are only capable of simulating the functions that are required to simulate a service and capture attacks against it.

([mailoney](https://github.com/awhitehatter/mailoney) and [dionaea](https://github.com/DinoTools/dionaea))

**Medium-Interaction** honeypots collect data by emulating both vulnerable services as well as the underlying OS, shell, and file systems. This allows adversaries to complete initial exploits and carry out post-exploitation activity.

([Cowrie](https://github.com/cowrie/cowrie))

**High-Interaction** honeypots are fully complete systems that are usually Virtual Machines that include deliberate vulnerabilities. Adversaries should be able (but not necessarily allowed) to perform any action against the honeypot as it is a complete system.

<figure><img src="../../../.gitbook/assets/image (624).png" alt=""><figcaption></figcaption></figure>

### Deployment Location

**Internal honeypots** are deployed inside a LAN. This type can act as a way to monitor a network for threats originating from the inside, for example, attacks originating from trusted personnel or attacks that by-parse firewalls like phishing attacks.

**External honeypots** are deployed on the open internet and are used to monitor attacks from outside of the LAN. These honeypots are able to collect much more data on attacks since they are effectively guaranteed to be under attack at all times.

<figure><img src="../../../.gitbook/assets/image (625).png" alt=""><figcaption></figcaption></figure>

### Cowrie SSH Honeypot

<figure><img src="../../../.gitbook/assets/image (626).png" alt=""><figcaption></figcaption></figure>

there are ways to identify this type of Cowrie deployment. For example, it's not possible to execute bash scripts as this is a limitation of low and medium interaction honeypots. It's also possible to identify the default installation as it will mirror a Debian 5 Installation and features a user account named Phil. The default file system also references an outdated CPU.﻿

<figure><img src="../../../.gitbook/assets/image (628).png" alt=""><figcaption><p>file isn't here after reconnection</p></figcaption></figure>

### Cowrie Event Logging

There is a JSON parser `jq` on the demo machine to simplify log parsing.

<figure><img src="../../../.gitbook/assets/image (629).png" alt=""><figcaption></figcaption></figure>

The amount of data collected by honeypots, especially external deployments can quickly exceed the point where it's no longer practical to parse manually. As a result, it's often worth deploying Honeypots alongside a logging platform like the ELK stack.

<figure><img src="../../../.gitbook/assets/image (630).png" alt=""><figcaption></figcaption></figure>

### SSH and Brute-Force Attacks

By default, Cowrie will only expose SSH. This means adversaries will only be able to compromise the honeypot by attacking the SSH service.

Defending against these attacks is relatively simple in most cases as they can be defeated by only allowing public-key authentication or by using strong passwords.

{% content-ref url="../../../interacting-with-protocols-and-tools/tools/hydra.md" %}
[hydra.md](../../../interacting-with-protocols-and-tools/tools/hydra.md)
{% endcontent-ref %}

### Typical Post Exploitation Activity

The majority of attacks against typical SSH deployments are automated in some way. As a result, most of the post-exploitation activity that takes place after a bot gains initial access to the honeypot will follow a broad pattern.

_**What CPU does the honeypot "use"?**_

<figure><img src="../../../.gitbook/assets/image (631).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (632).png" alt=""><figcaption></figcaption></figure>

### Identification Techniques

#### Bot Identification

some artifacts tend to be consistent across bots including, the IP addresses requested by bots and the specific order of commands. Identifiable messages may also be present in scripts or commands though this is uncommon.

### SSH Tunnelling

#### Attacks Performed Using SSH Tunnelling

Some bots will not perform any actions directly against honeypot and instead will leverage a compromised SSH deployment itself. This is accomplished with the use of SSH tunnels. In short, SSH tunnels forward network traffic between nodes via an encrypted tunnel.

#### SSH Tunnelling Data in Cowrie

By default, Cowrie will record all the SSH tunnelling requests received by the honeypot but, will not forward them on to their destination. This data is of particular importance as it allows for the monitoring and discovery of web attacks, that may not have been found by another honeypot.
