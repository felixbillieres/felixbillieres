# 🕸️ Web Application

`assetfinder` is a command-line tool written in Go by Tom Hudson (tomnomnom). It is designed to discover and enumerate subdomains associated with a given domain. Subdomains can be valuable information for security assessments, bug bounty hunting, and other cybersecurity purposes.

The tool works by querying various sources such as public DNS databases and certificate transparency logs to identify subdomains associated with the target domain. It can help security professionals and researchers identify potential entry points for further investigation.

Here is a basic example of how to use `assetfinder`:

```bash
assetfinder example.com
```

Replace `example.com` with the target domain for which you want to discover subdomains. The tool will then query different sources and print the identified subdomains.

let's take for example tesla.com

```
┌──(root㉿kali)-[/home/kali/Desktop/PNPT]
└─# assetfinder tesla.com >> tesla_subdomain.txt
                                                                                                                                                                                                                                          
┌──(root㉿kali)-[/home/kali/Desktop/PNPT]
└─# cat tesla_subdomain.txt| wc -l
584

┌──(root㉿kali)-[/home/kali/Desktop/PNPT]
└─# assetfinder --subs-only tesla.com    
```
