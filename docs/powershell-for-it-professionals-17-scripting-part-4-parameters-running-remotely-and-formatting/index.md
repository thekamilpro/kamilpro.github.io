# PowerShell for IT Professionals [#17] – Scripting part 4 – parameters, running remotely and formatting

{{<youtube yJ7nmSjPABM>}}

In the last part of scripting series, we will make the script to be able to query remote machines, e.g. servers.

We will also check how to add parameters to the script (and configure the default value of parameter) so that user will able to pass the parameter name like in a standard PowerShell cmdlet. 

Finally, we are going to format the script so that it looks more reliable and make some refactoring so that the logic is simpler.

## Exercises

  * Convert all commandlets that have more than one parameter so that they are called with splatting
  * Add another property to the $result so that it will include the time when information was gattered
  * The $pingWebsite has a hardcoded value of &#8216;kamilpro.com&#8217; &#8211; change it so that it uses a parameter instead
  * If you have multiple computers in your lab, retrieve all computers from AD and use script to query them. Hint: you can use the command to retrieve all computers as a parameter for $ComputerName, in that case, you might need to wrap the command in ( ) brackets.
  * Read about\_Comment\_Based_Help and add help to the script. You will need to add it at very top, above [cmdletbinding()], and wrap everything in <# ////your help #>. One of the keywords is .SYNOPSIS.

## Script

```powershell
[cmdletbinding()]
param(
    [string[]]$ComputerName = 'localhost'
)

Invoke-Command -ComputerName $ComputerName -ScriptBlock {
    Write-Verbose "Getting OS Info"
    $OS = Get-CimInstance -ClassName Win32_OperatingSystem
    
    Write-Verbose "Checking if its a server"
    $server = $false
    IF ($os.Caption -like "*Server*") { $server = $true }
    
    Write-Verbose "Retrieving IP configuration"
    $Network = Get-NetAdapter | 
        Where-Object { $_.Status -eq 'Up' } | Get-NetIPConfiguration
    
        Write-Verbose "Last updates"
    $Updates = Get-HotFix | 
        Sort-Object InstalledOn -Descending | Select-Object -First 5

    $result = &#91;pscustomobject]@{
        'OSName'        = $OS.Caption
        'OSVersion'     = $OS.Version
        'OSInstallDate' = $OS.InstallDate
        'isServer'      = $Server
        'IPv4Address'   = $Network.IPv4Address.IPAddress
        'Gateway'       = $Network.IPv4DefaultGateway.nexthop
        'DNS'           = $Network.DNSServer | Where-Object { $_.AddressFamily -eq '2' } | Select-Object ServerADdresses -ExpandProperty ServerAddresses
        'Updates'       = $Updates.HotfixID | Out-String
    }
    Write-Verbose "Checking network connectivity"
    $pingGateway = Test-NetConnection -ComputerName $result.Gateway -InformationLevel Quiet
    $pingDNS = Test-NetConnection -ComputerName $result.DNS&#91;0] -InformationLevel Quiet
    $pingWebsite = Test-NetConnection -ComputerName 'kamilpro.com' -InformationLevel Quiet

    
    $params = @{
        MemberType = 'NoteProperty'
        Value = $pingGateway
        Name = 'pingGateway'
    }
    $result | Add-Member @params
    
    $result | Add-Member -MemberType NoteProperty -Value $pingDNS -Name 'pingDNS'
    $result | Add-Member -MemberType NoteProperty -Value $pingWebsite -Name 'pingWebsite'
    # Installed roles – server only

    IF ($result.isServer = $true) {
        $roles= Get-WindowsFeature | 
            Where-Object { $_.Installed -eq $True -AND $_.FeatureType -eq 'Role' } |
            Select-Object Name -ExpandProperty Name
        $result | 
            Add-Member -MemberType NoteProperty -Value $roles -Name 'Roles'
    }

    return $result
}
```

