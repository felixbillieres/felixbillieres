# 🦓 Common Web Vulnerabilities

Here are some examples are some of the most common vulnerability types for web applications that we should look for in a pentest, part of [OWASP Top 10](https://owasp.org/www-project-top-ten/) vulnerabilities for web applications.

### Broken Authentication/Access Control

`Broken Authentication` refers to vulnerabilities that allow attackers to bypass authentication functions. For example, this may allow an attacker to login without having a valid set of credentials or allow a normal user to become an administrator without having the privileges to do so.

### Malicious File Upload

If the web application has a file upload feature and does not properly validate the uploaded files, we may upload a malicious script (i.e., a `PHP` script), which will allow us to execute commands on the remote server.

### Command Injection

A web application may install a plugin of our choosing by executing an OS command that downloads that plugin, using the plugin name provided. If not properly filtered and sanitized, attackers may be able to inject another command to be executed alongside the originally intended command (i.e., as the plugin name), which allows them to directly execute commands on the back end server and gain control over it.

### SQL Injection (SQLi)

If the user input is not properly filtered and validated (as is the case with `Command Injections`), we may execute another SQL query alongside this query, which may eventually allow us to take control over the database and its hosting server.
