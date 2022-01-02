---
title: 'PowerShell for IT Professionals [#9] – Setting up Active Directory'
author: Kamil
type: post
date: 2020-08-19T12:49:09+00:00
url: /powershell-for-it-professionals-9-setting-up-active-directory/
featured_image: /wp-content/uploads/2020/08/AD-Setup.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - active directory
  - AD
  - course
  - dcpromo
  - install-windowsfeature
  - powershell
  - youtube

---
{{<youtube oVvTe9VCJgM >}}

## Exercises

  * On domain controller, find module that allows to manage Active Directory
  * List all the Active Directory users
  * List members of &#8220;Enterprise Administrators&#8221;
  * Find the feature name for _Windows Server Backup_ and install it with PowerShell

## Commands

```powershell
### Server

# Check IP configuration
# Show steps in Server how to install

# Check hostname
Hostname

# Rename server
Rename-Computer ps-svr1

# Restart computer
Reboot-computer

# Get-WindowsFeature

# Install-WindowsFeature 
Install-WindowsFeature -Name AD-Domain-Services -IncludeManagementTools

Install-ADDSForest -DomainName posh.pri

### Client

# Check IP
GIP

Hostname

Add-Computer -DomainName posh.pri
```