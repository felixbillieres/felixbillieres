# 🛷 Authentication Bypasses

The most straightforward way of bypassing authentication checks is to request the protected resource directly from an unauthenticated context. An unauthenticated attacker can access protected information if the web application does not properly verify that the request is authenticated.

if the app is vulnerable, let's imagine the application redirects users to the `/admin.php` endpoint after successful authentication, we can directly request for /admin.php to bypass any auth user protection

## Authentication Bypass via Direct Access

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

However, if  the browser follows the redirect and displays the login prompt instead of admin page. We can trick the browser into displaying the admin page by intercepting the response and changing the status code from `302` to `200`

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

and after we forward, we can edit the request to display the wanted content from `302 Found` to `200 OK`:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Apply what you learned in this section to bypass authentication to obtain the flag.**_

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12) (1) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Authentication Bypass via Parameter Modification

This type of vulnerability is closely related to authorization issues such as `Insecure Direct Object Reference (IDOR)`

Let's say  user `htb-stdnt` is redirected to `/admin.php?user_id=183` after login

<figure><img src="../../../.gitbook/assets/image (13) (1) (1).png" alt=""><figcaption></figcaption></figure>

If we remove the `user_id` parameter, we are redirected tp /index.php login page even tho or php cookie is still valid so we can guess that user id is related to auth and we can authenticate entirely by accessing the URL `/admin.php?user_id=183` directly

<figure><img src="../../../.gitbook/assets/image (14) (1) (1).png" alt=""><figcaption></figcaption></figure>

_**Apply what you learned in this section to bypass authentication to obtain the flag.**_

```
ffuf -w num.txt -u http://94.237.56.194:56707/admin.php?user_id=FUZZ -X GET -b "PHPSESSID=roa555l3drvufld0ru41ib7jgs" -fr "Could not load admin data"
```

<figure><img src="../../../.gitbook/assets/image (15) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (16) (3).png" alt=""><figcaption></figcaption></figure>
