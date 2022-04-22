# WMI/WMIC

## Requirement

| Subject   | Value                         |
| --------- | ----------------------------- |
| Port      | 135, 445                      |
| Authority | Administrators, Domain Admins |
| Service   | WMI, RPC                      |

## <mark style="color:red;">RedTeam</mark>

### **wmic**

```powershell
# Remote Execute 'hostname' command to '10.0.77.130' with 'marvel\panther' account and 'WakandaForever!' password
wmic /node:10.0.77.130 /user:marvel\panther /password:WakandaForever! process call create "cmd.exe /c hostname"
```

## <mark style="color:blue;">BlueTeam</mark>

### Process tree (target host)

* wmiprvse.exe
  * cmd.exe

### Event Logs (source host)

| Source        | Event ID                                               | Condition                                                                                                             | Focus On                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4648: A logon was attempted using explicit credentials | <p>Process Target Server\Additional Information == host*<br>AND <br>Process Information\Process Name == *WMIC.exe</p> | <p><strong>Target IP</strong><br><strong></strong>(Network Information\Network Address)<br><br><strong>Target Server Name</strong><br><strong></strong>(Target Server\Target Server Name)</p>                                                                                                                                                                                                                                                                                                                        |
| Security.evtx | 4688: A new process has been created                   | Process Information\New Process Name == \*wmic.exe AND Process Information\Process Command Line == \*/node\*          | <p><strong>Parent Process ID</strong><br><strong></strong>(Process Information\Creator Process ID)<br><br><strong>Parent Process</strong><br><strong></strong>(Process Information\Creator Process Name)<br><br><strong>Process ID</strong><br><strong></strong>(Process Information\New Process ID)<br><br><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name)<br><br><strong>Process Command Line</strong><br><strong></strong>(Process Information\Process Command Line)</p> |

### Event Logs (target host)

| Source        | Event ID                                    | Condition                              | Focus On                                                                                                                                                                                                                                                  |
| ------------- | ------------------------------------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4624: An account was successfully logged on | Logon Type == 3                        | <p><strong>Source IP</strong><br><strong></strong>(Network Information\Source Network Address)<br><br><strong>Account Domain</strong><br><strong></strong>(New Logon\Account Domain)<br><br><strong>Account Name</strong><br>(New Logon\Account Name)</p> |
| Security.evtx | 4688: A new process has been created        | Creator Process Name == \*wmiprvse.exe | <p><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name Process)<br><br><strong>Command Line</strong><br><strong></strong>(Information\Process Command Line)</p>                                                       |
