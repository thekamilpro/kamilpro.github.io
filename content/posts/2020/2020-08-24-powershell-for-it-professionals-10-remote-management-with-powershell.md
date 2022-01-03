---
title: 'PowerShell for IT Professionals [#10] – Remote management with PowerShell'
author: Kamil

date: 2020-08-24T10:00:00+00:00
url: /powershell-for-it-professionals-10-remote-management-with-powershell/
featuredImage: /wp-content/uploads/2020/08/Remoting.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - command
  - course
  - enter
  - invoke
  - IT
  - linux
  - powershell
  - psexec
  - pssesion
  - rdp
  - remote
  - remoting
  - ssh
  - sysadmin
  - telnet
  - ws-man

---
{{<youtube TB2-Y3zFUb0>}}

In this lesson we&#8217;re going to learn how to do one-to-one and one-to-many remote management with PowerShell. There&#8217;s no need for telnet, ssh or psexec as PowerShell has its own protocol that&#8217;s built in right into Windows. We will look at how to create interactive sessions and send commands to multiple servers at once.

## Exercises

<ol type="1">
  <li>
    With Enter-PSSesion, Remote to remote server and&nbsp; restart &#8220;Windows update&#8221; service
  </li>
  <li>
    With Invoke-Command find out the remote computers last boot up/startup time
  </li>
  <li>
    List all running processes on remote server without using Enter-PsSession or Invoke-Command
  </li>
  <li>
    If you have a Domain Controller available, use Invoke-Command to list all computers on that network (you can use * for filter), remember that you need to provide all necessary parameters with your command in the script block so command just runs and doesn&#8217;t prompt for any additional information.
  </li>
</ol>

## Notes

```powershell
Enable-Psremoting

Enter-PSSesion ps-svr1

Hostname
Get-Service
GIP

# I can even run commands that are not available on my source machine

Get-ADDomainController
Get-ADUser

Exit or Exit-PSSession

# Caution about double hoping

Invoke-Command

Invoke-Command -computerName ps-svr1 -command  { get-service}

# Invoke command executes commands on the remote comptuers and brings back the results

# Can you tell a difference?

Invoke-Command -computerName ps-svr1 -command  { get-service | Where {$_.status -eq 'stopped' } }

Invoke-Command -computerName ps-svr1 -command  { get-service } | Where {$_.status -eq 'stopped' }

# -ComputerName in the commandlet

Get-Service -ComputerName ps-svr1

Invoke-Command -computerName ps-svr1 -command  { get-service  }

# they look the same, but are they?

Get-Service -ComputerName ps-svr1 | GM

Invoke-Command -computerName ps-svr1 -command  { get-service  } | GM
```
