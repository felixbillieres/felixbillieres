# 🌭 Kerbrute

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Kerbrute has three main commands:

* **bruteuser** - Bruteforce a single user's password from a wordlist
* **bruteforce** - Read username:password combos from a file or stdin and test them
* **passwordspray** - Test a single password against a list of users
* **userenum** - Enumerate valid domain usernames via Kerberos

A domain (`-d`) or a domain controller (`--dc`) must be specified. If a Domain Controller is not given the KDC will be looked up via DNS.

Output is logged to stdout, but a log file can be specified with `-o`

Kerbrute has a `--safe` option. When this option is enabled, if an account comes back as locked out, it will abort all threads to stop locking out any other accounts.

### User Enumeration

Kerbrute sends TGT requests with no pre-authentication. If the KDC responds with a `PRINCIPAL UNKNOWN` error, the username does not exist. However, if the KDC prompts for pre-authentication, we know the username exists and we move on.

This generates a Windows event ID [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) if Kerberos logging is enabled.

```
./kerbrute_linux_amd64 userenum -d lab.ropnop.com usernames.txt
```

### Password Spray

Kerbrute will perform a horizontal brute force attack against a list of domain users. This is useful for testing one or two common passwords when you have a large list of users. WARNING: this does will increment the failed login count and lock out accounts. This will generate both event IDs [4768 - A Kerberos authentication ticket (TGT) was requested](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) and [4771 - Kerberos pre-authentication failed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4771)

```
./kerbrute_linux_amd64 passwordspray -d lab.ropnop.com domain_users.txt PasswordToSpray
```

### Brute User

If no lockout policy -> traditional bruteforce account against a username

generate both event IDs [4768 - A Kerberos authentication ticket (TGT) was requested](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) and [4771 - Kerberos pre-authentication failed](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4771)

```
./kerbrute_linux_amd64 bruteuser -d lab.ropnop.com passwords.list username
```
