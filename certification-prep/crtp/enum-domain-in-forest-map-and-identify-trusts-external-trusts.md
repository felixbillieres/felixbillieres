# 🌲 Enum Domain in forest, map & identify trusts, external trusts

### PowerView

We'll start by enumerating all domains in the current forest:

```
Get-ForestDomain -Verbose
```

<figure><img src="../../.gitbook/assets/image (1059).png" alt=""><figcaption></figcaption></figure>

So this is very interesting information, let's dig deeper by mapping the trusts:

```
Get-DomainTrust
```

<figure><img src="../../.gitbook/assets/image (1060).png" alt=""><figcaption></figcaption></figure>

To get what we're saying here, we can look at the AD theory documentation&#x20;

{% embed url="https://felix-billieres.gitbook.io/felix-billieres/active-directory/ad-in-theory/questions-about-ad#trust" %}

If we want to list only the external trusts in the moneycorp.local forest:

```
Get-ForestDomain | %{Get-DomainTrust -Domain $_.Name} | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

<figure><img src="../../.gitbook/assets/image (1061).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1082).png" alt=""><figcaption><p>So eurocorp.local is indeed a external domain</p></figcaption></figure>

* **`Get-ForestDomain`**:
  * This cmdlet retrieves all domains in the current Active Directory forest.
  * The output is a collection of domain objects.
* **`%{Get-DomainTrust -Domain $_.Name}`**:
  * The `%` symbol is an alias for the `ForEach-Object` cmdlet. This means for each domain object retrieved by `Get-ForestDomain`, the script will execute the script block `{Get-DomainTrust -Domain $_.Name}`.
  * `$_` represents the current object in the pipeline (each domain object).
  * `Get-DomainTrust -Domain $_.Name` retrieves the trust relationships for the domain specified by `$.Name`.
* **`?{$_.TrustAttributes -eq "FILTER_SIDS"}`**:
  * The `?` symbol is an alias for the `Where-Object` cmdlet. This filters objects in the pipeline based on a condition.
  * `$_` represents each trust relationship object coming from the previous command.
  * `{$_.TrustAttributes -eq "FILTER_SIDS"}` filters the trust relationships to only include those where the `TrustAttributes` property is equal to `"FILTER_SIDS"`.

#### What the Command Does:

* **Retrieve all domains** in the current Active Directory forest.
* **For each domain**, retrieve its trust relationships.
* **Filter** the trust relationships to only include those where the `TrustAttributes` property is set to `"FILTER_SIDS"`.

If we want to identify external trusts of the dollarcorp domain, we can use the below command:

```
Get-DomainTrust | ?{$_.TrustAttributes -eq "FILTER_SIDS"}
```

<figure><img src="../../.gitbook/assets/image (12) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see the trust is bidirectional, that means that the 2 domains can extract information from each other:

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So if the trust is either bidirectional or one-way trust from eurocorp.local to dollarcorp we would be able to use the below command. the next task is to enumerate trusts for eurocorp.local forest:

```
Get-ForestDomain -Forest eurocorp.local | %{Get-DomainTrust -Domain $_.Name}
```

But there is an error because PowerView attempted to list trusts even for eu.eurocorp.local. Because external trust is non-transitive it was not possible!

<figure><img src="../../.gitbook/assets/image (1083).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### AD-Module

Now let's try all of this using AD-Module&#x20;

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat
Import-Module C:\AD\Tools\ADModule-master\Microsoft.ActiveDirectory.Management.dll
Import-Module C:\AD\Tools\ADModule-master\ActiveDirectory\ActiveDirectory.psd1
```

and start by enumerating all the domains in the current forest:

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we want to map all the trusts in the current domain, it's a very simple syntax

```
Get-ADTrust -Filter * 
```

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now if we want to list all the trusts in the moneycorp.local forest:

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

But if we'd rather only list external trusts in moneycorp.local domain ->

```
(Get-ADForest).Domains | %{Get-ADTrust -Filter '(intraForest -ne $True) -and (ForestTransitive -ne $True)' -Server $_} 
```

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To identify external trusts of the dollarcorp domain, we can use the below command. The output is same as above because there is just one external trust in the entire forest. Otherwise, output of the above command would be different from the below one:

```
Get-ADTrust -Filter '(intraForest -ne $True) -and(ForestTransitive -ne $True)' 
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And to profit our inbound trust with eurocorp.local, we can enumerate trusts for it:

```
Get-ADTrust -Filter * -Server eurocorp.local
```

```
PS C:\AD\Tools>Get-ADTrust -Filter * -Server eurocorp.local     
Direction : BiDirectional
DisallowTransivity : False
DistinguishedName : CN=eu.eurocorp.local,CN=System,DC=eurocorp,DC=local
ForestTransitive : False
IntraForest : True
IsTreeParent : False
IsTreeRoot : False
Name : eu.eurocorp.local
ObjectClass : trustedDomain
ObjectGUID : bfc7a899-cc5d-4303-8176-3b8381189fae
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source : DC=eurocorp,DC=local
Target : eu.eurocorp.local
TGTDelegation : False
TrustAttributes : 32
TrustedPolicy :
TrustingPolicy :
TrustType : Uplevel
UplevelOnly : False
UsesAESKeys : False
UsesRC4Encryption : False
```
