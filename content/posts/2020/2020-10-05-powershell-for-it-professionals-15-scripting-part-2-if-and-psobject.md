---
title: 'PowerShell for IT Professionals [#15] – Scripting part 2 – IF and psobject'
author: Kamil

date: 2020-10-05T17:16:21+00:00
url: /powershell-for-it-professionals-15-scripting-part-2-if-and-psobject/
featuredImage: /wp-content/uploads/2020/10/Writing-first-script-pt2.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - course
  - custom
  - else
  - first script
  - hash table
  - if
  - object
  - powershell
  - ps1
  - psobject
  - script
  - video
  - write-host

---
{{<youtube FVXEhZAz_YU>}}

In this lesson we carry on writing the scripting by gathering requirements and putting them together as comments in code.

Then we will retrieve OS information with the help of WMI and display it on the screen with Write-Host.

Although using Write-Host is easy to use, it doesn&#8217;t really allow us to do very much e.g. we cannot export information to the CSV, therefore we change it and start using custom PSObject &#8211; that way our script will start returning information like a regular PowerShell command.

Finally, we will use an IF statement to decide whether the OS we are checking is a server.

## Reference

<a href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_if?view=powershell-7" target="_blank" rel="noreferrer noopener">IF statement</a>

<a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/powershell/scripting/learn/deep-dives/everything-about-pscustomobject?view=powershell-7" target="_blank">PS Custom Object</a>

<a rel="noreferrer noopener" href="https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_hash_tables?view=powershell-7" target="_blank">Hash table</a>

## Script

```powershell
$OS = Get-CimInstance -ClassName Win32_OperatingSystem
# OS Name
Write-Host "OS Name: $($OS.Caption)"
# Version
Write-Host "OS Version: $($OS.Version)"
# Install Date
Write-Host "OS Install date: $($OS.InstallDate)"
# Client/Server
IF ($os.Caption -like "*Server*") { $server = $true } else { $server = $false}
# IP: Interface, ip, gateway, dns
# Network: Can I ping gateway, DNS Server, website
# Installed roles – server only
# Last updates
# It would be nice if could run this remotely 

$props = [pscustomobject]@{
    'OSName' = $OS.Caption
    'OSVersion' = $OS.Version
    'OSInstallDate' = $OS.InstallDate
    'isServer' = $Server
}
$props
```
