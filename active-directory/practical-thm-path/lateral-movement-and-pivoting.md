# 🌋 Lateral Movement and Pivoting

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>



### Lateral movement with Psexec

* **Ports:** 445/TCP (SMB)
* **Required Group Memberships:** Administrators

`PsExec` uses Windows administrative shares (like `\\<RemoteComputer>\ADMIN$`) to copy an executable file (`PSEXESVC.exe`) to the remote machine. This service handles the execution of the desired command.

Once the `PSEXESVC.exe` service is running on the remote system, `PsExec` executes the specified command or application. It then redirects the output back to the local machine, allowing you to see the results as if you were running the command locally.

* Basic syntax:&#x20;

```
psexec \\<RemoteComputer> -u <Username> -p <Password> <Command>
```

* Example:&#x20;

```
psexec \\RemotePC -u Admin -p Password123 ipconfig
```
