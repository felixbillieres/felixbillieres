# 📻 ldapdomaindump

LDAPDomainDump is a tool used for Active Directory enumeration via LDAP (Lightweight Directory Access Protocol). It allows users to collect and parse information from an Active Directory domain, providing both human-readable and machine-readable output. The tool can be used for reconnaissance and security assessments in Active Directory environments.

<figure><img src="../../../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

LDAPDomainDump is a Python tool designed for Active Directory information gathering through LDAP. It enables users to enumerate essential details such as delegations, group policy objects (GPOs), groups, machines, password settings objects (PSOs), trusts, and users. The tool runs LDAP queries to extract and organize this information, providing insights into the Active Directory structure.

LDAPDomainDump is employed in security assessments and post-compromise activities to understand the configuration, users, and groups within an Active Directory domain. It aids in identifying potential security vulnerabilities, misconfigurations, and other relevant details for both defensive and offensive security purposes.

```
sudo ldapdomaindump ldaps://172.16.91.129 -u 'MARVEL\fcastle' -p Passw0rd
```

<figure><img src="../../../../.gitbook/assets/image (181).png" alt=""><figcaption></figcaption></figure>

and now you have plenty of informations about the domain and possibly clear text credentials if msiconfigured:

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1)   (6).png" alt=""><figcaption></figcaption></figure>
