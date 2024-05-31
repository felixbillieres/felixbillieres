# 🌭 List users, computers, domain, and enterprise admins

In this section, we will enumerate the following for the dollarcorp domain: users, computers, domain administrators, and enterprise administrators.

### PowerView

We'll start by using Powershell to run InviShell:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

I manually open the `C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat` file and run the following commands to start enumeration:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

To go even deeper in our enumeration, we can use the select-object cmdlet (it's good to know that there is an alias for this cmdlet that is simply the select cmdlet ->

```
Get-DomainUser | select -ExpandProperty samaccountname
```

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

We can start to look for users that could seem more interesting to us or start looking for the member computers in the domain:

```
Get-DomainComputer | select -ExpandProperty dnshostname
```

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

To continue the enumeration, we could be interested in the domain groups and users that are part of certain groups ->

```
Get-DomainGroup -Identity "Domain Admins"
```

This would obviously be for the group Domain Admin but with our previous Get-DomainUser enumeration we can choose any of the groups

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

If we directly want to enumerate the users inside a group (and we're root domain) we can use the following command:

```
Get-DomainGroupMember -Identity "Enterprise Admins"
```

If we're not in the root domain we need to query the root domain as Enterprise Admins group is present only in the root of a forest.

```
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

### ADModule

We can start by enumerating all the users in the current domain using ADModule:

```
Get-ADUser -Filter *
```

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Same for the computers ->

```
Get-ADComputer -Filter *
```

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

We can print out specific properties of computers but we can also dump everything:

```
Get-AdComputer -Filter * -Properties *| select
```

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

To enumerate the Domain Administrators using ADModule we can query like that:

```
Get-ADGroupMember -Identity 'Domain Admins'
```

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Same as above, if we need to call a group that is not in our domain we need to query the root domain:

```
Get-ADGroupMember -Identity 'Domain Admins' -Server moneycorp.local
```

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>
