# ➿ Information Gathering - Web Edition

We have to think about information gathering as the preparatory phase before delving into deeper analysis and potential exploitation.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The goal is to correctly:

* `Identifying Assets`
* `Discovering Hidden Information`
* `Analysing the Attack Surface`
* `Gathering Intelligence`

There are 2 main types of reconnaissance, active and passive&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

When we start with passive recon, WHOIS is a nice way to start, it's a query and response protocol designed to access databases that store information about registered internet resources. It provide details about IP address blocks and autonomous systems.

Each WHOIS record typically contains the following information:

* `Domain Name`: The domain name itself (e.g., example.com)
* `Registrar`: The company where the domain was registered (e.g., GoDaddy, Namecheap)
* `Registrant Contact`: The person or organization that registered the domain.
* `Administrative Contact`: The person responsible for managing the domain.
* `Technical Contact`: The person handling technical issues related to the domain.
* `Creation and Expiration Dates`: When the domain was registered and when it's set to expire.
* `Name Servers`: Servers that translate the domain name into an IP address.

It's good for initial recon because it often reveal the names, email addresses, and phone numbers of individuals responsible for managing the domain. It gives name servers and IP addresses and can provide clues about the target's network infrastructure and it's good to note [WhoisFreaks](https://whoisfreaks.com/) that can reveal changes in ownership, contact information, or technical details over time.
