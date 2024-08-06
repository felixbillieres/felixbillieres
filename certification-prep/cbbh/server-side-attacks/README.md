# 🦄 Server-side Attacks

Server-side attacks target the application or service provided by a server, whereas a client-side attack takes place at the client's machine, not the server itself.

four classes of server-side vulnerabilities:

* Server-Side Request Forgery (SSRF)
* Server-Side Template Injection (SSTI)
* Server-Side Includes (SSI) Injection
* eXtensible Stylesheet Language Transformations (XSLT) Server-Side Injection

{% tabs %}
{% tab title="Server-Side Request Forgery (SSRF)" %}
This attack occures when an attacker can manipulate a web application into sending unauthorized requests from the server.
{% endtab %}

{% tab title="Server-Side Template Injection (SSTI)" %}
Let's imagine templating engines and server-side templates to generate responses such as HTML content dynamically that is generated based on user input. Our goal as an attacker is to inject template code leak data or have an RCE
{% endtab %}

{% tab title="Server-Side Includes (SSI) Injection" %}
(SSI) can also be used to generate HTML responses dynamically. It can be used to include content that is present in all HTML pages to get leakage or RCE
{% endtab %}

{% tab title="XSLT Server-Side Injection" %}
XSLT is a language used to transform XML documents into other formats, such as HTML. Attackers exploit weaknesses in how XSLT transformations are handled, allowing them to inject and execute arbitrary code on the server.
{% endtab %}
{% endtabs %}
