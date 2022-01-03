# PowerShell for IT Professionals [#5] – Providers

{{<youtube MQ0hbnD8ERE>}}

## Exercises

- Create a folder in root of C: with the name of PowerShell
- Create a file inside the folder with the name of your choice and no content
- Retrieve all items from Env: drive
- Check the version of Notepad.exe in Windows directory
- Discover current Windows build version by registry property: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion


## Notes from the lesson

```powershell
# Provider allows to access some data storage (e.g. File system, registry, remote system, Active Directory)

# This concept might sound a bit abstract at first, but becomes very straight forward once you've used it for a while

# Provider once connected to the data storage, kind of "mounts" it like a network drive, allowing to browse and manage it

# Once the storage is mounted PowerShell creates so called PSDrive

# Let's review available providers
Get-Psprovider

# Let's review some PSDrives
Get-PSDrive

# A quick refresher how file system is organised: Drives, Folders, Files

# Since PowerShell connects to many different file data storages, it uses the generic term Item and specifies with type. Thus a folder is an Item of Type Directory, same as Registry key, it's best described in the help:
Help New-Item -full

#We have three main nouns we're working with:
	- Item -managing files, folders, registry keys, certificates etc.
	- ItemProperty - Get-ItemProperty c:\windows\explorer.exe | Format-List *
	- ChildItem - lists content of the directory - Get-ChildItem c:\Windows

# If you've used other shells in the past
# You change directories by using Set-Location, however CD is an alias to the command
# And since it's PowerShell, you're not only limited to file system

# You can create a new directory with New-Item -Name Folder -Type Directory, however you can use MKDIR

# You can list a content of current directory with Get-ChildItem, however DIR, and LS are also available

# You can list a content of a file with Get-Content, or you can use CAT

# Let's put some of what we've seen into practice

# File operations
New-Item -Name Temp -ItemType Directory -Path c:\

Set-Location C:\Temp

new-item -Type File -Name file1 -Value "Kamil"

 new-item -Type File -Name file2 -Value "Kamil"

Get-ChildItem

Get-Content file2

#Certificate
Get-ChildItem Cert:\CurrentUser\My\

# Exercises
	1. Create a folder in root of C: with the name of PowerShell
	2. Create a file inside the folder with the name of your choice and no content
	3. Retrieve all items from Env: drive
	4. Check the version of Notepad.exe in Windows directory
	5. Discover current Windows build version by registry property: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion
	```
