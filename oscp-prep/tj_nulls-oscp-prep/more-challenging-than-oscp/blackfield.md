---
description: https://app.hackthebox.com/machines/255
---

# 🧑‍✈️ Blackfield

We start by doing a quick nmap:&#x20;

<figure><img src="../../../.gitbook/assets/image (893).png" alt=""><figcaption></figcaption></figure>

We can see DNS on port 53, Kerberos on port 88 and secured LDAP on port 3268 and a domain BLACKFIELD.local

So in our /etc/hosts we add the following:

```
10.129.229.17 BLACKFIELD.local BLACKFIELD
```

It's good to note we do not have a webserver&#x20;

then, always good to try rpcclient on an Active Directory:

<figure><img src="../../../.gitbook/assets/image (894).png" alt=""><figcaption></figcaption></figure>

```
rpcclient 10.129.229.17
#did not work
rpcclient 10.129.229.17 -U ''
#we tried to do a null auth and it worked
```

We tried enumerating users with the following command:

```
enumdomusers
```

But we got access denied so we leave rpcclient. More on enumerating rpccclient ->

{% embed url="https://www.hackingarticles.in/active-directory-enumeration-rpcclient/" %}

Since we saw SMB, we can try enumerating that:

```
smbclient -L 10.129.229.14
smbclient -L 10.129.229.14 -U ''
```

So we get few shares but we got the same output from the 2 commands:

<figure><img src="../../../.gitbook/assets/image (895).png" alt=""><figcaption></figcaption></figure>

We continue our smb enumeration with CrackMapExec since smbclient is not very verbose on the read/write side of things:

It's good to know that some tools need multiple tries to be sure we didn't miss anything:

```
cme smb 10.129.229.17 --shares
cme smb 10.129.229.17 --shares -u ''
cme smb 10.129.229.17 --shares -u '' -p ''
cme smb 10.129.229.17 --shares -u 'testUser'
cme smb 10.129.229.17 --shares -u 'testUser' -p ''
```

<figure><img src="../../../.gitbook/assets/image (896).png" alt=""><figcaption></figcaption></figure>

So know we know we can read the IPC and "profiles shares:

```
smbclient '//10.129.229.17/profiles$'
```

We put it in single quotes because the $ is a shady character that could cause some errors

<figure><img src="../../../.gitbook/assets/image (897).png" alt=""><figcaption></figcaption></figure>

So we got a big list of users, the good thing to do now is to mount it:

