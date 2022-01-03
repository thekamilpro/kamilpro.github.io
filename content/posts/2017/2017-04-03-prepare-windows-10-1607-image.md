---
title: How to prepare Windows 10 1607 image
author: Kamil

date: 2017-04-03T22:35:09+00:00
url: /prepare-windows-10-1607-image/
featuredImage: /wp-content/uploads/2017/04/rawpixel-com-196509.jpg
categories:
  - Deployment
  - IT
  - Windows
tags:
  - 1607
  - answer
  - deployment
  - feat
  - image
  - sysprep
  - unattend
  - windows 10

---
Sooner or later every company will come to the point when using standard Windows image is just not enough. Manual installation, fetching all updates or installing software etc. etc. takes time, and in IT, we really can use that time for something way more productive.

<!--more-->

Therefore, I propose you to create your very own, customised, Windows 10 image which will save you tonnes of time.

Let&#8217;s set some goals:

  * Automatic installation &#8211; no questions about user accounts, settings etc.
  * Hardware neutral &#8211; image will be installable on any kind of computer with necessary drivers equipped within.
  * Customised &#8211; image will be shipped with software and other preferences so that employees will be able to &#8220;just&#8221; log in straight after deployment.

Sounds good? So let&#8217;s start preparing.

# Prepare for Windows 10 image creation

I know you&#8217;d like to just jump in and start creating the new image, but to prepare the image properly there are necessary setup steps needed before the actual process. The benefits of this preparing will come handy in the future.

What you&#8217;re going to need is:

  * PC capable of virtualisation &#8211; Since the image creation will happen in VM (Virtual Machine), we need hardware capable of this. If you&#8217;ve got a dedicated machine already &#8211; that&#8217;s great &#8211; if not, then any computer manufactured within the last few years, which can afford at least 2GBs of memory for virtualisation (you can assign less than 2GBs of memory, but everything will be slow. Like really, slow)  and hard disk space &#8211; only you know how much space you&#8217;re going to use for this task. However, bare in mind you should allow extra space for the checkpoints. If you can afford to use SSD that&#8217;s great &#8211; it hugely improves the speed of the process.
  * Windows 10 ISO file  &#8211; you&#8217;d need this to install the actual image and to create so-called &#8220;answer&#8221; file out of it.
  * Windows ADK &#8211; this will be used for creating the answer file.
  * Hypervisor with checkpoint function &#8211; this will allow us to create a VM with the actual image, while checkpoints can save plenty of time in case something goes wrong or must be amended.
  * Software and customisation files which are going to be shipped with Windows.
  * Drivers.
  * Internet &#8211; not necessary, but really helpful.

As long as you&#8217;ve got the hardware capable of virtualisation, the rest comes from the Internet, or your software repositories.

## Windows 10 ISO file

The fastest way of getting the Windows 10 ISO file is to go to: <https://www.microsoft.com/en-gb/software-download/windows10ISO> or google &#8220;Windows Media Creation Tool&#8221;. Once on the website, pick &#8220;Windows 10&#8221; from the drop down menu (unless you&#8217;ve got licence for a different version). If your company happens to have a volume licence with Microsoft, then expand &#8220;More Download Options&#8221; where you can find links to MSDN websites.

When using Media Creation Tool, make sure to pick *.ISO file during the process. The file will be later on used in both, hypervisor and answer file.

Do not try to obtain images from any other, unofficial sources, as there&#8217;s always a chance of getting something &#8220;extra&#8221; from dishonest people. Nobody likes &#8220;extras&#8221; in Windows images.

## Windows ADK

<https://developer.microsoft.com/en-us/windows/hardware/windows-assessment-deployment-kit>

Windows  ADK (Assessment and Deployment Kit) is a set of tools required for image customisation and checking the performance of the computer. You need to fetch the version corresponding to the Windows image creation, as of writing this article, the newest version is Windows 10, version 1607.

Start the downloaded file, and upon the choice you need only 2 out of all:

  * Deployment tools &#8211; SIM (Windows System Image Manager) &#8211; this tool is used to create the answer file, which will allow for automatic installation of Windows.
  * Windows PE (Preinstallation Environment) &#8211; will be used for capturing the image. Unless you&#8217;ve got other solution in place (e.g. Norton Ghost) this tool is necessary.

## Hypervisor

The customised image will be created within the VM and therefore hypervisor is essential to it. The second requirement are snapshots, which will allow us to roll back any wrong changes, as well as re-use the image for any amendments/updates required after creating the initial image.

> Snapshot = Checkpoint. These words can be used interchangeably.

I can only think about 2 free options here &#8211; first is Microsoft Hyper-V, which comes built in Windows 8 or 10 in Pro edition, or any Windows Server since 2008 version. If you don&#8217;t have a such a possibility, you can use VirtualBox, which is free and has desired checkpoints functionality. Obviously, any hypervisor with checkpoints feature should do the trick.

I&#8217;ll be using Hyper-V since I&#8217;ve got Windows 10 Pro in my setup. To install Hyper-V on either Windows client or Server, make sure that Virtualization is enabled in your BIOS/UEFI setting, this is often called Intel VT, AMD-V or simply Virtualization Technology.

In order to instal Hyper-V in Windows Clients, go to Control Panel>Programs and Features>Turn Windows features on or off. Then expand Hyper-V menu and make sure that you can tick Hyper-V Management Tools and Hyper-v Platform. You will need to reboot the PC for the role to become active.

In Windows Server, you will need to add Hyper-V role to the server and reboot the server afterwards.

## Software

It might be any sort of installer or even copy-paste type of application. If it works with the standard Windows installation, it should work with the customised image.

Customised files &#8211; this will be any specific wallpapers, files etc. which are used to brand the image.

## Drivers

Since Windows 10 is shipped with plenty of drivers out of box, there are still many propetiary ones which must be either downloaded via WU (Windows Update) or installed manually. If your environment does not allow/provide drivers download via Windows Update, you must add them to the image.

<a href="https://kamilpro.com/add-any-drivers-to-windows-on-example-of-gpu-drivers/" target="_blank" rel="noopener noreferrer">I did write a whole blog of how to add any drivers to the image.</a>

In other words, you need drivers with \*.inf files, some manufacturers allow you to download the whole package of \*.inf files for a computer series, some don&#8217;t. In such a case, you might need to unpack the drivers manually to get *.inf files out of it.

# Prepare the environment for Windows 10 image

I hope you&#8217;re still bearing with me to this point, the most boring parts is behind us &#8211; I mean gathering all the stuff needed for the image creation.

Let&#8217;s look up the steps to follow:

  * Create VM
  * Install Windows
  * Put set Windows into Audit mode
  * Install software
  * Add drivers
  * Customise Menu Start
  * Creating answer file
  * Testing the image

Shall we begin?

## Create VM + Virtual Switch for VM

VM will need a connection to the local network as well as the Internet. Therefore we need to create a virtual switch to which the VM is going to be connected.

### Creating Virtual Switch

To create a virtual switch you need to:

  1. Virtual Switch manager
  2. Highlight new virtual switch, pick &#8220;External&#8221;, and click Create Virtual Switch
  3. Give it meaningful name e.g. Internet, make sure that External radius is ticked and choose the network card which used to connect to the Internet, click OK.
  4. The network switch is being created.

Once your virtual switch has been created, we can start creating the new VM and use the switch to connect VM to the Internet.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-1-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-1.png&quot;,&quot;id&quot;:&quot;892&quot;,&quot;title&quot;:&quot;Virtual Switch 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-2.png&quot;,&quot;id&quot;:&quot;893&quot;,&quot;title&quot;:&quot;Virtual Switch 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-3.png&quot;,&quot;id&quot;:&quot;894&quot;,&quot;title&quot;:&quot;Virtual Switch 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-4.png&quot;,&quot;id&quot;:&quot;895&quot;,&quot;title&quot;:&quot;Virtual Switch 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-5.png&quot;,&quot;id&quot;:&quot;896&quot;,&quot;title&quot;:&quot;Virtual Switch 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Virtual-Switch-6.png&quot;,&quot;id&quot;:&quot;897&quot;,&quot;title&quot;:&quot;Virtual Switch 6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

### Creating Virtual Machine

Now it&#8217;s time to create our virtual computer &#8211; a place where a customised image will be created, maintained and captured! Are you excited, like me?

  1. Right click on the name of your server, highlight new and choose Virtual Machine.
  2. Specify the name e.g. Windows 10_1607, and you can change the location where VM will be saved if needed. Make sure the disk you&#8217;re going to use will have at least the amount of space you plan to use for your VM.
  3. You can choose either generation, but I strongly recommend Generation 2 (which is supported for Windows 2012/8 onwards), generation 2 allows you to simulate UEFI, boot from SCSi adapters (which perform faster than IDE) and are power-failure proof.
  4. Memory. Assign at least 2 GBs, if you can 4GBs. You can leave Dynamic memory tick on, however, it shouldn&#8217;t make any difference in this scenario.
  5. In the configuring networking window, choose the Virtual Switch created earlier on, this will allow the VM to talk to the rest of the world.
  6. Virtual disk &#8211; in other words, the special file which behaves like a hard drive. Probably you will only want to amend the size of, I find 60GB very minimum, but again, it&#8217;s entirely up to you.
  7. The last step needed for the configuration is Installation option &#8211; in other words, pointing VM to the installation media. Simply choose the ISO downloaded earlier on.
  8. On the Summary window, verify all the settings and hit finish if you&#8217;re happy with all of them.

Hyper-V will create your VM now, you should be able to see it in the main Windows of the Hyper-V.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-2-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-1.jpg&quot;,&quot;id&quot;:&quot;911&quot;,&quot;title&quot;:&quot;VM Create 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-2.jpg&quot;,&quot;id&quot;:&quot;912&quot;,&quot;title&quot;:&quot;VM Create 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-3.jpg&quot;,&quot;id&quot;:&quot;913&quot;,&quot;title&quot;:&quot;VM Create 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-4.jpg&quot;,&quot;id&quot;:&quot;914&quot;,&quot;title&quot;:&quot;VM Create 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-5.jpg&quot;,&quot;id&quot;:&quot;915&quot;,&quot;title&quot;:&quot;VM Create 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-6.jpg&quot;,&quot;id&quot;:&quot;916&quot;,&quot;title&quot;:&quot;VM Create 6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/VM-Create-7.jpg&quot;,&quot;id&quot;:&quot;917&quot;,&quot;title&quot;:&quot;VM Create 7&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

# Windows 10 installation

Yes, we are just a step before installing, but hold your horses for a second, there&#8217;s one more point worth considering.

## Audit mode or OOBE mode?

When you perform the standard installation of Windows, you simply boot to OOBE (Out of box experience) every single time. Then you go through the process of customising the installation and creating user account etc.

Audit mode is, on the other hand, a special mode dedicated to preparing Customised Windows image. It will e.g. automatically log in to the Administrator account, which will be removed once the work in the Audit mode has been finished. Changes made in the Audit mode will be then copied to every new user logging to the PC, which keeps the user experience. Yet working in the Audit mode is exactly identical like in OOBE.

You can even switch between OOBE/Audit modes once Windows 10 has been installed, but why not start from using Audit and simply keep using it, from the beginning?

Therefore it&#8217;s not required, but I&#8217;d recommend you to use the Audit mode for the image creation.

## Let&#8217;s install Windows 10!

All the preparation will now pay off &#8211; you&#8217;ll be able to go and set up everything as desired, using the software which you&#8217;ve prepared earlier on.

As mentioned earlier, I&#8217;ll follow the Audit mode way and prepare my image in that way. The process of installing is almost the same for Audit and OOBE mode, however, I&#8217;m going to indicate the moment when you need to make the call.

  1. Start the VM.
  2. On the screen &#8220;Press any key to boot&#8221;, press any key.
  3. On the &#8220;Windows Setup&#8221; screen, choose Install now.
  4. Now you&#8217;re going to be asked to provide a licence key. In case you don&#8217;t have it yet, you can simply skip this and provide the key later.
  5. Choose the edition of Windows 10 which you&#8217;re going to install. I recon it&#8217;s going to be Pro, which will allow you, among other things, join the computer to the domain.
  6. Once you are aware of the agreement content, tick the box and move forward.
  7. Since this is a freshly created VM, its hard drive it&#8217;s literally empty. Therefore the only thing you can do now is to choose Custom: Install Windows only.
  8. On the following screen just click next, Windows installer will partition the drive and utilise all space. Even if you&#8217;d need additional partitions, you&#8217;ll be able to order Windows to create them during the deployment process.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-3-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-1.jpg&quot;,&quot;id&quot;:&quot;925&quot;,&quot;title&quot;:&quot;Windows 10 installation 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-2.jpg&quot;,&quot;id&quot;:&quot;926&quot;,&quot;title&quot;:&quot;Windows 10 installation 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-3.jpg&quot;,&quot;id&quot;:&quot;927&quot;,&quot;title&quot;:&quot;Windows 10 installation 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-4.jpg&quot;,&quot;id&quot;:&quot;928&quot;,&quot;title&quot;:&quot;Windows 10 installation 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-5.jpg&quot;,&quot;id&quot;:&quot;929&quot;,&quot;title&quot;:&quot;Windows 10 installation 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-7.jpg&quot;,&quot;id&quot;:&quot;930&quot;,&quot;title&quot;:&quot;Windows 10 installation 7&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-8.jpg&quot;,&quot;id&quot;:&quot;931&quot;,&quot;title&quot;:&quot;Windows 10 installation 8&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

The Windows installation starts now. It will take some time and reboots before reaching the next screen, on which you&#8217;ll need to make a decision about the mode used for installation.

The Audit mode trigger is hidden; you must use a keyboard shortcut to activate it.

Press simultaneously ALT + SHIFT + F3, after which VM will instantly restart and boot to audit mode shortly.

Windows will now automatically log in with Administrator&#8217;s account, and present you with SYSPREP (System Preparation Tool) window, which you&#8217;d like to cancel. The tool allows you to set Windows 10 back to OOBE mode generalise (which I&#8217;ll explain later on) and restart it. Just cancel it. It might be worth to take a check point at this moment.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-4-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Windows-10-installation-9.jpg&quot;,&quot;id&quot;:&quot;934&quot;,&quot;title&quot;:&quot;Windows 10 installation 9&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Audit-mode-1.jpg&quot;,&quot;id&quot;:&quot;933&quot;,&quot;title&quot;:&quot;Audit mode 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

# Customise the image

Now it&#8217;s time to install all the software you need, but how to provide it? We could use so-called guest services and start copying all the installation files, but that would take first time to do so, secondly the size of the VHD would start growing.

Rather than that, let&#8217;s go good, old network sharing. Not only it will allow us to install folders without copying them all to the VM, but also will deliver the answer file, or will allow exporting e.g. customised menu start settings.

## Create the network share on your host machine

Open the folder where you&#8217;re going to store all files needed by your VM.

  1. Open properties of the folder.
  2. On the properties window, change tab to &#8220;Sharing&#8221; and choose &#8220;Advanced Sharing&#8230;&#8221;
  3. Tick the box &#8220;Share this folder&#8221; and choose &#8220;Permissions&#8221;. You might note down the folder name as this will become the name of the actual share.
  4. Grant full access to the account which is going to access the network share on your host machine. I granted access to Everyone.
  5. Click OK twice and once backed to the properties window, change tab to &#8220;Security&#8221; and choose &#8220;Edit&#8221; button.
  6. Click Add.
  7. Type the same username which you&#8217;ve used in the Sharing permissions.
  8. Grant &#8220;Allow &#8211; Modify&#8221; rights to the user. Click OK on all windows.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-5-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-1.jpg&quot;,&quot;id&quot;:&quot;938&quot;,&quot;title&quot;:&quot;Network Share 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-2.jpg&quot;,&quot;id&quot;:&quot;939&quot;,&quot;title&quot;:&quot;Network Share 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-3.jpg&quot;,&quot;id&quot;:&quot;940&quot;,&quot;title&quot;:&quot;Network Share 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-4.jpg&quot;,&quot;id&quot;:&quot;941&quot;,&quot;title&quot;:&quot;Network Share 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-5.jpg&quot;,&quot;id&quot;:&quot;942&quot;,&quot;title&quot;:&quot;Network Share 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-6.jpg&quot;,&quot;id&quot;:&quot;943&quot;,&quot;title&quot;:&quot;Network Share 6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-7.jpg&quot;,&quot;id&quot;:&quot;944&quot;,&quot;title&quot;:&quot;Network Share 7&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Network-Share-8.jpg&quot;,&quot;id&quot;:&quot;945&quot;,&quot;title&quot;:&quot;Network Share 8&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

You might ask why we&#8217;ve had grant access to the same user in 2 different places. Since this is not an article about shares and permissions I&#8217;ll keep it short &#8211; Windows differentiates 2 levels of access &#8211; local and networked. The user must have access locally to be able to access the files at all and locally, that is, while working on the actual computer. Networked access allows users to access the files over the network, and folders accessed that way are called shares.  In the case of when a user has only network rights, one still won&#8217;t be able to access the files because of the lack of the local permission.

## Map the network drive

Once the network share has been created, it&#8217;s time to map it. That process is called mapping the network drive because ultimately it will show up alongside other drives &#8211; hard and flash disks and optical drives.

  1. Head towards &#8220;This computer.&#8221;
  2. Choose &#8220;Computer&#8221; tab and click &#8220;Map network drive.&#8221;
  3. On the following screen specify the location of the network share &#8211; the format is: **\\<name of the host>\<name of the share>**, the name of my host is KMLPRO and the name of the share is VM\_installers therefore the full path will be: \\KMLPRO\VM\_installers.
  4. Tick the box &#8220;Connect using different credentials&#8221;. You might also tick the box &#8220;Reconnect at sign-it&#8221;, that way Windows will store your login details to the server and automatically map the drive the next time you log in. Feel free to change the drive, if you&#8217;d like to.
  5. You&#8217;ll be asked to provide details of the account which you&#8217;ve granted access rights earlier on.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-6-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Mapping-network-drive-1.jpg&quot;,&quot;id&quot;:&quot;947&quot;,&quot;title&quot;:&quot;Mapping network drive 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Mapping-network-drive-2.jpg&quot;,&quot;id&quot;:&quot;948&quot;,&quot;title&quot;:&quot;Mapping network drive 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Mapping-network-drive-3.jpg&quot;,&quot;id&quot;:&quot;963&quot;,&quot;title&quot;:&quot;Mapping network drive 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Mapping-network-drive-4.jpg&quot;,&quot;id&quot;:&quot;964&quot;,&quot;title&quot;:&quot;Mapping network drive 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

Once completed, you&#8217;ll find your mapped network drive alongside other drives in the computer view.

## Install the software, customise the settings

Now you&#8217;ve got to the point where you can perform an installation of any software needed.

  * Try to use officially released MSI files. In the case of updating the image later on, MSIs always make sure to get rid of the old version of the file and upgrade.
  * If possible, avoid launching the applications. It might amend some default settings which users might not like.
  * Install the latest .NetFramework (as well as other libraries) possible, that will save time, both your and your users.
  * Do not forget about the new Menu Start, you can pin applications often used by the users.

Once you&#8217;ve completed the customisation process, take the checkpoint. Just in case.

## Drivers

&#8220;Software comes and goes. Hardware is forever,&#8221; they said, but to make that hardware work, we need drivers.

### Why not rely on the Internet/Windows updates?

The problem with drivers is that they often won&#8217;t install, unless there&#8217;s actual hardware present at the moment of installation. You can obviously rely on Windows Update and hope that everything will be self-installed, but do you really want to?

I personally cannot imagine deploying Windows to more than 10 computers and allowing them to go to Windows Update and start downloading, then wait for reboots etc. Nope, that doesn&#8217;t work like that in an enterprise.

Unless you&#8217;re in the situation where you can utilise Windows Update, then you can skip that part. But you&#8217;ll miss so much fun with having control of every single driver on the image.

### PNPUTIL &#8211; Drivers&#8217; best mate

Microsoft includes in the Windows a brilliant tool called PNPUTIL, which basically allows managing drivers and give you as the administrator the control over the drivers inside of the VM.

Have I mentioned that we will need to go command line? Oh well, now I&#8217;m telling you. But it won&#8217;t be scary, I promise. PNPUTIL is a command line only.

#### Main PNPUTIL switches which you should know about:

<pre>/enum-drivers - enumarates (lists) all stored drivers
/add-driver - adds driver to the driver store
/delete-driver - removes the driver from the driver store</pre>

#### Driver store &#8211; what is this?

Windows comes with plenty of drivers out of the box &#8211; however, it&#8217;s impossible to include every single driver to every single device, or even make sure that all the drivers are up to date.

Therefore, here comes the idea of the Driver Store &#8211; think about it like installers for your software &#8211; you have the installer, but you install the software when it&#8217;s actually needed by the user.

Same happens with the hardware &#8211; when a new device is connected, Windows goes through the store and looks for the appropriate driver. That way we can load all needed drivers and Windows will use them only when they are needed. This is cool, isn&#8217;t it?

#### Let&#8217;s add some drivers!

If you&#8217;ve done a good job preparing your drivers (*.inf files) organising them into folders (e.g. C:\Temp\Drivers\Audio, C:\Temp\Drivers\Video etc.), then follow the instruction below. If not, go and organise them now.

  1. Go to the folder where your drivers are, press and hold the SHIFT ket + right-click the mouse and choose &#8220;Open command window here&#8221;.
  2. Type &#8220;pnputil&#8221; and hit enter, familiarise yourself with the command details (for your own reference)
  3. Type **pnputil /enum-drivers** and look how many drivers are needed. Since this is VM, there a just a few in use.
  4. Type **pnputil /add-driver *.inf /subdirs**. This will order pnputil to go through all the folders and add any driver it finds to the store. It might take a while depending on amount and size of drivers you&#8217;ve prepared.
  5. Upon the completion, you&#8217;ll see how many drivers have been added to the store.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-7-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/PNPUTIL-1.jpg&quot;,&quot;id&quot;:&quot;965&quot;,&quot;title&quot;:&quot;PNPUTIL 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/PNPUTIL-2.jpg&quot;,&quot;id&quot;:&quot;966&quot;,&quot;title&quot;:&quot;PNPUTIL 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/PNPUTIL-3.jpg&quot;,&quot;id&quot;:&quot;967&quot;,&quot;title&quot;:&quot;PNPUTIL 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/PNPUTIL-4.jpg&quot;,&quot;id&quot;:&quot;968&quot;,&quot;title&quot;:&quot;PNPUTIL 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/PNPUTIL-5.jpg&quot;,&quot;id&quot;:&quot;969&quot;,&quot;title&quot;:&quot;PNPUTIL 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

This is it, that one step closer to the hardware neutral Windows image. Enumerate the drivers now, see how many has been added to the Drivers now. Don&#8217;t forget to take the checkpoint.

# Unattended installation

Are we there yet? Yes, almost there!

Once you are happy with all customisations you&#8217;ve just done, it&#8217;s time to tell Windows, what to do during the installation. Why? To not get all that prompts about licencing, partitioning, the name of the computer and user etc. We are going to be like this bartender from the local pub, who knows us good enough to serve everything we like. Sounds cool? Keep it coming then!

I must admit, I was really impressed (like WOW impressed) when saw fully automatic Windows deployment for the first time in my life &#8211; from the beginning of the process &#8211; when Windows configured itself until it reached the Logon/Desktop screen.

## Creating WIM file

Unattended installation means automatic installation &#8211; which is our goal. Essential part of it is the Answer file &#8211; the file which will tell Windows installer what to do during the installation. We obviously, are going to create are own answer file.

We are going to need WIM file, which is in short take, the actual compressed Windows &#8211; whenever you install Windows, it&#8217;s being decompressed and installed to the local hard disk. Where&#8217;s the file then? You&#8217;ve guessed it right, on the Windows 10 ISO, the same which we&#8217;ve used to install Windows 10 earlier on.

Microsoft started using ESD files to include Windows installation files, which are way more compressed than actual WIM files. Therefore first step is to copy the ESD file off the ISO, then convert it to the WIM file to finally use for the Answer file creation.

  1. Right click on the ISO which you&#8217;ve used to install the Windows 10, mount it.
  2. ISO will show as ordinary DVD drive, open it and go to the &#8220;Sources&#8221; folder, and find **install.esd** file. It&#8217;s the biggest (approximately 4GBs) file over there. Copy the file off the image. I can suggest to create a dedicated folder where you&#8217;re going to store all ISOs used for installation and another folder called WIM where you&#8217;ll store WIM files. The reason behind is that you should always use the WIM file from ISO used for installation. Don&#8217;t forget about meaningful names.
  3. We need elevated (administrator) command prompt, right click on the Windows menu and choose it then. Change directory to where the ISO is.
  4. You can use DIR command to make sure you&#8217;re in the right folder and can see install.erd file.
  5. Type command: **dism /Get-WimInfo /WimFile:install.esd** You should see the list of available Windows 10 edition included in the ISO. I&#8217;m going to need Pro edition therefore will choose Index:1
  6. Type command: **dism /export-image /SourceImageFile:install.esd /SourceIndex:<span style="color: #ff0000;">1</span> /DestinationImageFile:<span style="color: #ff0000;">Windows10_pro_1607</span>.wim /Compress:max /CheckIntegrity **I&#8217;ve highlighted the index number and the file name in case you&#8217;d like to change it.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-8-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/WIM-1.jpg&quot;,&quot;id&quot;:&quot;990&quot;,&quot;title&quot;:&quot;WIM 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/WIM-2.jpg&quot;,&quot;id&quot;:&quot;991&quot;,&quot;title&quot;:&quot;WIM 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/WIM-3.jpg&quot;,&quot;id&quot;:&quot;992&quot;,&quot;title&quot;:&quot;WIM 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/WIM-4.jpg&quot;,&quot;id&quot;:&quot;993&quot;,&quot;title&quot;:&quot;WIM 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

Wait until the process has finished and we can move on to the actual answer file creation. You&#8217;ll find the WIM file in the same location where ESD file is.

## Answer File

It&#8217;s time to tell Windows off, literally. We&#8217;re going to prepare the answer file, which will guide Windows through the installation &#8211; and achieve automatic installation there.

There are tonnes of possible settings to configure, therefore I will focus on bringing you to the automated installation which you might use as the base for your answer file. Most of the settings are well documented which is a great help.

To create the answer file:

  1. Open Windows System Image Manager.
  2. Press the &#8220;New Answer File&#8221; button.
  3. Say yes to open the Windows Image file.
  4. Point to the WIM file created earlier on.
  5. Confirm creation of the catalog file.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-9-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-1.jpg&quot;,&quot;id&quot;:&quot;999&quot;,&quot;title&quot;:&quot;Answer file 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-2.jpg&quot;,&quot;id&quot;:&quot;1000&quot;,&quot;title&quot;:&quot;Answer file 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-3.jpg&quot;,&quot;id&quot;:&quot;1001&quot;,&quot;title&quot;:&quot;Answer file 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-4.jpg&quot;,&quot;id&quot;:&quot;1002&quot;,&quot;title&quot;:&quot;Answer file 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-5.jpg&quot;,&quot;id&quot;:&quot;1003&quot;,&quot;title&quot;:&quot;Answer file 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/03\/Answer-file-6.jpg&quot;,&quot;id&quot;:&quot;1004&quot;,&quot;title&quot;:&quot;Answer file 6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

It will take a few minutes for SIM to complete the image creation process. Upon finish, you&#8217;ll be presented with the new SIM windows with more options than before.

Answer file reflects different Windows installation stages (passes) &#8211; there are 7 of them, however, we need only a few for our needs.

The components part contains the actual jobs that Windows Installer performs during different parts of the installation. One component can be added to multiple installation stages. Components are added by right-clicking on them and choosing to which part they need to be added.

<img loading="lazy" class="aligncenter wp-image-1005 size-full" src="https://i1.wp.com/kamilpro.com/wp-content/uploads/2017/03/Answer-file-6-1.jpg?resize=900%2C726&#038;ssl=1" alt="" width="900" height="726" srcset="https://i1.wp.com/kamilpro.com/wp-content/uploads/2017/03/Answer-file-6-1.jpg?w=939&ssl=1 939w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2017/03/Answer-file-6-1.jpg?resize=300%2C242&ssl=1 300w, https://i1.wp.com/kamilpro.com/wp-content/uploads/2017/03/Answer-file-6-1.jpg?resize=768%2C619&ssl=1 768w" sizes="(max-width: 900px) 100vw, 900px" data-recalc-dims="1" /> 

Necessary components for automated installation:

Phase 1 WinPE:

  * amd64_Microsoft-Windows-International-Core-WinPE
  * amd64_Microsoft-Windows-Setup

Phase 4 Specialize:

  * amd64_Microsoft-Windows-Shell-Setup

Phase 7 OOBE

  * amd64_Microsoft-Windows-International-Core
  * amd64_Microsoft-Windows-Shell-Setup

I really wanted to describe all the components, meaning what you should type and where, but it would take way too much time and space. Instead, I&#8217;ll simply paste the answer file here and describe what it does in each phase.

[Answer file &#8211; Windows\_10\_pro.xml][1] (right click and dowload/save link as)

Use SIM to open the file above, and point to Windows 10 1607 WIM file.

### Phase 1 WinPE

That one happens when you actually boot a computer from the installation media and are presented with the install now screen.

What happens in &#8220;amd64_Microsoft-Windows-International-Core-WinPE&#8221; is to choose which display language should be picked.

The &#8220;amd64_Microsoft-Windows-Setup&#8221; tells installer to wipe the first hard drive, then create 4 partitions (as needed for UEFI) and to modify them. While first 3 partitions are needed for Windows to run and have a fixed size, the 4th &#8211; where the actual OS is &#8211; will use all remaining space for itself.

### Phase 4 Specialize

That phase happens when Windows is &#8220;Getting ready&#8221;. You can notice a CopyProfile option &#8211; it tells the installer to take Administrator&#8217;s profile (the same which you&#8217;ve customised earlier on in the Audit mode) and set it as the default profile &#8211; so that every profile created on the computer will have the same customisation.

The product key used here is generic Windows 10 Pro, you should change it to yours. You won&#8217;t be able to activate Windows with that key.

### Phase 7 OOBE

OOBE &#8211; Out of box experience &#8211; is the Wizard which you see during the first Windows boot.

&#8220;amd64_Microsoft-Windows-International-Core&#8221; will tell the computer which language, region and keyboard layout will be used for the users.

&#8220;amd64_Microsoft-Windows-Shell-Setup&#8221; will accept the agreement, set Windows to get automatic Windows Updates and create local user &#8220;Dave&#8221; who will be an administrator. Notice that password field is hashed, therefore you should change it to yours.

#### Defaultuser0 bug

When at some point you&#8217;ll start testing/deploying the image, you might notice a 2nd user account created during the installation, called defaultuser0. It&#8217;s being created during the CopyProfile function, I couldn&#8217;t find an explanation why it happens.

We could switch off the CopyProfile, but then it would be necessary to spend a way more time with customising (including start menu, theme, wallpaper etc. would need to be set, configured with text files and exported &#8211; which is not really user friendly).

Or we could keep using CopyProfile and call a command &#8220;net user defaultuser0 /delete&#8221; during first user logon, which removes it automatically. I prefer the second option, which is included in the answer file.

## Save the answer file

Once you&#8217;re happy with your answer file, save in the shared folder created earlier on.

# Preparing the image for capture/housekeeping

We need to do some housekeeping to make the image clean and neat for the end user:

  * Run Windows updates
  * Run antivirus updates
  * Remove all browsing history from Internet browsers
  * Unmount all mounted network drives
  * Right click on the address bar in Windows explorer and choose &#8220;Delete history&#8221;
  * Run disk clean up
  * Make sure there are no passwords saved in the Credential manager (Control panel>Credential Manager)
  * Remove any directories used during image preparation (e.g. C:\temp)

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-10-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Housekeeping-1.jpg&quot;,&quot;id&quot;:&quot;1014&quot;,&quot;title&quot;:&quot;Housekeeping 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Housekeeping-2.jpg&quot;,&quot;id&quot;:&quot;1015&quot;,&quot;title&quot;:&quot;Housekeeping 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Housekeeping-3.jpg&quot;,&quot;id&quot;:&quot;1016&quot;,&quot;title&quot;:&quot;Housekeeping 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Housekeeping-4.jpg&quot;,&quot;id&quot;:&quot;1017&quot;,&quot;title&quot;:&quot;Housekeeping 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Housekeeping-5.jpg&quot;,&quot;id&quot;:&quot;1018&quot;,&quot;title&quot;:&quot;Housekeeping 5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

In other words &#8211; when your users get freshly wiped PC, they&#8217;ll get an impression it&#8217;s spanking new, yet customised by their great IT department. It makes a difference, a huge difference to the end user.

# Test the image within VM

That&#8217;s a great moment, isn&#8217;t it? I&#8217;m always excited when testing the new image and see how all the work turns out. But before deploying it to the actual physical machine, it&#8217;s worth to test it within the actual VM.

Why? Because due to checkpoints, it&#8217;s extremely easy to roll back the deployment, amend the setting and deploy once again. It&#8217;s a real time saver. Therefore, do not expect the image to be ready after the first time. My first attempt took about a week to get fully polished and prepared image.

## Take the checkpoint now

Remember: if there&#8217;s something you don&#8217;t like after the deployment or test, you can always roll back, change the image and try once again (and don&#8217;t forget to take another checkpoint).

You can repeat that process until you&#8217;re fully satisfied with the result.

## Sysprep &#8211; the tool for generalization

Generalise makes the image truly hardware neutral. It removes all installed drivers, Windows activation, unique installation details for the VM &#8211; it then allows computer to install completely fresh and setup itself for the new installation or another computer.

To generalise Windows installation you&#8217;d like to:

  1. Open elevated command prompt
  2. Type: **CD C:\Windows\System32\Sysprep**
  3. Type: **SYSPREP /GENERALIZE /OOBE /REBOOT /UNATTEND:<path to your unattended file> **We call sysprep tool to generalise the installation, set Windows into OOBE mode, use answer file and restart the PC.
  4. Wait until the sysprep process is done and PC restart.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-774-11-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Sysprep-1.jpg&quot;,&quot;id&quot;:&quot;1020&quot;,&quot;title&quot;:&quot;Sysprep 1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Sysprep-2.jpg&quot;,&quot;id&quot;:&quot;1021&quot;,&quot;title&quot;:&quot;Sysprep 2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Sysprep-3.jpg&quot;,&quot;id&quot;:&quot;1022&quot;,&quot;title&quot;:&quot;Sysprep 3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/04\/Sysprep-4.jpg&quot;,&quot;id&quot;:&quot;1023&quot;,&quot;title&quot;:&quot;Sysprep 4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

Once completed, VM should restart a few times and &#8211; if everything goes well &#8211; bring you to the login screen. You&#8217;d like then to review all your customisation settings, make sure that passwords are not saved etc. Remember that whatever you can see after the first login, will be available to any new user using the computer.

# Capturing and deploying the image

Once your VM is up to the standard it&#8217;s time to send it to the actual hardware and see if everything still works + all drivers are working.

However, it&#8217;s beyond this article as there are many tools which can be used. If you&#8217;ve got any favourite tools which can get the job done &#8211; let me know!

# A few tips:

  * Always take the checkpoint before syspreping. Windows has a sysprep counter, and after generalising an image few times it will not allow you to generalise it another! This means you&#8217;d need to create another image from the scratch. On the other hand, if you&#8217;ve got a VM with checkpoint just before sysprep, you can always come back to it, amend it, checkpoint it and sysprep again, unlimited number of times.
  * If there are too many checkpoints (and not enough hard disk space), you can always remove all of them and create a new one.
  * Creating the image is a long process &#8211; take your time!

Main photo by: https://unsplash.com/@rawpixel

 [1]: https://kamilpro.com/wp-content/uploads/2017/04/Windows_10_pro.xml
