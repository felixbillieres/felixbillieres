# 🤐 GPP / cPassword Attacks

Group Policy Preferences (GPP) allowed administrators to create domain policies with embedded credentials

<figure><img src="../../../../.gitbook/assets/image (10) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

These policies allowed them to set local accounts, and embed credentials for various purposes that may otherwise require an embedded password in a script. So when a new Group Policy Preference (GPP) is generated, a xml file (generally Groups.xml) with the configuration data, including any passwords associated with the GPP, is created in the SYSVOL share which are folders on domain controllers accessible and readable to all authenticated domain users.

It was patched a long time ago but it's still good to try because if the older files were never deleted than the attack could still work

<figure><img src="../../../../.gitbook/assets/image (12) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (11) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Nice CTF on the subject :

{% content-ref url="../../../../capture-the-flags/active-directory-and-networks/active.md" %}
[active.md](../../../../capture-the-flags/active-directory-and-networks/active.md)
{% endcontent-ref %}

We can also use metasploit:

<figure><img src="../../../../.gitbook/assets/image (13) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://blog.zsec.uk/path2da-pt1/" %}

{% embed url="https://xedex.gitbook.io/internalpentest/internal-pentest/active-directory/post-compromise-attacks/gpp-attacks" %}
