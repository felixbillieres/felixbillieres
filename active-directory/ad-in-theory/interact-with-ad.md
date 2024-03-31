# 📨 interact with AD

Identify current user domain in powershell:

```
$env:USERDNSDOMAIN
(Get-ADDomain).DNSRoot
```

Identify current computer domain in powershell:

```
(Get-WmiObject Win32_ComputerSystem).Domain
```

Forest information with Get-ADForest

```
Get-ADForest
```

#### Functional Modes <a href="#functional-modes" id="functional-modes"></a>

The modes are named based on the minimum Windows Server operative system required to work with them

```
(Get-ADForest).ForestMode
(Get-ADDomain).DomainMode
```

You can see the trusts of your domain with `nltest`

```
PS C:\Users\Administrator> nltest /domain_trusts
List of domain trusts:
    0: CONTOSO contoso.local (NT 5) (Direct Outbound) ( Attr: foresttrans )
    1: ITPOKEMON it.poke.mon (NT 5) (Forest: 2) (Direct Outbound) (Direct Inbound) ( Attr: withinforest )
    2: POKEMON poke.mon (NT 5) (Forest Tree Root) (Primary Domain) (Native)
The command completed successfully
```

Here we can see that our current domain is `poke.mon` (cause of the `(Primary Domain)` attribute) and there are a couple of trusts. The outbound trust with `contoso.local` indicates that its users can access to our domain, `poke.mon`. Moreover, there is a second bidirectional trust with `it.poke.mon` that is a subdomain of `poke.mon` and it is in the same forest.

```powershell
PS C:\Users\Anakin> nltest /domain_trusts
List of domain trusts:
    0: POKEMON poke.mon (NT 5) (Direct Inbound) ( Attr: foresttrans )
    1: CONTOSO contoso.local (NT 5) (Forest Tree Root) (Primary Domain) (Native)
The command completed successfully
```

Consequently, if we check the trust of `contoso.local`, we can see an inbound connection from `poke.mon`, which is consistent with the previous information. So users of `contoso.local` can access to `poke.mon`.

#### Trust transitivity <a href="#trust-transitivity" id="trust-transitivity"></a>

{% embed url="https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2008-r2-and-2008/cc754612(v=ws.10)" %}

```
      (trusting)   trusts   (trusted)  (trusting)   trusts   (trusted)
  Domain A  ------------------->  Domain B --------------------> Domain C
                    access                          access
            <-------------------           <--------------------
```

if the trust between `Domain A` and `Domain B` is transitive, then the users of `Domain C` can access to `Domain A` by traversing both trusts. If the `Domain A --> Domain B`trust was nontransitive, the `Domain C` users couldn't access to `Domain A`, but `Domain B` users could.

