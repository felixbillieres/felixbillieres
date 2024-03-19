# 🍃 Passback Attacks

Passback attacks refer to a category of cybersecurity threats where attackers gain unauthorized access to sensitive information, particularly credentials stored on devices such as printers, using protocols like LDAP or SMB. These attacks pose a significant risk as they exploit weak security practices related to credential storage.

<figure><img src="../../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

### **Introducing the Pass-Back Attack**

The stored LDAP credentials are usually located on the network settings tab in the online configuration of the MFP and can typically be accessed via the Embedded Web Service (EWS). If you can reach the EWS and modify the LDAP server field by replacing the legitimate LDAP server with your malicious LDAP server, then the next time an LDAP query is conducted from the MFP, it will attempt to authenticate to your LDAP server using the configured credentials or the user-supplied credentials.&#x20;

### **Accessing the EWS**

Most MFPs ship with a set of default administrative credentials to access the EWS. These credentials are usually located in the Administrator Guide of the MFP in question and are a good place to start for initial access:

VendorUsernamePasswordRicohadminblankHPadminadmin or blankCanonADMINcanonEpsonEPSONWEBadmin

Another way to potentially access the EWS is through the Printer Exploitation Toolkit (PRET) and Praeda. Both tools are capable of Information Disclosure and Code Execution. If you are looking to utilize the tools for the first time, here are a few resources to help you get started:

* • [https://github.com/RUB-NDS/PRET](https://github.com/RUB-NDS/PRET)&#x20;
* • [https://github.com/percx/Praeda](https://github.com/percx/Praeda)&#x20;
* • [http://www.hacking-printers.net/wiki/index.php/Printer\_Security\_Testing\_Cheat\_Sheet](https://www.hacking-printers.net/wiki/index.php/Printer\_Security\_Testing\_Cheat\_Sheet)&#x20;

### Common Credential Storage Protocols

#### 2.1 LDAP (Lightweight Directory Access Protocol)

LDAP is a widely used protocol for accessing and maintaining directory services. In the context of printers, LDAP is often used for storing user credentials and managing access to printing functionalities.

<figure><img src="../../../../.gitbook/assets/image (183).png" alt=""><figcaption></figcaption></figure>

#### 2.2 SMTP (Simple Mail Transfer Protocol)  SMTP is a protocol used for sending emails. Printers may use SMTP for storing credentials and facilitating communication with email servers.

<figure><img src="../../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

Very good documentation and most of the screenshots: [https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack](https://www.mindpointgroup.com/blog/how-to-hack-through-a-pass-back-attack)
