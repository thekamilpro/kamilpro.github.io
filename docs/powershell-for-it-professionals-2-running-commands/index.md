# PowerShell for IT Professionals [#3] – Running Commands

{{<youtube -KJfeC_Q-44>}}

## Exercises

  1. Display a list of all services that start with the letter W
  2. Display all processes (you might need to use Help to discover the right command. Remember the noun might be singular).
  3. Open Notepad (you can simply punch in notepad in the shell)
      1. Display only processes that have a name &#8220;Notepad&#8221;
      2. Once you can there&#8217;s only one instance of Notepad running, close it with PowerShell! (Verb would be Stop)
      3. Confirm with Shell there are no more Notepad processes running
  4. Display last 10 lines of _C:\windows\setupact.log_ file (the command you&#8217;d like to use has _Content_ in noun

## Commands used in video

```powershell
#We'll do some work on services now, let's see what commands are available
help *service

#Lets retrieve all services
Get-Service

#I'm more interested in bits and wuasvc
Get-Service -name  wuauserv,bits

#I can take a shortcut of parameter as long as 
Get-Service -n wuauserv,bits

#But as we know from help, name is positional parameter thus I can run same command as...
Get-Service wuauserv,bits

#Let's stop services
Stop-Service wuauserv,bits
Get-Service wuauserv,bits

#And start services
Start-Service wuauserv,bits
Get-Service wuauserv,bits

#In excercises I've asked to retrieve event logs
Get-EventLog -LogName Application -Newest 10 -EntryType Error

## Naming convention
# Verb-Noun. You can retrieve all "approved" verbs by
Get-Verb

## There's also a naming convention for nouns.
# Nouns from the same module usually start with the same "Prefix" e.g. Active Directory cmdlets start with AD

## As it's shell you can call other familiar programs, e.g. ping, nslookup etc
ping 8.8.8.8
nslookup bbc.com

## Some errors
GetService
Get Service
Gt-Service
```
