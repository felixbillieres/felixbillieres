# 🗼 Token Impersonation

Nice page about token impersonation with example:

{% content-ref url="../../../../red-team/technical-knowledge/privilege-escalation/windows/impersonation-and-potato-attacks.md" %}
[impersonation-and-potato-attacks.md](../../../../red-team/technical-knowledge/privilege-escalation/windows/impersonation-and-potato-attacks.md)
{% endcontent-ref %}

<figure><img src="../../../../.gitbook/assets/image (21) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

This technique that allows an attacker to steal the access token of a logged-on user on the system without knowing their credentials and impersonate them to perform operations with their privileges.

There are two types of tokens related to the token impersonation technique —&#x20;

**Delegation and Impersonation**:

Delegation refers to the process of granting permissions or access rights to a user or service, allowing them to act on behalf of another user or service.

Impersonation, on the other hand, refers to the process of a user or service pret to be another user or service in order to gain access to resources.&#x20;

<figure><img src="../../../../.gitbook/assets/image (23) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can now impersonate this user

<figure><img src="../../../../.gitbook/assets/image (18) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Attempt to dump hashes as non-Domain Admin with mimikatz->

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)  (10).png" alt=""><figcaption></figcaption></figure>

If we had a domain admin token available, we could impersonate it like this:

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) ( (1).png" alt=""><figcaption></figcaption></figure>

First you spot the token

```
impersonate_token MARVEL\\administrator
```

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

we could now try to dump hashes as domain admin with mimikatz:

<figure><img src="../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

that would give the following output:

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we found some way to impersonate a domain admin account, we could use remote commands to go further:

```
net user /add felix Mot2pAsse /domain
net group "Domain Admins" felix /ADD /DOMAIN
```

The first command, `net user /add felix Mot2pAsse /domain`, creates a new user account named "felix" with the password "Mot2pAsse" on the Windows domain. This command creates a new security principal in the domain, which can be used to authenticate and authorize access to resources.

The second command, `net group "Domain Admins" felix /ADD /DOMAIN`, adds the user "felix" to the "Domain Admins" group on the Windows domain. This group is a built-in group that is granted administrative privileges on the domain, allowing members to manage domain-wide settings and resources. By adding the user "felix" to this group, you are granting them administrative privileges on the domain.

And if this user add is successful, we could go and run secretsdump.py ->

```
secretsdump.py MARVEL.local/felix:'Mot2pAsse'@10.10.14.94
```

It would look something like this:

<figure><img src="../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Example in this box:

{% content-ref url="../../../../capture-the-flags/active-directory-and-networks/sauna.md" %}
[sauna.md](../../../../capture-the-flags/active-directory-and-networks/sauna.md)
{% endcontent-ref %}
