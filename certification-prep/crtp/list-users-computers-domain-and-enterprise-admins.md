# 🌭 List users, computers, domain, and enterprise admins

In this section, we will enumerate the following for the dollarcorp domain: users, computers, domain administrators, and enterprise administrators.

### PowerView

We'll start by using Powershell to run InviShell:

<figure><img src="../../.gitbook/assets/image (18) (1).png" alt=""><figcaption></figcaption></figure>

I manually open the `C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat` file and run the following commands to start enumeration:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we want quick informations about our domain, we can input the following:

```
Get-DomainController
```

To go even deeper in our enumeration, we can use the select-object cmdlet (it's good to know that there is an alias for this cmdlet that is simply the select cmdlet ->

```
Get-DomainUser | select -ExpandProperty samaccountname
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

There could be users with passwords who never expires or passwords in description field ->

<figure><img src="../../.gitbook/assets/image (1072).png" alt=""><figcaption></figcaption></figure>

We can start to look for users that could seem more interesting to us or start looking for the member computers in the domain:

```
Get-DomainComputer | select -ExpandProperty dnshostname
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To continue the enumeration, we could be interested in the domain groups and users that are part of certain groups ->

```
Get-DomainGroup -Identity "Domain Admins"
```

This would obviously be for the group Domain Admin but with our previous Get-DomainUser enumeration we can choose any of the groups

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we directly want to enumerate the users inside a group (and we're root domain) we can use the following command:

```
Get-DomainGroupMember -Identity "Enterprise Admins"
```

If we're not in the root domain we need to query the root domain as Enterprise Admins group is present only in the root of a forest.

```
Get-DomainGroupMember -Identity "Enterprise Admins" -Domain moneycorp.local
```

If we want to get password policy for password spraying:

```
Get-DomainPolicyData
```

<figure><img src="../../.gitbook/assets/image (1071).png" alt=""><figcaption></figcaption></figure>

When doing a stealthy red team engagment we need to look at the logonCount, if a user with a single digit logonCount is a non-active user and will be detected

<figure><img src="../../.gitbook/assets/image (1073).png" alt=""><figcaption></figcaption></figure>

Same for computers:

<figure><img src="../../.gitbook/assets/image (1074).png" alt=""><figcaption></figcaption></figure>

### ADModule

We can start by enumerating all the users in the current domain using ADModule:

```
Get-ADUser -Filter *
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Same for the computers ->

```
Get-ADComputer -Filter *
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can print out specific properties of computers but we can also dump everything:

```
Get-AdComputer -Filter * -Properties *| select
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To enumerate the Domain Administrators using ADModule we can query like that:

```
Get-ADGroupMember -Identity 'Domain Admins'
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Same as above, if we need to call a group that is not in our domain we need to query the root domain:

```
Get-ADGroupMember -Identity 'Domain Admins' -Server moneycorp.local
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
