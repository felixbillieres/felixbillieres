# 🌲 Enum Domain in forest, map & identify trusts, external trusts

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
