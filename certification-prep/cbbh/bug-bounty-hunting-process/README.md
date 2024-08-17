# 🔋 Bug Bounty Hunting Process

## Writing a Good Report

The essential elements of a good bug report are ->

| `Vulnerability Title`       | Including vulnerability type, affected domain/parameter/endpoint, impact etc.                                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `CWE & CVSS score`          | For communicating the characteristics and severity of the vulnerability.                                                                                             |
| `Vulnerability Description` | Better understanding of the vulnerability cause.                                                                                                                     |
| `Proof of Concept (POC)`    | Steps to reproduce exploiting the identified vulnerability clearly and concisely.                                                                                    |
| `Impact`                    | Elaborate more on what an attacker can achieve by fully exploiting the vulnerability. Business impact and maximum damage should be included in the impact statement. |
| `Remediation`               | Optional in bug bounty programs, but good to have.                                                                                                                   |

[CVSS v3.1 Calculator](https://www.first.org/cvss/calculator/3.1)  is used to identify the severity of an identified vulnerability.

Here are&#x20;

some good report examples selected by HackerOne:

* [SSRF in Exchange leads to ROOT access in all instances](https://hackerone.com/reports/341876)
* [Remote Code Execution in Slack desktop apps + bonus](https://hackerone.com/reports/783877)
* [Full name of other accounts exposed through NR API Explorer (another workaround of #476958)](https://hackerone.com/reports/520518)
* [A staff member with no permissions can edit Store Customer Email](https://hackerone.com/reports/980511)
* [XSS while logging in using Google](https://hackerone.com/reports/691611)
* [Cross-site Scripting (XSS) on HackerOne careers page](https://hackerone.com/reports/474656)
