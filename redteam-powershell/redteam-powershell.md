# Basic Powershell

### Basic Commands

Get help for any command

```powershell
#Get help for Get-Process Command
Get-Help Get-Process -Full
```

List all commands

```powershell
#List all commands
Get-Command
#List all commands related to Microsoft Defender
Get-Command -Name *MpThreat*
```

Return all named properties assosicate with the object

```powershell
#Return all named properties assosicate with processes
Get-Process | Format-List *
```

Show all aliases

```powershell
Get-Alias
```

### Module

List all imported modules

```powershell
Get-Module
```

List all available modules

```powershell
Get-Modules -ListAvailable
```

Import a module

```powershell
#Import modules from devil.psm1 file
Import-Module .\devil.psm1
```

Get the commands of specific module

```powershell
#Get commands from devil module
Get-Command -Module Devil
```

### File

Search for files that contain a specific keyword in thier contents

```powershell
#Search for txt files that contain abc* at C:\Users\admin\Downloads
Select-String -Path C:\Users\admin\Downloads\*.txt -Pattern abc*
```

Display full contents

```powershell
#Display full contents of the test.txt file
Get-Contents C:\Users\admin\Downloads\test.txt
```

### Sort, Uniq and Select Object

Sort and uniq the object

```powershell
#Sort and uniq object from Get-Process command
Get-Process | Sort-Object -Unique
```

Select the specific object

```powershell
#Select only ProcessName object from Get-Procrss command
Get-Process | Select-Object ProcessName 
```

Export result to csv

```powershell
#Export Process list to csv
Get-Process | Export-Csv process-list.csv
```

### Registry

List the contents of a registry hive

```powershell
#List the contents of the autorun registry
ls HKLM:\Software\Microsoft\Windows\CurrentVersion\Run
```

Access Registry Hive

```powerquery
#Access autorun registry hive
cd HKLM:\Software\Microsoft\Windows\CurrentVersion\Run
```
