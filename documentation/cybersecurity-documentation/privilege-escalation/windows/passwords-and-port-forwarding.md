---
description: https://app.hackthebox.com/machines/Chatterbox
---

# ↔️ Passwords and Port Forwarding

## Chatterbox

The nmap scan revealed a port 9256 opened that is aChat (a software that enables you to chat on your local network.)

So obviously we run a searchsploit on it:

```
searchsploit achat 
```

<figure><img src="../../../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

```
searchsploit -m 36025
```

we download the exploit, and then we see a buffer overflow, let's modify what we need to use it correctly:

