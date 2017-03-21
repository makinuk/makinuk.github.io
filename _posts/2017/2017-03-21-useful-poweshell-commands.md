---
layout: post
title: "Useful poweshell commands"
published: true
date: 2017-03-21 09:00:00
categories: [PowerShell]
excerpt: | 
        PowerShell is an automation platform and scripting language for Windows and Windows Server that allows you to simplify the management of your systems.
        kill process,find process, kill working more than 5 minutes php files. 
        Unlike other text-based shells, PowerShell harnesses the power of the .NET Framework, providing rich objects and a massive set of built-in functionality for taking control of your Windows environments.
---

# What is PowerShell?

PowerShell is an automation platform and scripting language for Windows and Windows Server that allows you to simplify the management of your systems. 
Unlike other text-based shells, PowerShell harnesses the power of the .NET Framework, providing rich objects and a massive set of built-in functionality for taking control of your Windows environments.
 
 
[Official Documentation](https://msdn.microsoft.com/en-us/powershell/reference/5.1/microsoft.powershell.management/microsoft.powershell.management)
 
## Find working process


Find working php files : 

```
Get-WmiObject win32_Process | where-object {$_.CommandLine -like "*php*"} | Select-Object ProcessId,CommandLine,ProcessName,CreationDate | Format-List
```
Output : 
```
ProcessId    : 7340
CommandLine  : "C:\Program Files (x86)\JetBrains\PhpStorm 2016.2.2\bin\PhpStorm64.exe"
ProcessName  : PhpStorm64.exe
CreationDate : 20170320114715.853150+180
```

Find working php files and disylay on table list
```
Get-WmiObject win32_Process | where-object {$_.CommandLine -like "*php*"} | Select-Object ProcessId,CommandLine,ProcessName,CreationDate | Format-Table -wrap
```

Output : 
```
ProcessId CommandLine                                                                                                                                                            ProcessName      CreationDate
--------- -----------                                                                                                                                                            -----------      ------------
     7340 "C:\Program Files (x86)\JetBrains\PhpStorm 2016.2.2\bin\PhpStorm64.exe"                                                                                                PhpStorm64.exe   20170320114715.8
                                                                                                                                                                                                  53150+180
     7772 "C:\Program Files (x86)\JetBrains\PhpStorm 2016.2.2\bin\fsnotifier64.exe"                                                                                              fsnotifier64.exe 20170320114736.5
```

## Kill process working more than 15 min
 ```
 (Get-WmiObject win32_Process | where-object {$_.CommandLine -like '*download-file.php*' -and $_.ConvertToDateTime($_.CreationDate) -lt (Get-Date).AddMinutes(-15)}).Invokemethod("terminate", $null)
 ```