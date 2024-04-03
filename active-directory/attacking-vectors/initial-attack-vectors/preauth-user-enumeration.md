# 🚘 PreAuth User Enumeration

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We can send a request for a TGT --- without a pre-authentication hash --- to the Kerberos Key Distribution Center (KDC) with specific usernames in the request. If the username is **valid**, the KDC will prompt us for pre-authentication if required, or return a TGT if pre-authentication is not required. If the username is **invalid**, the KDC will respond with `PRINCIPAL UNKNOWN`

Kerbrute has three main commands:

* **bruteuser** - Bruteforce a single user's password from a wordlist
* **bruteforce** - Read username:password combos from a file or stdin and test them
* **passwordspray** - Test a single password against a list of users
* **userenum** - Enumerate valid domain usernames via Kerberos

A domain (`-d`) or a domain controller (`--dc`) must be specified. If a Domain Controller is not given the KDC will be looked up via DNS.

Output is logged to stdout, but a log file can be specified with `-o`

Kerbrute has a `--safe` option. When this option is enabled, if an account comes back as locked out, it will abort all threads to stop locking out any other accounts.
