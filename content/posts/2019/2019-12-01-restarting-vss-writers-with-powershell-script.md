---
title: Restarting VSS Writers with PowerShell script
author: Kamil
type: post
date: 2019-12-01T18:27:18+00:00
url: /restarting-vss-writers-with-powershell-script/
featured_image: /wp-content/uploads/2019/12/jason-yu-qpZFTRM-Bec-unsplash.jpg
wp_last_modified_info:
  - 22nd July 2020 @ 9:34 am
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - IT
  - Powershell
tags:
  - backup
  - feat
  - powershell
  - regex
  - select-string
  - vss writer
  - vssadmin
  - vsswriter

---
VSS Writers are responsible for backups &#8211; if they stop working, then backups might fail. The solution to that is restarting VSS Writers, but the problem here is that there are many, and at least out of the box, there&#8217;s no mechanism to make this automatic&#8230; but there&#8217;s PowerShell.

## I&#8217;m not interested in your code, I just want to restart these VSS Writers

In that case, head on to: [mygithub][1], use the code and run:

<pre class="wp-block-code"><code>Get-KPVSSWriter -Status Failed | Restart-KPVssWriter</code></pre>

That function will go ahead, retrieve all failed VSS Writers, and restart associated services with them. If that won&#8217;t help, then you probably need to reboot the server.<figure class="wp-block-image size-full is-resized">

[<img loading="lazy" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/07/KPVssWriter.gif?resize=900%2C441&#038;ssl=1" alt="Loading KPVssWriter.ps1 to memory and executing Get-KPVssWriter function" class="wp-image-1624" width="900" height="441" data-recalc-dims="1" />][2]<figcaption>Get-KPVssWriter</figcaption></figure> 

## Let&#8217;s talk PowerShell

The source code is available on [GitHub][1].

As far as VSS Writers go, there were a few issues needed to be addressed for the function:

  * the VssAdmin.exe tool is CMD only, and returns only strings.
  * There&#8217;s no filter in VssAdmin, thus it always returns all writers
  * The writers are associated with many Windows Services, and it&#8217;s not always obvious which service is responsible for given VSS Writer

Therefore I&#8217;ve written a couple of functions:

  * Get-KPVssWriter &#8211; that function will convert the output string from VssAdmin into a PowerShell objects + it can filter results by state (at the moment Stable or Failed).
  * Restart-KPVssWriter &#8211; this function takes the name of the VSS writer and restarts required service with it. It has hard-coded names of VSS writers with associated services.

How do you make a use of these?

You pipe results from Get function, to Restart function thus:

<pre class="wp-block-code"><code>Get-KPVSSWriter -Status Failed | Restart-KPVssWriter </code></pre>

will retrieve all failed writers and restart them. Let&#8217;s look into actual code.

### Get-KPVssWriter

First is to run VSSAdmin and start converting the output into PowerShell object:

#### Select-String

<pre class="wp-block-code"><code>VSSAdmin list writers | 
        Select-String -Pattern 'Writer name:' -Context 0, 4</code></pre>

These two lines will convert the output from vssadmin into PowerShell object. The Select-String really shines here as:

  * -Pattern parameter allows it to search for a specific phrase. In that case I&#8217;m looking for &#8216;Writer name:&#8217; as each listed writer starts with that line.
  * -Context 0,4 tells PowerShell that for each pattern (line) you found, match 0 line prior to it, and 4 past it. 

Thus if we look for this sample:

<pre class="wp-block-preformatted">Writer name: 'SqlServerWriter'
   Writer Id: {a65faa63-5ea8-4ebc-9dbd-a0c4db26912a}
   Writer Instance Id: {9a2c071c-f231-4a9d-abb3-0b15e1866b79}
   State: [1] Stable
   Last error: No error
Writer name: 'ASR Writer'
   Writer Id: {be000cbe-11fe-4426-9c58-531aa6355fc4}
   Writer Instance Id: {1dcb9866-6b86-4a5f-8dcc-eaf270bbfb2b}
   State: [1] Stable
   Last error: No error
Writer name: 'MSSearch Service Writer'
   Writer Id: {cd3f2362-8bef-46c7-9181-d62844cdc0b2}
   Writer Instance Id: {b4a2be5b-71d4-4579-871c-94b725607c33}
   State: [1] Stable
   Last error: No error</pre>

You can see that there&#8217;s pattern: Each Writer Name is followed by four line of text, and then the next one appears.

Select-String will then create an object for each pattern found, with properties of Line, Context.PostContext and Context.PreContext. 

And since we know that:

Line = pattern found, 

Context.PostContext = an array of each line found past pattern 

Context.PreContext. = an array of each line found prior the pattern 

It&#8217;s very easy to create the object, but first we need to remove the leftovers of the string.

#### Replace

<pre class="wp-block-preformatted">ForEach-Object {
            #Removing clutter
            Write-Verbose "Removing clutter "
            $Name = $_.Line -replace "^(.*?): " -replace "'"
            $Id = $_.Context.PostContext -replace "^(.*?): "
            $InstanceId = $_.Context.PostContext -replace "^(.*?): "
            $State = $_.Context.PostContext -replace "^(.*?): "
            $LastError = $_.Context.PostContext -replace "^(.*?): "</pre>

For every object generated by Select-String, I will assign a specific variable, and run -replace to remove unnecessary characters. Replace can also take regex , making it super effective while working with text.

#### PsCustomObject

The final step is to assembly all variables created in previous step into a PowerShell object:

<pre class="wp-block-preformatted">foreach ($Prop in $_) {
                $Obj = [pscustomobject]@{
                    Name       = $Name
                    Id         = $Id
                    InstanceId = $InstanceId
                    State      = $State
                    LastError  = $LastError
                } 
            }#foreach  </pre>

Since the main purpose of using this function is to restart failed, I thought it would be worth to add a parameter for filtering out VSS Writers based on status, thus this simple if statement which amends what will be outputed by the function:

<pre class="wp-block-code"><code> If ($PSBoundParameters.ContainsKey('Status')) {
                Write-Verbose "Filtering out the results"
                $Obj | Where-Object { $_.State -like "*$Status" }
            } #if
            else {
                $Obj
            } #else</code></pre>

There we go, the analogue application has been converted into a digital form (LOL), we can go ahead now and work on the actual restarts.

### Restart-KPVssWriter

This function will take the name of the VSS writer that requires restart, match Windows service to it, and restart the service.

#### Switch

PowerShell has a beatifull command &#8211; Switch &#8211; that eliminates the need of writing complex if statements. Let&#8217;s have a look:

<pre class="wp-block-code"><code> Switch ($Name) {
            'ASR Writer' { $Service = 'VSS' }
            'BITS Writer' { $Service = 'BITS' }
            'Certificate Authority' { $Service = 'EventSystem' }
            'COM+ REGDB Writer' { $Service = 'VSS' }
            'DFS Replication service writer' { $Service = 'DFSR' }
            'DHCP Jet Writer' { $Service = 'DHCPServer' }
            'FRS Writer' { $Service = 'NtFrs' }
            'FSRM writer' { $Service = 'srmsvc' }
            'IIS Config Writer' { $Service = 'AppHostSvc' }
            'IIS Metabase Writer' { $Service = 'IISADMIN' }
            'Microsoft Exchange Writer' { $Service = 'MSExchangeIS' }
            'Microsoft Hyper-V VSS Writer' { $Service = 'vmms' }
            'NTDS' { $Service = 'NTDS' }
            'OSearch VSS Writer' { $Service = 'OSearch' }
            'OSearch14 VSS Writer' { $Service = 'OSearch14' }
            'Registry Writer' { $Service = 'VSS' }
            'Shadow Copy Optimization Writer' { $Service = 'VSS' }
            'SPSearch VSS Writer' { $Service = 'SPSearch' }
            'SPSearch4 VSS Writer' { $Service = 'SPSearch4' }
            'SqlServerWriter' { $Service = 'SQLWriter' }
            'System Writer' { $Service = 'CryptSvc' }
            'TermServLicensing' { $Service = 'TermServLicensing' }
            'WINS Jet Writer' { $Service = 'WINS' }
            'WMI Writer' { $Service = 'Winmgmt' }
            default {$Null = $Service}
        } #Switch</code></pre>

Hell yeah, that would be quite a if statement. Instead of that, switch will match the name on the left, and execute the code to the right. 

The last line is interesting, default &#8211; if switch won&#8217;t match the name, it will execute that one instead. I&#8217;m casting that into $null.

#### If

The final step is to restart service:

<pre class="wp-block-code"><code>  IF ($Service) {
            Write-Verbose "Found matching service"
            $S = Get-Service -Name $Service
            Write-Host "Restarting service $(($S).DisplayName)"
            $S | Restart-Service -Force
        }
        ELSE {
            Write-Warning "No service associated with VSS Writer: $Name"
        }</code></pre>

Simple restart of service if it was matches earlier on, or just write warning if it wasn&#8217;t found.

## Happy ever after

I hope you found this useful. What can be done here is e.g. put the script into a scheduled task to run hourly, and hopefully you will have one problem less to worry about &#8211; as nobody likes to see failed backups on Monday morning.

Photo by&nbsp;[Jason Yu][3]&nbsp;on&nbsp;[Unsplash][4]

 [1]: https://github.com/kprocyszyn/tools/tree/master/KPVSSWriter
 [2]: https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/07/KPVssWriter.gif?ssl=1
 [3]: https://unsplash.com/@jason_yu?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [4]: https://unsplash.com/s/photos/writer?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText