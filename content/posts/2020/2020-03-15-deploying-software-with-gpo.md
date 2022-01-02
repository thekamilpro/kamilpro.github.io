---
title: Deploying software with GPO
author: Kamil
type: post
date: 2020-03-15T14:25:59+00:00
url: /deploying-software-with-gpo/
featured_image: /wp-content/uploads/2020/03/ferdinand-stohr-rzA7ZuI8M5o-unsplash.jpg
wp_last_modified_info:
  - 15th March 2020 @ 2:26 pm
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - Deployment
  - IT
  - Windows
tags:
  - active directory
  - deployment
  - gpo
  - software

---
Having software consistently deployed across the fleet of computers is one of the key points that can be automated.

If I have a choice (and it makes sense), I will always choose GPO over any other solution because it&#8217;s embedded into Active Directory thus not requiring installation, learning and maintenance of additional management tool.

# MSI &#8211; Software Installation

One of the main advantages of MSI installers is that in 99% of cases you can just use the &#8220;Software Installation&#8221; option in GPO, and GPO will take care of the silent installation. In addition, if the newer version of the installer is ever listed, you can simply link the newer version of the installer, and GPO will take care of upgrading current installations. 

The rule is, if the installer doesn&#8217;t have any mandatory custom made parameters, it will silently push the software.

## How to find out if the MSI can be silently installed with GPO

We are going to use the MsiExec (Windows Installer) tool, my installer file is called: GoogleChromeStandaloneEnterprise64.msi

And command line syntax is: 

<pre class="wp-block-preformatted">Msiexec /i &lt;&lt;Path/File name>> /quiet</pre>

Thus making my command to look like this:

<pre class="wp-block-preformatted">msiexec /i GoogleChromeStandaloneEnterprise64.msi /quiet</pre>

If after issuing that command you can see your piece of software has been installed, it&#8217;s good to go with GPO.

## Set up GPO

To make the GPO working, we need to put an installer file on some network share, and since it&#8217;s going to be computers accessing the file, the computer must have read access &#8211; not the user &#8211; to the file. The easiest way to achieve this is to use NETLOGON folder. 

You access it via [\\<<DomainNAme>>\NETLOGON][1]

If you&#8217;re not sure what your domain name is, simply open Group Policy manager and it will be listed there:<figure class="wp-block-image size-large">

<img loading="lazy" width="351" height="310" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-4.jpg?resize=351%2C310&#038;ssl=1" alt="" class="wp-image-1587" srcset="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-4.jpg?w=351&ssl=1 351w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-4.jpg?resize=300%2C265&ssl=1 300w" sizes="(max-width: 351px) 100vw, 351px" data-recalc-dims="1" /> </figure> 

So I&#8217;m going to copy the installer file to my NETLOGON folder:

[\\company.pri\NETLOGON][2]

And create a new GPO:<figure class="wp-block-image size-large">

<img loading="lazy" width="1354" height="766" src="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/GPO-Software-Install.gif?fit=900%2C509&ssl=1" alt="" class="wp-image-1590" /> </figure> 

To create a GPO for the installer: 

After opening the Group Policy Management, right-click on the root/or OU and choose to create a new GPO and link it.

Give it an appropriate name and then expand settings:

Computer > Policies > Software Settings > Software Installation. Then right-click > New > Package.

Point it to your installer file &#8211; critical is you must point via network share; thus either [\\DomainName\someshare or \\ComputerName\share][3]&nbsp; 

Choose assign and job done.

# EXE and Non-standard MSI files

Sometimes the software installer requires providing custom switches, and the only way to achieve the installation is to use a custom script. Thankfully in GPO, we have an option of startup/shutdown scripts &#8211; that the script will execute every time the computers boots/shuts down. 

<p class="has-background has-cyan-bluish-gray-background-color">
  On the side note, I&#8217;ll add that you can schedule a task via GPO that will kick in the script on a regular basis, or every boot &#8211; in case you&#8217;d need that. I had once a need to deploy the management client that tended to stop its services with no apparent reason, making management of computers that were out in the field a daunting task. Got it solved by having the installer and script copied to the local machine and a scheduled task that was executing script once per hour. The problem of unmanaged computers has gone.
</p>

The problem with that kind of installers is the fact they often don&#8217;t advertise what switches are required, making it much more difficult. Thus at this stage, you might be forced to either check the developer&#8217;s website/manual or even contact them if the silent installation is supported. You can also have a look in the section of tips below which might point you to the solution.

Let&#8217;s take as an example the installer for Airtame &#8211; it&#8217;s MSI based, however, requires special switches for silent installation. Likely enough, the software vendor has provided the manual and switches required:

<https://help.airtame.com/en/articles/2543555-how-to-deploy-the-airtame-app-via-msi>

The fastest way to validate the script, without writing the script, is to run in the shell.

You can open the shell by pressing and holding SHIFT key, then right-click in the windows and choose &#8220;Open Windows shell here&#8221;

<pre class="wp-block-preformatted">msiexec /i "airtame-application-3.5.1-setup.msi" /quiet
WRAPPED_ARGUMENTS="/autostart=false /streaming_notification=true"</pre>

Once I&#8217;ve validated it&#8217;s been installed, time to create a script.

## GPO with the Startup script<figure class="wp-block-image size-large">

<img loading="lazy" width="1354" height="766" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/GPO-Startup-Script.gif?fit=900%2C509&ssl=1" alt="" class="wp-image-1591" /> </figure> 

I&#8217;ve put the installer to the NETLOGON as a previous installer.

Create a new GPO and configure it as follows:

Expand Computer Configuration > Policies > Windows Settings > Scripts > Startup

In the Startup Properties click &#8220;Show Files&#8230;&#8221; and create a new text file there, changing its extension to BAT (what I have set up on all my machines, is to show file extensions &#8211; this way I can change file extensions as a file name change), open it and paste the script below: 

<pre class="wp-block-preformatted">pushd //company.pri/NETLOGON
msiexec /i "Airtame-3.5.1-setup.msi" /quiet WRAPPED_ARGUMENTS="/autostart=false /streaming_notification=true"
Popd</pre>

First-line changes to the working directory to the folder where installation file is

The second line is the same command with testes moment ago

The last line will return the path the original location &#8211; not really needed, but it&#8217;s nice to have it.

Save the file. 

Before closing the Startup explorer window, copy the path (it&#8217;s ridiculously long) in my example:

[\\company.pri\SysVol\company.pri\Policies\{299CE059-AD3B-4F12-9DC3-3AC9EB36124E}\Machine\Scripts\Startup][4]

Coming back to &#8220;Startup Properties&#8221; window, click &#8220;Add&#8221; this time, and if you can&#8217;t see your script, paste the path to the window &#8211; now you can point to the script file you&#8217;ve just created.

Go to your test machine, run &#8220;gpupdate /force&#8221; from the shell and reboot your machine &#8211; validate the software got installed.

# Helpful websites

<https://chocolatey.org/search> &#8211; Chocolatey is a powerful package manager based on PowerShell. You can discover switches by reviewing their ps1 files, available on the website. It&#8217;s also a great product for installing and maintaining your application &#8211; have a look and you&#8217;ll forget what it&#8217;s like to Google for software installer and updating applications one by one.

<https://www.manageengine.com/products/desktop-central/software-installation/latest-software.html> &#8211; Another great tool, this one is being used for managing computers. On the link above, you can discover how they push particular pieces of software.

<https://www.itninja.com/> &#8211; a great website for IT Professionals, and in context of this post, has a lot of how-tos on deploying software.

<!--EndFragment-->

Photo by [Ferdinand Stöhr][5] on [Unsplash][6]

 [1]: file://DomainNAme/NETLOGON
 [2]: file://company.pri/NETLOGON
 [3]: file://DomainName/someshare%20or%20/ComputerName/share
 [4]: file://company.pri/SysVol/company.pri/Policies/%7b299CE059-AD3B-4F12-9DC3-3AC9EB36124E%7d/Machine/Scripts/Startup
 [5]: https://unsplash.com/@fellowferdi?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [6]: https://unsplash.com/s/photos/pattern?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText