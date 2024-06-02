# 📼 Enum ACLs

{% embed url="https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/acl-persistence-abuse" %}

We can use Get-DomainObjectACL from PowerView to enumerate ACLs

If we target a specific groups like the Domain Admin Group, we could enumerate it the following way:

```
Get-DomainObjectAcl -Identity "Domain Admins" -ResolveGUIDs -Verbose
```

<figure><img src="../../.gitbook/assets/image (11) (1) (1).png" alt=""><figcaption></figcaption></figure>

Next is we want to check for modify rights/permissions for a specific user we can use FindInterestingDomainACL

For the user “student613” I would use the following:

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "studentx"}
```

But unfortunately we don't find anything but if we try to specify a group instead of a user ->

```
Find-InterestingDomainAcl -ResolveGUIDs | ?{$_.IdentityReferenceName -match "RDPUsers"}
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

The command is used to find and filter specific ACLs in a domain that relate to a group or user containing "RDPUsers" in their name. Here is a step-by-step explanation of what the command does:

1. **`explainFind-InterestingDomainAcl -ResolveGUIDs`**:
   * Executes the function to find interesting domain ACLs and resolves GUIDs into readable names.
2. **`|` (Pipe)**:
   * Passes the resulting list of ACLs to the next command.
3. **`?{ $_.IdentityReferenceName -match "RDPUsers" }`**:
   * Filters the ACLs to include only those where the `IdentityReferenceName` matches or contains the string "RDPUsers".

More in depth:

**`?{ $_.IdentityReferenceName -match "RDPUsers" }`**:

* The `?` is an alias for the `Where-Object` cmdlet, which filters objects based on a specified condition.
* `{ $_.IdentityReferenceName -match "RDPUsers" }` is the condition being applied.
  * `$_` represents the current object in the pipeline.
  * `IdentityReferenceName` is a property of the objects being filtered, likely representing the name of a user or group.
  * `-match "RDPUsers"` checks if the `IdentityReferenceName` contains the string "RDPUsers".

{% embed url="https://book.hacktricks.xyz/windows-hardening/basic-powershell-for-pentesters/powerview#acl" %}
