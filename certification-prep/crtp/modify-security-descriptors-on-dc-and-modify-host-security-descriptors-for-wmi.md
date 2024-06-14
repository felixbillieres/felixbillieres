# ⛑️ Modify security descriptors on DC & modify host security descriptors for WMI

Once we have administrative privileges on a machine, we can modify security descriptors of services to access the services without administrative privileges. the following command modifies the host security descriptors for WMI on the DC to allow studentx access to WMI

{% hint style="info" %}
WMI provides powerful administrative capabilities, such as querying system information, modifying system configurations, and executing scripts remotely. Granting `studentx` access to WMI on a DC effectively elevates the user's privileges,
{% endhint %}
