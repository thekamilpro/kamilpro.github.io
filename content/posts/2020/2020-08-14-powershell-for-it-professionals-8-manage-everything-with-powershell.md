---
title: 'PowerShell for IT Professionals [#8] – Manage everything with PowerShell'
author: Kamil

date: 2020-08-14T16:31:00+00:00
url: /powershell-for-it-professionals-8-manage-everything-with-powershell/
featuredImage: /wp-content/uploads/2020/08/Modules.jpg
wp_last_modified_info:
  - 1st November 2020 @ 9:51 am
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - course
  - esxi
  - find-module
  - install-module
  - module
  - powershell
  - snmp
  - vmware

---
{{<youtube PRpn0B7nFN0>}}

## Exercises

  * Can you uninstall module that was installed with Install-Module? Confirm your answer with Get-Module -ListAvaiable
  * Can you update the installed module?
  * Perhaps, you&#8217;d like to install version 1.0.0.0 of SNMP module, how can you force Install-Module to do so?

## Install-Module error

If you encounter the issue with downloading modules, run this commandlet as a temporary workaround (it must be applied every time the shell is restarted): 

<pre class="wp-block-code"><code>&#91;Net.ServicePointManager]::SecurityProtocol = &#91;Net.SecurityProtocolType]::Tls12 </code></pre>

Source of solution, and also permanent solution can be found at: 

<a rel="noreferrer noopener" href="https://answers.microsoft.com/en-us/windows/forum/windows_7-performance/trying-to-install-program-using-powershell-and/4c3ac2b2-ebd4-4b2a-a673-e283827da143?auth=1" target="_blank">https://answers.microsoft.com/en-us/windows/forum/windows_7-performance/trying-to-install-program-using-powershell-and/4c3ac2b2-ebd4-4b2a-a673-e283827da143?auth=1</a>

## Commands used in video:

```powershell
Get-Command

Get-Command | Measure-Object

Get-Module -ListAvailable | Measure

#### About special shell
There's only one PowerShell. 
Let's review AD module for PowerShell.


#### SNMP
Let's use PS Gallery

Use SNMP module
Get-Command -Module SNMP

Invoke-SnmpWalk -IP 10.0.0.71 -Community public -OID ".1.3.6.1.2.1.1"

### VMWARE
Find-Module vmware*

#Install
Install-Module vmware.powercli

Get-command -Module vmware.powercli

Connect-ViServer

Get-VM

New-VM VM2

Get-VM VM2

Remove-VM VM2
```
