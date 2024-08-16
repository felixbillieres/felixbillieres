# Domain Fuzzing

Browsers only understand how to go to IPs, and if we provide them with a URL, they try to map the URL to an IP by looking into the local `/etc/hosts` file and the public DNS `Domain Name System`.

To quickly add our new DNS to our host file we would need to type the following command ->

```shell-session
sudo sh -c 'echo "94.237.59.199  academy.htb" >> /etc/hosts'
```

## Sub-domain Fuzzing

A sub-domain is any website underlying another domain. For example, `https://photos.google.com` is the `photos` sub-domain of `google.com`.

Here is the wordlist and the syntax for subdomain enumeration ->

```
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

But we need to remember that sometimes there are no `public` sub-domains under a DNS

_**Try running a sub-domain fuzzing test on 'inlanefreight.com' to find a customer sub-domain portal. What is the full domain of it?**_

```
ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```

<figure><img src="../../../.gitbook/assets/image (1391).png" alt=""><figcaption></figcaption></figure>

## Vhost Fuzzing

The key difference between VHosts and sub-domains is that a VHost is basically a 'sub-domain' served on the same server and has the same IP, such that a single IP could be serving two or more different websites.

To scan for VHosts, without manually adding the entire wordlist to our `/etc/hosts`, we will be fuzzing HTTP headers, specifically the `Host:` header. To do that, we can use the `-H` flag

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```

With `ffuf`, we can get many responses with code `200` so we need to filter out sometimes -> if the response size of the incorrect results, let's say `900`we can filter it out with `-fs 900`.

```shell-session
ffuf -w /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```

_**Try running a VHost fuzzing scan on 'academy.htb', and see what other VHosts you get. What other VHosts did you get?**_

<figure><img src="../../../.gitbook/assets/image (1392).png" alt=""><figcaption></figcaption></figure>

```
 ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:35902/ -H 'Host: FUZZ.academy.htb' -fs 986
```

<figure><img src="../../../.gitbook/assets/image (1393).png" alt=""><figcaption></figcaption></figure>
