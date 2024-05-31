# 📼 Enum ACLs

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse" %}

We can use Get-DomainObjectACL from PowerView to enumerate ACLs

If we target a specific groups like the Domain Admin Group, we could enumerate it the following way:

```
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Next is we want to check for modify rights/permissions for a specific user we can use FindInterestingDomainACL

For the user “student613” I would use the following:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "studentx"}
```

But unfortunately we don't find anything but if we try to specify a group instead of a user ->

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
