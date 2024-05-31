---
description: https://tryhackme.com/r/room/windowsuseractivity
---

# 🪟 Windows User Activity Analysis

On the Desktop we can see 12 folders/files:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The Windows Registry is basically an organized database that stores all kinds of settings and configuration information for your Windows operating system, as well as for installed applications and user preferences. Think of it as the control center for your computer's settings.

It's stored in what we call Hives;

{% embed url="https://fr.wikipedia.org/wiki/Base_de_registre" %}

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The user related hives are SECURITY and DEFAULT which have very sensitive files:

`NTUSER.DAT`:

* User-specific settings for each user profile. Contains information about user preferences and specific configurations.

`USRCLASS.DAT`:

* User-specific class settings. Stores information about Explorer's settings and interaction.

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Here are some important registry hives:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If you Win + R and type `regedit` you can access those:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
