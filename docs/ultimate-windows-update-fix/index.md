# Ultimate Windows Update fix

## The reason

I manage hundreds of workstations working on various Windows versions. As we know, Windows (and other Microsoft’s software) is updated monthly. While usually the process is just working fine, sometimes it simply doesn’t want to work.

And yes, probably Google is your best friend here&#8230; but really, who has time to search for the solution, and apply it, manually, if there is more than 1 computer?

## Use the script

I used technet, different posts and own knowledge of fixing that kind of issues. All it needs, is admin rights.

So what does the script do?

  1. Stops all services needed by Windows update
  2. Points client to download updates directly from Microsoft servers. I’ve noticed that if you don’t downstream all categories of updates (as this takes tonnes of free space) there might be an issue with downloading further updates. Still, if you’d like to keep using your internal update server, simply remove line no.5, starting with ‘reg delete’ and ending with ‘WindowsUpdate /f’.
  3. Removes local copies/cache/information of updates. Sometimes updates get corrupted, and the only way is to purge the catalogue, so that Windows Update will rebuild it next time you run updates.
  4. Restarting all services. You might notice that there is one service which hasn’t been stopped before &#8211; MpsSvc &#8211; it’s responsible for Windows Firewall. I’ve noticed that Windows Firewall service must be running (even if you don’t use it/use a different firewall), otherwise updates will fail to download.
  5. Run Windows update. I couldn’t figure it out (and couldn’t find the actual answer to this) but triggering the Windows Update from the command line doesn’t always work.

After all this, just go and fire out Windows Update, allow a few minutes and it should work.

## How to use the script?

<pre spellcheck="false" class="">net stop wuauserv
net stop cryptSvc
net stop bits
net stop msiserver

reg delete HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\WindowsUpdate /f
Del "%ALLUSERSPROFILE%\Application Data\Microsoft\Network\Downloader\qmgr*.dat"
Ren %systemroot%\SoftwareDistribution SoftwareDistribution.bak
Ren %systemroot%\system32\catroot2 catroot2.bak

netsh winsock reset

net start wuauserv
net start cryptSvc
net start bits
net start msiserver
net start MpsSvc

</pre>

All you need to do now, is paste the script from above to the notepad, save it as ‘Fix Updates.bat’ (don’t forget to pick All Files (\*.\*) from the drop down menu in Save As window. Then right click a newly created file, and choose run as administrator.

## Let me know how it worked!

This is my first ever IT strict post &#8211; let me know how you’ve found it! Was it useful? Was it explained well? I’d like to know!
