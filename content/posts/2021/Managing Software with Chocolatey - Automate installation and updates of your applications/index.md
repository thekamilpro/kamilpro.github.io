---
title: Managing Software with Chocolatey - Automate installation and updates of your applications
author: Kamil
date: 2021-03-10
featuredimage: feat.jpg
categories:
  - Software
  - Deployment
  - Automation
  - Windows
tags:
  - cli
  - patching
  - software
  - installation
  - windows
  - sysadmin
  - automation
---

{{< youtube Tmh8c77tHQ8 >}}

Software management is not trivial task. Preparing for silent deployment is challenging, and even if you manage to install the application, how to keep it up to date? 

Chocolatey is package manager for Windows, that can fully automate lifecycle of your software.

In this video, I'll show you how to install Chocolatey, then how to find software, install it and finally keeping it up to date - all coming with free open-source license.

```ps1
# Install 
https://chocolatey.org/install

# Syntax
choco 
choco -?

# Search Software
choco find|search|list
choco find firefox
choco info firefox

# Install Software
choco install firefox
choco install notepadplusplus vlc -y

# Review what's installed
choco find --local-only
choco outdated
# Uninstall Software
choco uninstall vlc

# Upgrade Software
choco outdated
choco Upgrade
choco upgrade firefox
choco upgrade all
cup all -y
```