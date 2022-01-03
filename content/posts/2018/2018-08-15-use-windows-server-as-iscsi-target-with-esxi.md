---
title: Use Windows Server as iSCSI target with ESXi
author: Kamil

date: 2018-08-15T17:54:47+00:00
url: /use-windows-server-as-iscsi-target-with-esxi/
featuredImage: /wp-content/uploads/2018/08/leone-venter-469710-unsplash.jpg
categories:
  - ESXi
  - IT
  - Server
  - Uncategorised
  - Windows
tags:
  - 2012r2
  - esxi
  - iscsi
  - lun
  - powershell
  - server

---
The task: to move a virtual machine from one ESXi to another. The ESXis are on two completely separate networks. VM size is 5.5 TB, with one file being 5TB.

The problem: VM cannot be started due to maxed out resources. There&#8217;s no NAS or any other means of storage where the files could be transferred to. From previous experience, I knew that transfer over SSH or USB would be well too slow, as the task was time critical. There wasn&#8217;t also a big enough single hard disk which could be used to copy the files onto it.

My proposed solution: Use any spare desktop/server and load it with hard drives, so that all of them can create spanned virtual disk big enough to accommodate files from ESXi, install at least server 2012r2 on it and configure as an ISCSI target.

As you can see above, it was quite an unusual task with quite unusual conditions.

Requirements:

  * Windows box: 
      * Spare computer
      * 1HDD for OS
      * Number of HDDs, which summarised capacity would be enough to accommodate files from ESXi. HDDs might be a different size
      * One network adapter
  * Source and destination ESXi: 
      * at least one spare network port which can be configured as uplink
      * Possibility to enable iSCSI (I was working on ESXi 6.5)

Let&#8217;s start preparation!



## Prepare Windows server

I take it you&#8217;ve prepared a fresh installation of Windows server, if not go ahead and do it. I&#8217;ve used 2012 R2 for the purpose of the task. Don&#8217;t forget to activate it.

Another important step is to configure network card you&#8217;re going to use with static IP &#8211; as we will tell ESXi to connect to the server over that IP. All you need is <g class="gr_ gr\_5 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="5" data-gr-id="5">an </g>IP address and subnet mask.

Unless you&#8217;re willing to connect over the network then obviously gateway would be needed. However, it is strongly recommended to use iSCSI on <g class="gr_ gr\_6 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="6" data-gr-id="6">the </g>isolated network &#8211; due to security, performance and stability reasons.

### Create spanned volume

Once you&#8217;ve got your server ready for a spin &#8211; the first step is to create a spanned volume &#8211; we will simply use all spare hard drives and create one massive disk out of them. I&#8217;ve used 1TB, 3TB and 4TB disks, which <g class="gr_ gr\_4 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del" id="4" data-gr-id="4">gave</g> me some 8TB of space altogether.

You&#8217;d like to press CTRL + X and bring disk manager. Then right-click on each of the disks and initialize it, then bring them online. In my example, the disks are numbered 1,2 and 3.

Next step is the actual creation of the Spanned Volume, you click on any of the spare disks, and choose &#8220;New Spanned Volume&#8221;, click next and on &#8220;Select Disks&#8221; page Add all disks, so that total volume size shows summed storage space from all of them. You assign any letter drive you want, give any name (I called mine BIG) and make sure to tick Perform <g class="gr_ gr\_4 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="4" data-gr-id="4">a </g>quick format.  
<figure class="wp-block-image">

[<img loading="lazy" width="1844" height="962" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2018/08/SpannedVolume.gif?fit=1844%2C962&ssl=1" alt="" class="wp-image-1276" />][1]<figcaption>Spanned volume creation</figcaption></figure> 

That&#8217;s it, once you click finish you will see the colour of disks changing to purple-ish, and all three disks will have the same partition name with <g class="gr_ gr\_6 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins replaceWithoutSep" id="6" data-gr-id="6">the </g>same letter assigned &#8211; in my case &#8220;BIG (F:). You can go to my computer and check yourself that this &#8220;disk&#8221; in fact exists.

### Install iSCSI Target Server role

Now it&#8217;s time to add iSCSI role so that we can create the iSCSI disk and attach it to ESXi.  iSCSI is a technology which allows you to <g class="gr_ gr\_4 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del multiReplace" id="4" data-gr-id="4">attach</g> a network drive to the PC and use it as it was a local disk.

Bring up Add roles and features wizard, and on &#8220;Server roles&#8221; tab check: File and Storage Service > File and iSCSI Services > iSCSI Target Server, accept the default and go ahead with the installation.<figure class="wp-block-image">

[<img loading="lazy" width="1844" height="962" src="https://i2.wp.com/kamilpro.com/wp-content/uploads/2018/08/iSCSI-role-installation.gif?fit=1844%2C962&ssl=1" alt="" class="wp-image-1277" />][2]<figcaption>iSCSI target role installation</figcaption></figure> 

You can achieve the same result using Windows Powershell by using the command:

<pre class="wp-block-code"><code>Install-WindowsFeature -Name FS-iSCSITarget-Server</code></pre>

### Create iSCSI disk and target

Final step of our configuration is the actual creation of iSCSI virtual disk.

Head on to the server manager, go to the Files and storage pane > choose iSCSI tab. Click on TASKS and choose &#8220;New iSCSI Virtual Disk&#8221; 

On the new windows that shows up, choose the location on your newly created spanned disk (F: in my case), provide any meaningful name. On the disk size, choose max available space (7.81TB in my case) and choose the <g class="gr_ gr\_8 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Grammar only-ins replaceWithoutSep" id="8" data-gr-id="8">disk</g> <g class="gr_ gr\_9 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins replaceWithoutSep" id="9" data-gr-id="9">to </g>grow dynamically.

Since <g class="gr_ gr\_4 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar multiReplace" id="4" data-gr-id="4">there are</g> no iSCSI targets configured yet, choose to create a new one, give it any name and choose to identify by the IP address (in my case 10.4.1.5<g class="gr_ gr\_5 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Style multiReplace" id="5" data-gr-id="5">) .</g>

There&#8217;s no need for authentication here, so just move on next and finish the wizard, which will end up in creating the iSCSI disk with <g class="gr_ gr\_3 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="3" data-gr-id="3">the </g>associated target.<figure class="wp-block-image">

[<img loading="lazy" width="1844" height="962" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/iSCSI-virtual-disk.gif?fit=1844%2C962&ssl=1" alt="" class="wp-image-1278" />][3]<figcaption>iSCSI virtual disk creation</figcaption></figure> 

Powershell way:

<pre class="wp-block-code"><code>#Create iSCSI disk
New-IscsiVirtualDisk -Path F:\iSCSIVirtualDisks\iSCSI1.vhdx -Size 7.81TB

#Create iSCSI target
New-IscsiServerTarget -TargetName target1 -InitiatorIds IPAddress:10.4.1.5

#Assign iSCSI disk to the target
Add-IscsiVirtualDiskTargetMapping -TargetName target1 -Path F:\iSCSIVirtualDisks\iSCSI1.vhdx</code></pre>

At this point<g class="gr_ gr\_6 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Punctuation only-ins replaceWithoutSep" id="6" data-gr-id="6">,</g> you&#8217;re ready to plug your Windows Server directly to the spare ESXi network port. We&#8217;re going to do some configuration on ESXi now.

## Configure ESXi

## Create VMKernel NIC  


To start with I wanted to thanks Andy who helped me with VMkernel NIC configuration &#8211; it would take me much longer to figure out this myself! Drop me the line if you&#8217;re reading <g class="gr_ gr\_106 gr-alert gr\_spell gr\_inline\_cards gr\_run\_anim ContextualSpelling ins-del multiReplace" id="106" data-gr-id="106">thi</g>!

In order to connect <g class="gr_ gr\_3 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins replaceWithoutSep" id="3" data-gr-id="3">the </g>iSCSI disk to ESXi we need network port configured, so if you haven&#8217;t yet plugged in your just prepared Windows server to the hypervisor &#8211; go and do it now.

We need to configure Virtual switch, port group and finally VMKernal NIC

Click on Networking pane and choose **Virtual Switches** tab. Down there choose an option to add a new switch. In the new Window that pops out give it a <g class="gr_ gr\_5 gr-alert sel gr\_spell gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear ContextualSpelling ins-del multiReplace" id="5" data-gr-id="5">meaningful</g> name and choose the right uplink (port on NIC)

Next click on **Port groups** tab and choose ato add port group, give it meaningful name, choosing the switch you&#8217;ve just created.

The final step is to click on **VMkernel NICs** tab and add the new VMkernel NIC. Choose port group you&#8217;ve created in the previous step, then choose that IP should be static and give it IP address in the same subnet as your Windows machine. 

I configured my Windows machine to be 10.4.1.5/24 and VMkernel NIC to be 10.4.1.4/24<figure class="wp-block-image">

[<img loading="lazy" width="1844" height="962" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/ESXi-VMKernel-NIC.gif?fit=1844%2C962&ssl=1" alt="" class="wp-image-1282" />][4]<figcaption>Create VMKernel NIC</figcaption></figure> 

## Add iSCSI disk and configure it as datastore

We&#8217;re coming to the very end of the configuration &#8211; everything has been configured and the last step is to create a datastore on the iSCSI disk.

Click on storage pane and choose **Adapters** and click on **Configure iSCSI,** on the new windows choose to enable iSCSI, and type in the dynamic address field type in the IP of Windows machine, confirm.

After that head on to **Datastores** tab and choose **Add Datastore** to create new VMFS datastore, point it to the Windows datastore and <g class="gr_ gr\_4 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Style multiReplace" id="4" data-gr-id="4">create your</g> datastore.

That&#8217;s it, you can use newly created datastore as it was the <g class="gr_ gr\_3 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar multiReplace" id="3" data-gr-id="3">locally</g> attached disk.  
<figure class="wp-block-image">

[<img loading="lazy" width="1844" height="962" src="https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/ESXi-add-iSCSI-and-create-datastore.gif?fit=1844%2C962&ssl=1" alt="" class="wp-image-1283" />][5]<figcaption>ESXi adding iSCSI disk and creating datastore</figcaption></figure> 

## You can obviously use this configuration in any different scenarios

Since having <g class="gr_ gr\_3 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="3" data-gr-id="3">the </g>possibility of using Windows as <g class="gr_ gr\_4 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Grammar only-ins doubleReplace replaceWithoutSep" id="4" data-gr-id="4">iSCSI</g> target is not that very popular, I think using ESXi with attached iSCSI datastores can be an easy and cheap way of making more space for your VMs.

Yet this non-typical topic allowed me to complete this very strange situation within <g class="gr_ gr\_5 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins replaceWithoutSep" id="5" data-gr-id="5">a </g>relatively short and easy way.

Have you found yourself? Do you know any other means of achieving <g class="gr_ gr\_10 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins replaceWithoutSep" id="10" data-gr-id="10">the </g>same goals? Please let me know!

## How&#8217;s my story ended?

After connecting Windows Server to vSphere with <g class="gr_ gr\_6 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="6" data-gr-id="6">the </g>iSCSI disk, I was able to create the datastore. I then copied the massive VM to it and replugged Win server to the other ESXi server. Once connected and mounted the iSCSI disk &#8211; I was able to register the VM.

Hit the start button, choose the VM was copied and boom &#8211; I had a working VM directly from the external disks. From that point on I was able to install <g class="gr_ gr\_90 gr-alert sel gr\_gramm gr\_replaced gr\_inline\_cards gr\_disable\_anim\_appear Grammar only-ins doubleReplace replaceWithoutSep" id="90" data-gr-id="90">the </g>backup agent on it and back it up directly to the tape.

Problem solved! Yay!

 [1]: https://i2.wp.com/kamilpro.com/wp-content/uploads/2018/08/SpannedVolume.gif?fit=1844%2C962&ssl=1
 [2]: https://i2.wp.com/kamilpro.com/wp-content/uploads/2018/08/iSCSI-role-installation.gif?fit=1844%2C962&ssl=1
 [3]: https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/iSCSI-virtual-disk.gif?fit=1844%2C962&ssl=1
 [4]: https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/ESXi-VMKernel-NIC.gif?fit=1844%2C962&ssl=1
 [5]: https://i0.wp.com/kamilpro.com/wp-content/uploads/2018/08/ESXi-add-iSCSI-and-create-datastore.gif?fit=1844%2C962&ssl=1
