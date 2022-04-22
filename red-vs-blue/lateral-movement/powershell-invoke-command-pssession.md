# PowerShell (Invoke-Command, PSSession)

## Requirement

| Subject   | Value                         |
| --------- | ----------------------------- |
| Port      | 5985 (HTTP), 5986 (HTTPS)     |
| Authority | Administrators, Domain Admins |
| Service   | WinRM                         |

{% hint style="danger" %}
Only work if WinRM service has enabled on the target host and already config TrustedHosts to allow the source host to remote execute
{% endhint %}

Powershell Command to **check WinRM** service and **TrustedHosts config**

```powershell
# Enable WinRM on the target
Enable-PSRemoting -force
# Set to trust all hosts
Set-Item WSMan:\localhost\Client\TrustedHosts -Value * -Force
# Check trusted hosts
Get-Item WSMan:\localhost\Client\TrustedHosts
```

## <mark style="color:red;">RedTeam</mark>

### **Invoke-Command**

```powershell
# Check WinRM on the target host (IP: 10.0.77.130)
Test-NetConnection 10.0.77.130 -CommonTCPPort WINRM

# Remote Execute 'hostname' command to 'Wakanda-WrkStn' (IP: 10.0.77.130) with 'marvel\panther' account and 'WakandaForever!' password
$pass=(ConvertTo-SecureString "WakandaForever!" -AsPlainText -Force); $cred=(New-Object System.Management.Automation.PSCredential("marvel\panther", $pass)); Invoke-Command -ComputerName Wakanda-WrkStn -Credential $cred -ScriptBlock { cmd.exe /c hostname }
```

### **PSSession**

```powershell
# Create PSSession on 'Wakanda-WrkStn' host with 'marvel\panther' account and 'WakandaForever!' password
$pass=(ConvertTo-SecureString "WakandaForever!" -AsPlainText -Force); $cred=(New-Object System.Management.Automation.PSCredential("marvel\panther", $pass)); New-PSSession -ComputerName Wakanda-WrkStn -Credential $cred

 Id Name            ComputerName    ComputerType    State         ConfigurationName     Availability
 -- ----            ------------    ------------    -----         -----------------     ------------
 24 WinRM24         Wakanda-WrkStn  RemoteMachine   Opened        Microsoft.PowerShell     Available
 
# Enter to session id: 24
Enter-PSSession 24
```

## <mark style="color:blue;">BlueTeam</mark>

### Process tree (target host)

* wsmprovhost.exe
  * cmd.exe

### Event Logs (source host)

| Source                                         | Event ID                                               | Condition                                                                                                                                                                                     | Focus On                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ---------------------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx                                  | 4648: A logon was attempted using explicit credentials | <p>Target Server\Additional Information == HTTP*<br>AND<br>Process Information\Process Name == *powershell.exe</p>                                                                            | <p><strong>Target IP</strong><br><strong></strong>(Network Information\Network Address)<br><br><strong>Target Server Name</strong><br><strong></strong>(Target Server\Target Server Name)</p>                                                                                                                                                                                                                                                                                                                        |
| Security.evtx                                  | 4688: A new process has been created                   | (Process Information\New Process Name == \*powershell.exe AND Process Information\Process Command Line == \*-ComputerName\*) OR Process Information\Process Command Line == Enter-PSSession\* | <p><strong>Parent Process ID</strong><br><strong></strong>(Process Information\Creator Process ID)<br><br><strong>Parent Process</strong><br><strong></strong>(Process Information\Creator Process Name)<br><br><strong>Process ID</strong><br><strong></strong>(Process Information\New Process ID)<br><br><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name)<br><br><strong>Process Command Line</strong><br><strong></strong>(Process Information\Process Command Line)</p> |
| Microsoft-Windows-PowerShell%4Operational.evtx | 4104: PowerShell script block                          | -                                                                                                                                                                                             | <p><strong>Scriptblock</strong><br><strong></strong>(Creating Scriptblock text)</p>                                                                                                                                                                                                                                                                                                                                                                                                                                  |

### Event Logs (target host)

| Source        | Event ID                                    | Condition                                 | Focus On                                                                                                                                                                                                                                                  |
| ------------- | ------------------------------------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4624: An account was successfully logged on | Logon Type == 3                           | <p><strong>Source IP</strong><br><strong></strong>(Network Information\Source Network Address)<br><br><strong>Account Domain</strong><br><strong></strong>(New Logon\Account Domain)<br><br><strong>Account Name</strong><br>(New Logon\Account Name)</p> |
| Security.evtx | 4688: A new process has been created        | Creator Process Name == \*wsmprovhost.exe | <p><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name Process)<br><br><strong>Command Line</strong><br><strong></strong>(Information\Process Command Line)</p>                                                       |
