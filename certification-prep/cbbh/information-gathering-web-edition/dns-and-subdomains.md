# DNS & Subdomains

## DNS

DNS translates human-readable domain names (like `www.example.com`) into the numerical IP addresses (like `192.0.2.1`) that computers use to communicate.

Here is how it works more in depth:

1. `Your Computer Asks for Directions (DNS Query)`
2. `The DNS Resolver Checks its Map (Recursive Lookup)`
3. `Root Name Server Points the Way`
4. `TLD Name Server Narrows It Down`
5. `Authoritative Name Server Delivers the Address`
6. `The DNS Resolver Returns the Information`
7. `Your Computer Connects`

The `hosts` file is a simple text file used to map hostnames to IP addresses, providing a manual method of domain name resolution that bypasses the DNS process. While DNS automates the translation of domain names to IP addresses, the `hosts` file allows for direct, local overrides.

### The Domain Information Groper

The `dig` command (`Domain Information Groper`) is used for querying DNS servers and retrieving types of DNS records.

| Command                         | Description                                                                                                                                                                                          |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `dig domain.com`                | Performs a default A record lookup for the domain.                                                                                                                                                   |
| `dig domain.com A`              | Retrieves the IPv4 address (A record) associated with the domain.                                                                                                                                    |
| `dig domain.com AAAA`           | Retrieves the IPv6 address (AAAA record) associated with the domain.                                                                                                                                 |
| `dig domain.com MX`             | Finds the mail servers (MX records) responsible for the domain.                                                                                                                                      |
| `dig domain.com NS`             | Identifies the authoritative name servers for the domain.                                                                                                                                            |
| `dig domain.com TXT`            | Retrieves any TXT records associated with the domain.                                                                                                                                                |
| `dig domain.com CNAME`          | Retrieves the canonical name (CNAME) record for the domain.                                                                                                                                          |
| `dig domain.com SOA`            | Retrieves the start of authority (SOA) record for the domain.                                                                                                                                        |
| `dig @1.1.1.1 domain.com`       | Specifies a specific name server to query; in this case 1.1.1.1                                                                                                                                      |
| `dig +trace domain.com`         | Shows the full path of DNS resolution.                                                                                                                                                               |
| `dig -x 192.168.1.1`            | Performs a reverse lookup on the IP address 192.168.1.1 to find the associated host name. You may need to specify a name server.                                                                     |
| `dig +short domain.com`         | Provides a short, concise answer to the query.                                                                                                                                                       |
| `dig +noall +answer domain.com` | Displays only the answer section of the query output.                                                                                                                                                |
| `dig domain.com ANY`            | Retrieves all available DNS records for the domain (Note: Many DNS servers ignore `ANY` queries to reduce load and prevent abuse, as per [RFC 8482](https://datatracker.ietf.org/doc/html/rfc8482)). |

## Subdomains

subdomains are extensions of the main domain, often created to organise and separate different sections or functionalities of a website. For instance, a company might use `blog.example.com` for its blog, `shop.example.com` for its online store, or `mail.example.com` for its email services.

It's useful to enumerate those to find hidden login portals, admin panels, information disclosure, internal data etc..

There is active subdomain enumertion like `brute-force enumeration`, which involves systematically testing a list of potential subdomain names against the target domain. Tools like `dnsenum`, `ffuf`, and `gobuster`

Or there is passive subdomain enumeration with `Certificate Transparency (CT) logs`, public repositories of SSL/TLS certificates or google dorking (e.g., `site:`)

### Subdomain Bruteforcing

Here are the most well known tools for subdomain bruteforcing:

| Tool                                                    | Description                                                                                                                     |
| ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| [dnsenum](https://github.com/fwaeytens/dnsenum)         | Comprehensive DNS enumeration tool that supports dictionary and brute-force attacks for discovering subdomains.                 |
| [fierce](https://github.com/mschwager/fierce)           | User-friendly tool for recursive subdomain discovery, featuring wildcard detection and an easy-to-use interface.                |
| [dnsrecon](https://github.com/darkoperator/dnsrecon)    | Versatile tool that combines multiple DNS reconnaissance techniques and offers customisable output formats.                     |
| [amass](https://github.com/owasp-amass/amass)           | Actively maintained tool focused on subdomain discovery, known for its integration with other tools and extensive data sources. |
| [assetfinder](https://github.com/tomnomnom/assetfinder) | Simple yet effective tool for finding subdomains using various techniques, ideal for quick and lightweight scans.               |
| [puredns](https://github.com/d3mondev/puredns)          | Powerful and flexible DNS brute-forcing tool, capable of resolving and filtering results effectively.                           |

#### DNSEnum

Here is how the tool is used:

```
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -r
```

And it would give out something like that:

```shell-session
[...]
inlanefreight.com.                       300      IN    A        134.209.24.248

[...]

Brute forcing with /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt:
_______________________________________________________________________________________

www.inlanefreight.com.                   300      IN    A        134.209.24.248
support.inlanefreight.com.               300      IN    A        134.209.24.248
[...]
```

## DNS Zone Transfers

A DNS zone transfer is essentially a wholesale copy of all DNS records within a zone (a domain and its subdomains) from one name server to another. if not adequately secured, unauthorised parties can download the entire zone file, revealing a complete list of subdomains, their associated IP addresses, and other sensitive DNS data.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* **Request Initiation:** The secondary DNS server sends a zone transfer request (AXFR) to the primary server.
* **SOA Record Transfer:** The primary server responds with its Start of Authority (SOA) record, including the serial number for version control.
* **Records Transmission:** The primary server transfers all DNS records (e.g., A, AAAA, MX, CNAME, NS) to the secondary server.
* **Completion Signal:** The primary server signals the end of the transfer once all records are sent.
* **Acknowledgement:** The secondary server confirms receipt and processing of the data, completing the zone transfer.

**Exploiting Zone Transfers**
