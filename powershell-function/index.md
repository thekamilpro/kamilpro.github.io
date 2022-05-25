# PowerShell function - converting script into function with parameters


{{< youtube VGzEbEUfZCU >}}

So you've been writing your scripts for some time and wondering how to make them more PowerShell-like, so that they can be invoked from the console like all the other cmdlets?

In this quick video I'll show you how to convert a sample script into function, add parameters and indicated to your user how to use your function with mandatory parameters and types.

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
```

```ps1
# In this step we formalised parameters, but user must explicitly provide them
param (
    $Name,

    $Age
)

Write-Host "=================="

$email = '{0}@company.com' -f $Name.ToLower()

Write-Host "Hi $($Name.ToUpper())"
Write-Host "your email address is: $email"

if ($Age -gt 17) {
    Write-Host "You will be able to access any website"
} else {
    Write-Host "We will turn on content blocker approriate to your age"
}
```

```ps1
# In this case user can either pass parameters, or will be prompted as they are mandatory
# type is specified, therefore $Age only accepts integers
param (
    [Parameter(Mandatory)]
    [string]$Name,

    [Parameter(Mandatory)]
    [int]$Age
)

Write-Host "=================="

$email = '{0}@company.com' -f $Name.ToLower()

Write-Host "Hi $($Name.ToUpper())"
Write-Host "your email address is: $email"

if ($Age -gt 17) {
    Write-Host "You will be able to access any website"
} else {
    Write-Host "We will turn on content blocker approriate to your age"
}
```

```ps1
# Now we converted script into function, it must be first loaded to memory before it
# can be executed.
function Get-Person {
    param (
        [Parameter(Mandatory)]
        [string]$Name,

        [Parameter(Mandatory)]
        [int]$Age
    )

    Write-Host "=================="

    $email = '{0}@company.com' -f $Name.ToLower()

    Write-Host "Hi $($Name.ToUpper())"
    Write-Host "your email address is: $email"

    if ($Age -gt 17) {
        Write-Host "You will be able to access any website"
    }
    else {
        Write-Host "We will turn on content blocker approriate to your age"
    }
}
```
