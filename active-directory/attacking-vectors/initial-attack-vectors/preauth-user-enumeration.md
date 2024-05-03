# 🚘 PreAuth User Enumeration

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can send a request for a TGT --- without a pre-authentication hash --- to the Kerberos Key Distribution Center (KDC) with specific usernames in the request. If the username is **valid**, the KDC will prompt us for pre-authentication if required, or return a TGT if pre-authentication is not required. If the username is **invalid**, the KDC will respond with `PRINCIPAL UNKNOWN`

While doing pre auth, always good to start with Kerbrute:

{% content-ref url="../../../interacting-with-protocols-and-tools/tools/kerbrute.md" %}
[kerbrute.md](../../../interacting-with-protocols-and-tools/tools/kerbrute.md)
{% endcontent-ref %}

