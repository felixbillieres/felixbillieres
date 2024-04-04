# 🌟 Attacking O365

## Attacking Login Forms on O365: A Documentation

### Overview

The majority of targets using Active Directory (AD) often integrate with Microsoft O365 due to its seamless compatibility with AD and domain environments. One tool commonly employed in this context is Trevorspray, available at [blacklanternsecurity/TREVORspray](https://github.com/blacklanternsecurity/TREVORspray). This tool is utilized to identify valid emails by spraying them against the O365 login forms.

### Key Steps in the Attack

1. **Email Validation:**
   * Trevorspray is employed to identify valid email addresses by spraying them on O365 login forms.

<figure><img src="../../../../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

* **Multi-Factor Authentication (MFA) Detection:**
  * The tool also aids in determining if MFA is in use for a given email address.
* **Consideration of Lockout Policies:**
  * It is crucial to be aware of the client's lockout policy to avoid unnecessary account lockouts. This step is essential to prevent causing additional workload for the client.

### Good documentation:

Very good documentation on the README of the GitHub for more in depth subjects: [https://github.com/blacklanternsecurity/TREVORspray/blob/trevorspray-v2/README.md](https://github.com/blacklanternsecurity/TREVORspray/blob/trevorspray-v2/README.md)

also: [https://github.com/mdsecactivebreach/o365-attack-toolkit](https://github.com/mdsecactivebreach/o365-attack-toolkit)
