---
title: 'PowerShell for IT Professionals [#18] – Working with text strings'
author: Kamil

date: 2020-11-03T17:33:18+00:00
url: /powershell-for-it-professionals-18-working-with-text-strings/
featuredImage: /wp-content/uploads/2020/11/Slide1.jpg
categories:
  - IT
  - PowerShell for IT Professionals
tags:
  - course
  - creation
  - grep
  - offset
  - powershell
  - regex
  - select-string
  - string
  - text
  - time
  - training
  - username
  - video
  - vssadmin
  - w32tm

---
{{<youtube ZZcHBTUhl5w>}}

Although PowerShell treats everything as object - including text strings - working with text might be particularly difficult - at least when first approached.

In this lesson we are going to have a closer look at what is possible with built in text methods and see how these can help us e.g. by extracting a substring of text.

PowerShell can also be great at parsing log files - we will use Select-String to quickly filter out the lines of text with the phrase we are looking for e.g. error.

Finally, we will have a go with regular expressions / regex and see how this can be used to find out a pattern of text, rather than a specific word.

## Script

```powershell
# String creation
$FirstName = "Kamil"
$Surname = "Procyszyn"

# String concatenation
"Hi, my name is " + $FirstName + " and my surname is " + $Surname.ToUpper() + ", it's nice to meet you."

# String interpolation
"Hi, my name is $FirstName and my surname is $($Surname.ToUpper()), it's nice to meet you."

# String with -f
"Hi, my name is {0} and my surname is {1}, it's nice to meet you." -f $FirstName, $Surname.ToUpper()

# Quotation marks
"$FirstName $SurName"
'$FirstName $SurName'
'{0} {1}' -f $FirstName, $Surname

# Joining string
$FullName = "$FirstName $Surname"
$FullName

# String as array
$FirstName[0]
$FirstName[-1]
$FirstName[0..2]

# String methods
$FirstName | Get-Member
$FirstName.ToLower()

$FullName
$FullName.IndexOf(' ')
$index = $FullName.IndexOf(' ')
$FullName.Substring($index + 1)

#Creating a username with three first letters of firstname, followed by underscore, followed by surname from full name
$FullName
$FullName.IndexOf(' ')
$index = $FullName.IndexOf(' ')

$Fir = $FullName.Substring(0, 3)
$Sur = $FullName.Substring($index + 1)
$Username = "{0}_{1}" -f $Fir, $Sur
$Username

### Select-String - Grep of PowerShell
### Don't confuse with Select/Select-Object

# Finding specific prase
vssadmin list shadows
vssadmin list shadows | Select-String 'creation time:'
vssadmin list shadows | Select-String 'creation time:' -Context 1
vssadmin list shadows | Select-String 'creation time:' -Context 0,4
vssadmin list shadows | Select-String 'creation time:' -Context 1,4

# Same with text files
Get-Content C:\Windows10Upgrade\upgrader_default.log
Get-Content C:\Windows10Upgrade\upgrader_default.log | Measure-Object
Get-Content C:\Windows10Upgrade\upgrader_default.log | Select-String "Error"

# Practical example - stripping time offset from w32tm
$time = w32tm /stripchart /computer:time.windows.com /dataonly /samples:1
$time
$time[-1]
$time[-1].Split(',')
$time[-1].Split(',')[-1]
$offset = $time[-1].Split(',')[-1]
$offset
$offset.IndexOf('.')
$offset = $offset.Substring(0, $offset.IndexOf('.'))
$offset.Trim()

# Regex - Regular expressions
$time | Select-String -Pattern '(\d+\.\d+s)$'
# We can check this website: https://regexr.com/
```
