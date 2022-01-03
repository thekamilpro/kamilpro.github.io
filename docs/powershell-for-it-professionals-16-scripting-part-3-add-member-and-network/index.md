# PowerShell for IT Professionals [#16] – Scripting part 3 – Add-Member and network

{{<youtube rA8CWs2Of2Y>}}

In this lesson we are taking our script further by adding details about IP configuration and last installed hotfixes.

But it turns out, the networking information is not that shallow as it seems at the first glance &#8211; thus I&#8217;ll show you how to retrieve the information from the configuration.

We will also look on how to reuse our already created object so that we don&#8217;t need to duplicate code.

And finally, we will use Add-Member to add more properties to the custom object.

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
$Network = Get-NetAdapter | Where-Object {$_.Status -eq 'Up'} | Get-NetIPConfiguration
# Last updates
$Updates = Get-HotFix | Sort InstalledOn -Descending | Select-Object -First 5
# It would be nice if could run this remotely 

$props = [pscustomobject]@{
    'OSName' = $OS.Caption
    'OSVersion' = $OS.Version
    'OSInstallDate' = $OS.InstallDate
    'isServer' = $Server
    'IPv4Address' = $Network.IPv4Address.IPAddress
    'Gateway' = $Network.IPv4DefaultGateway.nexthop
    'DNS' = $Network.DNSServer | Where-Object {$_.AddressFamily -eq '2'} | Select-Object ServerADdresses -ExpandProperty ServerAddresses
    'Updates' = $Updates.HotfixID | Out-String
}
# Network: Can I ping gateway, DNS Server, website
$pingGateway = Test-NetConnection -ComputerName $props.Gateway -InformationLevel Quiet
$pingDNS = Test-NetConnection -ComputerName $props.DNS&#91;0] -InformationLevel Quiet
$pingWebsite = Test-NetConnection -ComputerName 'kamilpro.com' -InformationLevel Quiet

$props | Add-Member -MemberType NoteProperty -Value $pingGateway -Name 'pingGateway'
$props | Add-Member -MemberType NoteProperty -Value $pingDNS -Name 'pingDNS'
$props | Add-Member -MemberType NoteProperty -Value $pingWebsite -Name 'pingWebsite'
# Installed roles – server only

IF ($props.isServer = $true) { $Roles = Get-WindowsFeature }

$props
```

