---
layout: post
published: true
title: How to Check if a Server Needs a Reboot
date: '2017-02-22'
---
## Using Powershell

If you’re trying to determine which of your servers require reboots, you’ll love this PowerShell script to check the status. It turns out that a simple way to identify **servers that are pending reboot** is to check the registry. This information is stored in the HKeyLocalMachine hive of the registry.
PowerShell is born and bred for working with the registry. Registry is one of the built-in PowerShell providers. There’s even already a PSDrive connected to that registry hive! You can browse around the registry just like you can the filesystem.

```
#Change to the registry drive. 
#Set-Location can also be invoked through its aliases - CD and SL
#Get-ChildItem can also be invoked through its aliases - Dir and LS
 
Set-Location HKLM:
Get-ChildItem
```
Wow! Super easy, right?
Now you just need to know where the “pending reboot” location is. There are a couple of places to check.

> HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired

Is where patches installed through automatic updates register the need to reboot.

> HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending

Is another place where pending reboots can be identified.

> HKLM\SYSTEM\CurrentControlSet\Control\Session Manager

Is yet another. Finally, there is Configuration Manager which, if present, can be queried via WMI.

I found a function that I really like to check all four locations. I’ll need to wrap it up with some parameters to check remote computers, but in general it was a great start. I’ve adapted the function to return $true on the first condition that satisfies, since I only care about whether the computer is pending a reboot, and not where the source of the reboot is comping from.

```
#Adapted from https://gist.github.com/altrive/5329377
#Based on <http://gallery.technet.microsoft.com/scriptcenter/Get-PendingReboot-Query-bdb79542>
function Test-PendingReboot
{
 if (Get-ChildItem "HKLM:\Software\Microsoft\Windows\CurrentVersion\Component Based Servicing\RebootPending" -EA Ignore) { return $true }
 if (Get-Item "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\WindowsUpdate\Auto Update\RebootRequired" -EA Ignore) { return $true }
 if (Get-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager" -Name PendingFileRenameOperations -EA Ignore) { return $true }
 try { 
   $util = [wmiclass]"\\.\root\ccm\clientsdk:CCM_ClientUtilities"
   $status = $util.DetermineIfRebootPending()
   if(($status -ne $null) -and $status.RebootPending){
     return $true
   }
 }catch{}
 
 return $false
}
```