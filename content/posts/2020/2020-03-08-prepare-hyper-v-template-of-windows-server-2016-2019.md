---
title: Prepare Hyper-V template of Windows Server 2016/2019
author: Kamil
date: 2020-03-08T10:52:17+00:00
url: /prepare-hyper-v-template-of-windows-server-2016-2019/
featuredimage: /wp-content/uploads/2020/03/dan-gold-mgaS4FlsYxQ-unsplash.jpg
wp_last_modified_info:
  - 8th March 2020 @ 10:52 am
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - Deployment
  - IT
  - Server
  - Windows
tags:
  - hyper-v
  - server
  - sysprep
  - template

---
Deploying a new server is always fun! Well, at least the part after ISO download, installation, patching. Having said that, why not create a template so that you jump right into the fun? 

So we are essentially going to create a preconfigured VHDX file that can be attached to the newly created VM, and it will take you straight to the login screen.

# What&#8217;s needed

  * Windows server or Windows 10 with Hyper-V role and management tools enabled
  * [Windows Assessment and Deployment Toolkit (ADK)][1] &#8211; I always pick the latest version available, at the moment of writing this &#8211; Windows 10, version 1903 &#8211; is latest.
  * ISO with the source &#8211; I&#8217;ve used Server for 2016/2019 from my.visualstudio.com (you need an MSDN subscription for this), or from [trial center][2]

# Preparation

## Hyper-V

Open Windows PowerShell as admin and paste this command:

<pre class="wp-block-preformatted">Install-WindowsFeature -Name Hyper-V -IncludeAllSubFeature -Verbose</pre>

You will need to reboot your computer for hyper-v to become available

**Install External switch**

If you want to allow your template VM to connect to the Internet e.g. To download and install patches, you need an external switch set up on Hyper-V.

Open Hyper-V Manage and choose &#8220;Virtual Switch Manager&#8221; from the right-hand side pane,<figure class="wp-block-image size-large">

<img loading="lazy" width="900" height="822" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-e1583662120195-1.jpg?resize=900%2C822&#038;ssl=1" alt="" class="wp-image-1575" srcset="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-e1583662120195-1.jpg?w=916&ssl=1 916w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-e1583662120195-1.jpg?resize=300%2C274&ssl=1 300w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-e1583662120195-1.jpg?resize=768%2C702&ssl=1 768w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-e1583662120195-1.jpg?resize=755%2C690&ssl=1 755w" sizes="(max-width: 900px) 100vw, 900px" data-recalc-dims="1" /> </figure> 

Make sure you highlight the external switch and click Create Virtual Switch

On the next page give it a name, specify which network card connect it to and confirm.

Or use Powershell (it will attach VM switch to the first network adapter):

<pre class="wp-block-preformatted">$A = Get-NetAdapter 
New-VMSwitch -Name "ext1" -NetAdapterName $a[0].name</pre>

## ADK

Go to the [link][1] and download the latest ADK. When installing, make sure &#8220;Deployment tools&#8221; is selected &#8211; we are after Windows System Image Manager (SIM).<figure class="wp-block-image size-large">

<img loading="lazy" width="900" height="668" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-1-e1583663086572-1.jpg?resize=900%2C668&#038;ssl=1" alt="" class="wp-image-1580" srcset="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-1-e1583663086572-1.jpg?w=950&ssl=1 950w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-1-e1583663086572-1.jpg?resize=300%2C223&ssl=1 300w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-1-e1583663086572-1.jpg?resize=768%2C570&ssl=1 768w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-1-1-e1583663086572-1.jpg?resize=930%2C690&ssl=1 930w" sizes="(max-width: 900px) 100vw, 900px" data-recalc-dims="1" /> </figure> 

## Files extraction from ISO

We need to extract _install.wim_ file from the installation media. The file is a compressed Windows Image and is being used by SIM to create the answer file.

Double click on the ISO so that it mounts, go to Mounted ISO\Sources and copy _install.wim_ (it has a few gigabytes in size, biggest file in the folder) to your work directory.

You can unmount the ISO now, by right-clicking on it and choosing Unmount.

At this stage, we have everything we need to work.

# Create VM and install from ISO

Head to the Hyper-V Manager and choose to create a new VM, and options you choose:

Name: Something meaningful like e.g. Template &#8211; Server 2019 Standard GUI

Generation: Generation 2  
Assign Memory: Anything above 2048MB should be enough  
Configure networking: If you&#8217;ve created the external switch, pick it here  
Virtual Hard Disk: You can change default location and size. By default it will be dynamically expanding, that means it will only take as much space as needed.  
Installation Options: point to the ISO

On summary click finish.

What you can do at the stage is to give it more CPUs, 2 at least. It&#8217;s all about speed.

## Start your VM and prepare it as you wish

Now you can customise your server installation as you wish. Since it&#8217;s a server, I have tendency to only install updates, with the principal the less the better. But you can do whatever you&#8217;re fancy. 

Once you&#8217;re finished, take a checkpoint, as we are going to test if syspreping works. 

Press Windows + R and run:

%windir%\system32\sysprep\sysprep.exe

This will launch the System Preparation tool, aka sysprep. <figure class="wp-block-image size-large">

<img loading="lazy" width="424" height="317" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image.jpg?resize=424%2C317&#038;ssl=1" alt="System Preparation Tool 3.14 
Systern Prepua&m Tod 
Systen 
Ente SFten (lat •of-ux " class="wp-image-1552" srcset="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image.jpg?w=424&ssl=1 424w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/image.jpg?resize=300%2C224&ssl=1 300w" sizes="(max-width: 424px) 100vw, 424px" data-recalc-dims="1" /> </figure> 

Configure it as:

  * OOBE
  * Generalize
  * Reboot

IF you haven&#8217;t taken the checkpoint when I&#8217;ve asked last time, do it now. 

Checkpoint taken? Great, press ok and let the system reboot.

What will happen now is Windows will remove any unique identifiers, enabled drivers etc. Making it neutral.

Once your sysprep is completed, you&#8217;ll see a standard installation questions like language and administrator password &#8211; this is expected. At this stage, we are only testing if whatever changes you&#8217;ve made so far, don&#8217;t interfere with the sysprep process.

The successful test is considered when you can log in to Windows.

Roll back the VM to the checkpoint you&#8217;ve taken before trying sysprep.

# Prepare the answer file

Now we can move to a so-called answer/unattend file. The purpose of the file is to &#8220;answer&#8221; the installation questions for you, thus if the installer can find all answers, it will simply get you to the login screen after the first run.<figure class="wp-block-image size-large">

<img loading="lazy" width="900" height="452" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=900%2C452&#038;ssl=1" alt="" class="wp-image-1565" srcset="https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=1024%2C514&ssl=1 1024w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=300%2C151&ssl=1 300w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=768%2C386&ssl=1 768w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=1536%2C772&ssl=1 1536w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?resize=1373%2C690&ssl=1 1373w, https://i2.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-new-catalogue.gif?w=1800&ssl=1 1800w" sizes="(max-width: 900px) 100vw, 900px" data-recalc-dims="1" /> </figure> 

Open SIM, choose File > New Answer File > Point it to the install.wim file you&#8217;ve copied earlier

In the window say Yes

Choose the Windows edition you&#8217;d like to install and in following, windows say Yes

SIM will now start creating a catalogue with all possible options for the image, this process can take good 10 minutes. Just let it work.

When completed, you will see windows like that:<figure class="wp-block-image size-large">

<img loading="lazy" width="1920" height="1080" src="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?fit=900%2C506&ssl=1" alt="untitled - Windows System Image Manager 
File Edit Insert Tools Help 
Select a 
17763. I _nedrS 
177631 _neWS 
Add Setting to pass I windowsPE 
Add Setting to Pass 2 offlineServicing 
Add Seting to pass 3 genealize 
Add Setting to pass specialize 
Add Setting to pass 5 auditSystem 
Add Setting to Pass 6 auditUser 
Add Setting to pass 7 00beSystem 
x 
I widowsPE 
2 Serve-g 
6 •H User 
XML (O (O Sd (O 
%rosoft-Whdows- BrowserServke 
Passes 
Serverl_Ä 
Copy 
Ctrl. C 
I O. 17763. I _ne•-n 
I O. 17763. I 
17763. I _ne-trS 
17763. I 
13:34 
05/03/2020 " class="wp-image-1557" srcset="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?w=1920&ssl=1 1920w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=300%2C169&ssl=1 300w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=1024%2C576&ssl=1 1024w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=768%2C432&ssl=1 768w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=1536%2C864&ssl=1 1536w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=1260%2C709&ssl=1 1260w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=800%2C450&ssl=1 800w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?resize=1227%2C690&ssl=1 1227w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-1.jpg?w=1800&ssl=1 1800w" sizes="(max-width: 900px) 100vw, 900px" /> </figure> 

And we are after to panes here: Answer File and Windows image.

  * Windows Image is where you choose what to add to and to which phase of OS deployment
  * The answer file is where you configure specific answers

For start you&#8217;d like to add components:

  * &nbsp;amd64_Microsoft-Windows-International-Core, to Pass 4 Specialize and Pass 7 OOBE system
  * amd64_Microsoft-Windows-International-Core, to Pass 7 OOBE System

You do this by right-clicking the component Windows Image pane and choose the phase you&#8217;d like to add. There are many very similarly named records, thus pay attention.

Once added, you will see three &#8220;empty boxes&#8221; in the Answer File section. 

Let&#8217;s configure&nbsp; amd64_Microsoft-Windows-International-Core first, in both phases:<figure class="wp-block-image size-large">

<img loading="lazy" width="1920" height="1080" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?fit=900%2C506&ssl=1" alt="unattend.xml• - Windows System Image Manager 
File Edit Insert Tools Help 
17Æ3_ 
I _neWd 
I _neWd 
17763. I _neWd 
I _neWd 
17763. I _neWd 
17763. I 
17763. I 
17Æ3. I _neWd 
I S-4_neWd 
17763. I _neWd 
17763. I 
I _O. 17Æ3_ 
I 17763. I 
I 17763. I 
I _nedrd 
17Æ3_ I 
x 
nater-d 
I wrdowsPE 
XML (O) (O) Cor-fvün Sa (O) 
%rosoft-Whdows- InternathnaVCore 
Pass 
O'-GB 
13:45 
05/03/2020 " class="wp-image-1556" srcset="https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?w=1920&ssl=1 1920w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=300%2C169&ssl=1 300w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=1024%2C576&ssl=1 1024w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=768%2C432&ssl=1 768w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=1536%2C864&ssl=1 1536w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=1260%2C709&ssl=1 1260w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=800%2C450&ssl=1 800w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?resize=1227%2C690&ssl=1 1227w, https://i0.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-3.jpg?w=1800&ssl=1 1800w" sizes="(max-width: 900px) 100vw, 900px" /> </figure> 

By selecting one, you enter its values on the right-hand side pane Properties. 

You&#8217;d like to fill three fields: InputLocale, SystemLocale, UILanguage &#8211; in my example below you can see set up for British language. How can you find out all possible combinations? When you right-click the components in Answer File and choose Help it will take you to the website which enlists all values/language you can insert into these fields.

Time for amd64_Microsoft-Windows-Shell-Setup in here we need to values:

In the root\OOBE amend the record of HideEULAPage to True

And in root\UserAccounts\AdministratorPassword enter the password for the Administrator account.

After finishing it should look like this:<figure class="wp-block-image size-large">

<img loading="lazy" width="1330" height="790" src="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/SIM-completed-answer-file.gif?fit=900%2C534&ssl=1" alt="" class="wp-image-1568" /> </figure> 

Press Save > Save Answer File to save the file. We are going to use it for sysprep.

# Sysprep with unattended file

Copy the answer file to the desktop, and take another checkpoint of your VM.

Open shell as administrator and issue command (My username is Administrator and file is called Unattend.xml):

<pre class="wp-block-preformatted">&nbsp;C:\Windows\System32\Sysprep\Sysprep.exe /unattend:C:\Users\Administrator\Desktop\unattend.xml</pre>

Select OOBE, Generalize and Reboot

What will happen now the system will reboot and after a few minutes you should be welcomed by a login prompt (thus no language, licence, admin password questions) &#8211; if so we&#8217;re good to export the template for future deployment.

If any questions appear during the deployment phase, double-check your answer file if you&#8217;ve included all answers as suggested above. 

# Export template

All right, revert your VM to the last checkpoint. You can rename that check as &#8220;Before sysprep&#8221; &#8211; so that you can always comeback to it and make amendments to your image, or install new updates. 

Issue same command once again:

<pre class="wp-block-preformatted">&nbsp;C:\Windows\System32\Sysprep\Sysprep.exe /unattend:C:\Users\Administrator\Desktop\unattend.xml</pre>

But this time choose to shutdown the VM.

When the VM is shutdown, take another snapshot and call it something like &#8220;Export this&#8221; and export this checkpoint!<figure class="wp-block-image size-large">

<img loading="lazy" width="1329" height="446" src="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-e1583663021491-1.jpg?fit=900%2C302&ssl=1" alt="" class="wp-image-1577" srcset="https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-e1583663021491-1.jpg?w=1329&ssl=1 1329w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-e1583663021491-1.jpg?resize=300%2C101&ssl=1 300w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-e1583663021491-1.jpg?resize=1024%2C344&ssl=1 1024w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2020/03/image-2-e1583663021491-1.jpg?resize=768%2C258&ssl=1 768w" sizes="(max-width: 900px) 100vw, 900px" /> </figure> 

Exporting a checkpoint will effectively create a VM with merged VHDX file, and it&#8217;s VHDX file we need. All you need to do now is to go to the directory where you&#8217;ve exported the VM, and copy VHDX file from it to some place where you store your templates.

Now whenever you create a new VM, rather than pointing it to the ISO and creating a blank hard disk, simply make a copy of your template and attach that disk to VM.

## I&#8217;ve got my template, what now?

Since you&#8217;ve got the checkpoint of VM before syspreping, you can always bring install new updates, software etc. 

You can use scripting to create new VMs, with the use of the VHDx, thus making it the fully automatic solution.

If the post-deployment configuration is required, but you can&#8217;t have this on the golden image (e.g. AV software, management agent) you can use [setupcomplete.cmd][3] file.

If you&#8217;re interested in creating Windows 10 image, [read this article][4]. 

Photo by [Dan Gold][5] on [Unsplash][6]

 [1]: https://docs.microsoft.com/en-us/windows-hardware/get-started/adk-install
 [2]: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2019?filetype=ISO
 [3]: https://docs.microsoft.com/en-us/windows-hardware/manufacture/desktop/add-a-custom-script-to-windows-setup
 [4]: https://kamilpro.com/prepare-windows-10-1607-image/
 [5]: https://unsplash.com/@danielcgold?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [6]: https://unsplash.com/s/photos/picture?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
