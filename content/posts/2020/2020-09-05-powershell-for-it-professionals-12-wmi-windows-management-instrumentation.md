---
title: 'PowerShell for IT Professionals [#12] – WMI – Windows Management Instrumentation'
author: Kamil

date: 2020-09-05T16:27:10+00:00
url: /powershell-for-it-professionals-12-wmi-windows-management-instrumentation/
featuredImage: /wp-content/uploads/2020/09/WMI.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - course
  - filter
  - get-ciminstance
  - get-wmiobject
  - gpo
  - powershell
  - training
  - windows management instrumentation
  - wmi

---
{{<youtube _NadlLhLldY>}}

In this lesson we learn how to use PowerShell to access WMI (Windows Management Instrumentation), so that we can gain often hidden or obscure information.

We will then use <a href="https://github.com/vinaypamnani/wmie2/releases" target="_blank" rel="noreferrer noopener">WMI Explorer</a> graphical tool to ease discovering all possible classes and instances that WMI provides. 

Finally, we will use WMI queries to add another level of granularity in GPO so that we can target very specific computers in it.

## Exercises

  * There&#8217;s a WMI class that lists all the installed software &#8211; find it and list all currently installed software with PowerShell. Feel free to use WMI explorer to help you discover the right class.
  * We often need to find where the software is installed. Use the class from previous step, to display these pieces of information: Name, version and installation directory
  * Open a Notepad application and leave it running. Discover methods of&nbsp; Win32_Process class. Find a method that allows you to kill process, and use it to close Notepad application.

## Notes

```powershell
# Slides
# Allows to retrieve and manage information of Windows
# Is Microsoft's implementation of Web-Based Enterprise Management (WBEM) and Common Information Model (CIM)
# Since Windows 2000
# Organised in namespaces e.g. Root\CIMv2 contians all information about Windows and hardware and than classes 
# It's bit obscure

# Let's look at an example
Get-CimInstance -Namespace root/CIMV2 -ClassName Win32_BIOS

# WMI Explorer to rescue - https://github.com/vinaypamnani/wmie2/releases

# Let's browse a bit here

Get-CimClass -Namespace root/CIMV2 -ClassName Win32_OperatingSystem

Get-CimInstance -Namespace root/CIMV2 -ClassName Win32_OperatingSystem

Get-CimInstance -Namespace root/CIMV2 -ClassName Win32_OperatingSystem| FL *

#Let's call a method
Get-CimClass -Namespace root/CIMV2 -ClassName Win32_OperatingSystem

Invoke-CimMethod -ClassName Win32_OperatingSystem -MethodName reboot

Get-CimInstance -Namespace root/CIMV2 -ClassName Win32_Service

Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct

# Filters and GPO

Get-CimInstance -Filter 'Version like "10.%" and ProductType="2"' -ClassName Win32_OperatingSystem

select * from Win32_OperatingSystem where Version like "10.%" and ProductType="3"

# CIM VS WMI

Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntiVirusProduct
Get-WmiObject -Namespace root/SecurityCenter2 -Class AntiVirusProduct
```
