---
title: PowerShell Parameter Attributes - validate, group, require params and add pipeline to your function
author: Kamil
date: 2022-02-07
resources:
  - name: "featured-image"
    src: "featured-image.jpg"
categories: ["PowerShell", "Video",]
tags: [ "PowerShell", "pwsh", "validate", "pipeline", "parameter"]
---

{{< youtube hJIAK3qjlZQ >}}

Having properly configured parameters help your users to understand the use of your function. It also helps fellow developers to appreciate the intended use of the function. In addition, it gives you more control of how the code flows through your script.

Parameter attributes is what takes parameters to the next level - you can specify that certain parameters are mandatory, group them together so that only certain combination of parameters is meant to work together, we can even add pipeline support so that function can be used as any other built-in function.

I will show you all the above with even more, as I always strive to go profoundly in PowerShell explanation!

```ps1
# Sample script which asks user for basic information, then manipulates it and
# display some information

# There's no way (at least easy) to pass these parameters from prompt, nor validate 
$Name = Read-Host -Prompt "What's your name?"
$Age = Read-Host -Prompt "What's your age?"

Write-Host "=================="

$email = '{0}@company.com' -f $Name.ToLower()

Write-Host "Hi $($Name.ToUpper())"
Write-Host "your email address is: $email"

if ($Age -gt 17) {
    Write-Host "You will be able to access any website"
} else {
    Write-Host "We will turn on content blocker approriate to your age"
}

```ps1
@{
    users = @(
        @{ID = 1; FirstName = "John"; Surname = "Smith"; Active = $true; Region = "EU" }
        @{ID = 2; FirstName = "Elen"; Surname = "Smith"; Active = $true; Region = "NZ"}
        @{ID = 3; FirstName = "Andy"; Surname = "Oliver"; Active = $true; Region = "US" }
        @{ID = 4; FirstName = "Richard"; Surname = "Mann"; Active = $false; Region = "EU" }
    )
}
```

```ps1
function Get-Person {
    [cmdletbinding()]
    param (
        #[Parameter(Mandatory)]
        [string]$Surname,

        [string]$Firstname,

        [Parameter(Mandatory)]
        [int]$Id,

        [switch]$All,

        [switch]$IncludeInactive,

        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    
    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSBoundParameters.ContainsKey('All')) {
        return $items
    }

    if ($PSBoundParameters.ContainsKey('ID')) {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSBoundParameters.ContainsKey('Surname')) {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```

```ps1
function Get-Person {
    [cmdletbinding(DefaultParameterSetName = "All")]
    param (
        [Parameter(Mandatory, ParameterSetName = "byName")]
        [string]$Surname,

        [Parameter(ParameterSetName = "byName")]
        [string]$Firstname,

        [Parameter(Mandatory, ParameterSetName = "byId")]
        [int]$Id,

        [Parameter(ParameterSetName = "All")]
        [switch]$All,

        [switch]$IncludeInactive,

        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSCmdlet.ParameterSetName -eq "All") {
        return $items
    }

    if ($PSCmdlet.ParameterSetName -eq "byId") {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSCmdlet.ParameterSetName -eq "byName") {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```

```ps1
function Get-Person {
    [cmdletbinding(DefaultParameterSetName = "All")]
    param (
        [Parameter(Mandatory, ParameterSetName = "byName", Position = 1)]
        [string]$Surname,

        [Parameter(ParameterSetName = "byName", Position = 2)]
        [string]$Firstname,

        [Parameter(Mandatory, ParameterSetName = "byId", Position = 1)]
        [int]$Id,

        [Parameter(ParameterSetName = "All")]
        [switch]$All,

        [switch]$IncludeInactive,

        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSCmdlet.ParameterSetName -eq "All") {
        return $items
    }

    if ($PSCmdlet.ParameterSetName -eq "byId") {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSCmdlet.ParameterSetName -eq "byName") {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```

```ps1
function Get-Person {
    [cmdletbinding(DefaultParameterSetName = "All")]
    param (
        [Parameter(Mandatory, ParameterSetName = "byName", Position = 1, ValueFromPipeline)]
        [string]$Surname,

        [Parameter(ParameterSetName = "byName", Position = 2)]
        [string]$Firstname,

        [Parameter(Mandatory, ParameterSetName = "byId", Position = 1, ValueFromPipeline)]
        [int]$Id,

        [Parameter(ParameterSetName = "All")]
        [switch]$All,

        [switch]$IncludeInactive,

        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSCmdlet.ParameterSetName -eq "All") {
        return $items
    }

    if ($PSCmdlet.ParameterSetName -eq "byId") {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSCmdlet.ParameterSetName -eq "byName") {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```

```ps1
function Get-Person {
    [cmdletbinding(DefaultParameterSetName = "All")]
    param (
        [Parameter(Mandatory, ParameterSetName = "byName", Position = 1, ValueFromPipeline)]
        [string]$Surname,

        [Parameter(ParameterSetName = "byName", Position = 2)]
        [ValidateNotNullOrEmpty()]
        [string]$Firstname,

        [Parameter(Mandatory, ParameterSetName = "byId", Position = 1, ValueFromPipeline)]
        [ValidateRange(1,4)]
        [int]$Id,

        [Parameter(ParameterSetName = "All")]
        [switch]$All,

        [switch]$IncludeInactive,

        [ValidateSet("EU", "US", "NZ")]
        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSCmdlet.ParameterSetName -eq "All") {
        return $items
    }

    if ($PSCmdlet.ParameterSetName -eq "byId") {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSCmdlet.ParameterSetName -eq "byName") {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```

```ps1
function Get-Person {
    [cmdletbinding(DefaultParameterSetName = "All")]
    param (
        [Parameter(Mandatory, ParameterSetName = "byName", Position = 1, ValueFromPipeline)]
        [Alias("SecondName")]
        [string]$Surname,

        [Parameter(ParameterSetName = "byName", Position = 2)]
        [ValidateNotNullOrEmpty()]
        [Alias("GivenName")]
        [string]$Firstname,

        [Parameter(Mandatory, ParameterSetName = "byId", Position = 1, ValueFromPipeline)]
        [ValidateRange(1,4)]
        [int]$Id,

        [Parameter(ParameterSetName = "All")]
        [switch]$All,

        [switch]$IncludeInactive,

        [ValidateSet("EU", "US", "NZ")]
        [string]$Region
    )

    $items = [System.Collections.Generic.List[pscustomobject]]::new()
    $data = Import-PowerShellDataFile -Path "$PSScriptRoot\data.psd1"
    $users = $data.users

    if (!($IncludeInactive.IsPresent)) {
        $users = $users | Where-Object {$_.Active -eq $true}
    }

    if ($PSBoundParameters.ContainsKey('Region')) {
        $users = $users | Where-Object {$_.Region -eq $Region}
    } 

    foreach ($u in $users) {
        $items.Add([pscustomobject]$u)
    }

    if ($PSCmdlet.ParameterSetName -eq "All") {
        return $items
    }

    if ($PSCmdlet.ParameterSetName -eq "byId") {
        return $items.Where({$_.ID -eq $Id})
    }

    if ($PSCmdlet.ParameterSetName -eq "byName") {

        if ($PSBoundParameters.ContainsKey("Firstname")) {
            return $items.Where( { $_.Surname -eq $Surname -and $_.FirstName -eq $Firstname })
        }
        else {
            return $items.Where( { $_.Surname -eq $Surname })
        }
    }
}
```