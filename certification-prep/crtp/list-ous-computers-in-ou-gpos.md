# 😸 List OUs,Computers in OU, GPOs

In this section, we will enumerate the following for the dollarcorp domain: List all the OUs, List all the computers in the StudentMachines OU. List the GPOs and Enumerate GPO applied on the StudentMachines OU.

We start by loading PowerView.ps1

If we want to list all the OUs:

```
Get-DomainOU
```

<figure><img src="../../.gitbook/assets/image (1062).png" alt=""><figcaption></figcaption></figure>

If we want to be a bit more slick, we can get only the name of the OUs ->

```
Get-DomainOU | select -ExpandProperty name
```

<figure><img src="../../.gitbook/assets/image (1063).png" alt=""><figcaption></figcaption></figure>

Now we want to target very specific stuff, let's go and fetch all the computers in a specific OU ->

```
(Get-DomainOU -Identity StudentMachines).distinguishedname | %{Get-DomainComputer -SearchBase $_} | select name
```

<figure><img src="../../.gitbook/assets/image (1064).png" alt=""><figcaption></figcaption></figure>

Now if we want to enumerate the GPOs

```
Get-DomainGPO
```

<figure><img src="../../.gitbook/assets/image (1065).png" alt=""><figcaption></figcaption></figure>

If for some reason we needed to list the GPOs on the StudenMachines OU, we would need to enumerate the gplink attribute with the following command ->

```
(Get-DomainOU -Identity StudentMachines).gplink
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then we can use this information for the following command ->

```
Get-DomainGPO -Identity '{7478F170-6A0C-490C-B355-9E4618BC785D}'
```

<figure><img src="../../.gitbook/assets/image (14) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Since the length for the GUIDs are static, we can do those 2 commands and become a one-liner master just like this ->

```
Get-DomainGPO -Identity (Get-DomainOU -Identity StudentMachines).gplink.substring(11,(Get-DomainOU -Identity StudentMachines).gplink.length-72)
```

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
