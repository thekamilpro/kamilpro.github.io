---
title: "PowerShell Error Handling - One error at the time please"
author: Kamil
date: 2023-01-21
resources:
  - name: "featured-image"
    src: "featured-image.jpeg"
categories: ["PowerShell", "Video", "Error", "Script"]
tags: [ "PowerShell", "pwsh", "Error", "Script", "Try", "Catch", "Finally"]
---

{{< youtube -rQTBuaIVS8 >}}

Code fails - whether because of reasons out of our control, or because we didn't consider the situation it might fail. Expired credentials, timeout connections, lack of permissions go genuine software bug - each of these are potential problem which might fall our code over. And if we don't handle the error, our script will simply throw and terminate. It's certainly not desired state.

In this video I'll show you different ways PowerShell can error, how to debug such errors and how to handle them - so that we are in control.

[Code](https://github.com/thekamilpro/About-PowerShell/tree/main/PowerShell%20Error%20Handling)