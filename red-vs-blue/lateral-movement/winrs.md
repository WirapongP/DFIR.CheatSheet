# WinRS

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

### **winrs**

```powershell
# Remote Execute 'hostname' command to 'Wakanda-WrkStn' (IP: 10.0.77.130) with 'marvel\panther' account and 'WakandaForever!' password
winrs -r:Wakanda-WrkStn -u:marvel\panther -p:WakandaForever! "cmd.exe /c hostname"
```

## <mark style="color:blue;">BlueTeam</mark>

### Process tree (target host)

* winrshost.exe
  * cmd.exe

### Event Logs (source host)

| Source        | Event ID                                               | Condition                                                                                                                | Focus On                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4648: A logon was attempted using explicit credentials | Process Information\Process Name == \*winrs.exe                                                                          | <p><strong>Target IP</strong><br><strong></strong>(Network Information\Network Address)<br><br><strong>Target Server Name</strong><br><strong></strong>(Target Server\Target Server Name)</p>                                                                                                                                                                                                                                                                                                                        |
| Security.evtx | 4688: A new process has been created                   | <p>Process Information\New Process Name == *winrs.exe <br>AND <br>Process Information\Process Command Line == \*-r\*</p> | <p><strong>Parent Process ID</strong><br><strong></strong>(Process Information\Creator Process ID)<br><br><strong>Parent Process</strong><br><strong></strong>(Process Information\Creator Process Name)<br><br><strong>Process ID</strong><br><strong></strong>(Process Information\New Process ID)<br><br><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name)<br><br><strong>Process Command Line</strong><br><strong></strong>(Process Information\Process Command Line)</p> |

### Event Logs (target host)

| Source        | Event ID                                    | Condition                               | Focus On                                                                                                                                                                                                                                                  |
| ------------- | ------------------------------------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4624: An account was successfully logged on | Logon Type == 3                         | <p><strong>Source IP</strong><br><strong></strong>(Network Information\Source Network Address)<br><br><strong>Account Domain</strong><br><strong></strong>(New Logon\Account Domain)<br><br><strong>Account Name</strong><br>(New Logon\Account Name)</p> |
| Security.evtx | 4688: A new process has been created        | Creator Process Name == \*winrshost.exe | <p><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name Process)<br><br><strong>Command Line</strong><br><strong></strong>(Information\Process Command Line)</p>                                                       |
