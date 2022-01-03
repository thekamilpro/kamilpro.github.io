# Add any drivers to Windows – on example of GPU drivers

Today we are going to inject drivers into the Windows image with the help of built-in tools in Windows.

# What do you need?

Just a few things:

  * Any hardware to which you’d like drivers to be added to
  * Reference computer with the hardware attached to it
  * Drivers
  * Windows image which is going to be deployed

I think there’s not much to explain, so let’s analyse the actual process.

# What we are going to do?

The problem with drivers is that they are often provided as one installation file which contains drivers with self installing package, which is convenient for a single user, but not for us, sysadmins.

I’ll show you on the example of GeForce drivers how to get and add drivers for your Windows image.

Therefore, we’re going to play smart, as always:

  1. Obtain the drivers
  2. Extract the drivers so that we have access to *.inf files
  3. Install the drivers to the actual Windows installation
  4. Export installed drivers
  5. Add exported drivers to the image

Sounds good? Ready, steady, ENTER!

# On your reference computer

Make sure that you have the relevant OS version. If you aim to add drivers to Windows 10, have a Windows 10 ready on your reference computer. It might be possible to have a cross Windows version drivers, but you’d need to test it thoroughly. I’ve seen many misbehaving computers just because of wrong drivers used with them.

You must plug-in the actual hardware to your reference computer. Very often single installation file contains multiple drivers used with multiple devices, and Windows will pick and install only the necessary ones.

## Obtaining the drivers

The easiest way to get latest drivers is to get them from the manufacturer&#8217;s website, for your convenience I put links to NVidia, AMD and Intel drivers websites:

NVidia GeForce &#8211; <http://www.nvidia.com/Download/index.aspx>

AMD Radeon &#8211; <http://support.amd.com/en-us/download>

Intel &#8211; <https://downloadcenter.intel.com/product/80939/Graphics-Drivers>

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-12-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Drivers-Download-1.jpg&quot;,&quot;id&quot;:&quot;1102&quot;,&quot;title&quot;:&quot;Drivers-Download-1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Drivers-Download-2.jpg&quot;,&quot;id&quot;:&quot;1103&quot;,&quot;title&quot;:&quot;Drivers-Download-2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

Make sure you pick the appropriate Windows and hardware versions.

## Extract *.INF files

Now we need to do something the usual user never does &#8211; intercept \*.inf files which will allow us to add the drivers to the image. If your hardware manufacturer was nice to sysadmins  &#8211; by that I mean they provided already the \*.inf files then simply skip this step.

Otherwise, there are 2 main ways of doing so.

### Get drivers extracted during the installation

If, like in my example, drivers came prepackaged, you run the installation file. On the first screen I can see (and choose) where the installation files are going to be extracted.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-13-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-1.jpg&quot;,&quot;id&quot;:&quot;1105&quot;,&quot;title&quot;:&quot;Driver-Instal-1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-2.jpg&quot;,&quot;id&quot;:&quot;1106&quot;,&quot;title&quot;:&quot;Driver-Instal-2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

I confirm where they’re going to be extracted, hit OK and once the extraction process has been completed &#8211; **I don’t proceed with installation**, not yet! Just let it be for time being.

Instead, I go to location where drivers were extracted and copy these files somewhere else.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-14-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Extract-1.jpg&quot;,&quot;id&quot;:&quot;1107&quot;,&quot;title&quot;:&quot;Driver-Extract-1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

### Extract the files yourself

Sometimes you’re not offered with the extraction option and might need to do it yourself. For that purpose you’d need to use an archiver manager like 7-zip. Once installed, use 7-zip to open the installation file, and if you can find *.inf files inside, extract everything where needed.

7-zip download page: <http://www.7-zip.org/download.html>

### If the above ways failed

Then manufacturer for some reason made it difficult to you, but we still have a few ways of tracking where the actual files went.

You can often find files in:

  * Root of system drive
  * Root of other partition
  * In Windows\Temp directory

While browsing through the folders set your folder view to details and make the most recent files appear on top. You can even use Windows Search and specify last modification time to when you start installation and search for *.inf files.

Finally, you can use resource monitor to track where installation files makes changes to the hard drive.

You can also, export already installed drivers. While it’s really convenient, it’s not always that easy to find all drivers once deployed. I always would like you to have a full picture of the whole process.

## Install the drivers

Now we can carry on with installation. But before doing so, I’d like you to open C:\Windows\System32\DriverStore\FileRepository folder and sort folders by “Date modified”, with the most recent on top. Note the time when you start the installation.

Once the installation of your drivers is commenced, you will notice as new folders are being created &#8211; these are drivers needed by your hardware. You can open these folders and see that there are plenty of files.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-15-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-6.jpg&quot;,&quot;id&quot;:&quot;1110&quot;,&quot;title&quot;:&quot;Driver-Instal-6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-5.jpg&quot;,&quot;id&quot;:&quot;1109&quot;,&quot;title&quot;:&quot;Driver-Instal-5&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-7.jpg&quot;,&quot;id&quot;:&quot;1111&quot;,&quot;title&quot;:&quot;Driver-Instal-7&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Driver-Instal-8.jpg&quot;,&quot;id&quot;:&quot;1112&quot;,&quot;title&quot;:&quot;Driver-Instal-8&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

You can notice that those folders have a naming convention &#8211; <span style="color: #339966;">name.inf</span><span style="color: #0000ff;">_architecture_</span>randomnumbers. You’re after the green part &#8211; name.inf That’s the name of the *.inf file you’d like to add to the image from your source folder &#8211; **I’d like you to take a note of the green part of the name**, as this will be essential to the next step.

I started installation of driver just after 22:20 on 24th of May, and as you can see I’ve highlighted the folders created just after that time. The files I’m going to look for are:

  * Nv_dispi.inf
  * Basicrender.inf
  * Nvstusb.inf
  * Nvhda.inf

If you’ve read my post about <a href="https://kamilpro.com/mastering-pnputil-add-drivers-windows-image/" target="_blank" rel="noopener noreferrer">Mastering PNPUTIL</a>, then you’ll know how to use this tool to add drivers to your image. Otherwise, stay with me.

# On your Windows image

Let’s armour now the image in the right drivers we’ve just acquired. You will obviously need to make them available to the image, either by shared folder or copy/paste. I simply copied all installation files to C:\Temp\NVidia folder. Keep this folder open as we’re going to need it very soon. Like in 2 paragraphs soon.

I’m going to show you now 2 ways of installing drivers &#8211; with GUI and command line.

On the side note: if you’d like to confirm the files are being added to the driver store, simply open another Windows Explorer window and head towards C:\Windows\System32\DriverStore\FileRepository like you did on your physical machine as you did before.

### Add drivers with GUI

While still in the folder with the drivers, simply type in the Search Box the filename of the first *.inf file of which I’ve asked you earlier on. In my case it is: Nv_dispi.inf. It should show up only one file. Now you’d like to right click the file, and choose Install.

Once completed you should receive a message stating that operation was successful.

Repeat the process for all drivers you need and that’s it &#8211; your image will come with drivers and system will install them automatically when hardware is detected.

You can confirm it by either going to FileRepository folder or Issuing **pnputil.exe /e** command in command prompt.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-16-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-1.jpg&quot;,&quot;id&quot;:&quot;1114&quot;,&quot;title&quot;:&quot;Add-drivers-1&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-2.jpg&quot;,&quot;id&quot;:&quot;1115&quot;,&quot;title&quot;:&quot;Add-drivers-2&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-3.jpg&quot;,&quot;id&quot;:&quot;1116&quot;,&quot;title&quot;:&quot;Add-drivers-3&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-4.jpg&quot;,&quot;id&quot;:&quot;1117&quot;,&quot;title&quot;:&quot;Add-drivers-4&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

### Add drivers with Command prompt &#8211; PNPUTIL

This way is slightly more complicated, nevertheless, let’s have a fun!

  1. Open elevated command prompt and use change directory to where your drivers are. In my case I issued the command **CD C:\Temp\Nvidia**
  2. Now type **pnputil -a filename.inf /subdirs.** In my case, it was **pnputil -a nvstusb.inf /subdir** This will tell PNPUTIL to search for this particular file in all sub directories of the current folder.
  3. You should receive a message stating something like:Microsoft PnP UtilityProcessing inf : NV3DVisionUSB.Driver\nvstusb.inf  
    Driver package added successfully.  
    Published name : oem20.infTotal attempted: 1  
    Number successfully imported: 1
  4. Repeat step 2 for all the files needed.

That’s pretty much it.

<p class="jetpack-slideshow-noscript robots-nocontent">
  This slideshow requires JavaScript.
</p>

<div id="gallery-1098-17-slideshow" class="slideshow-window jetpack-slideshow slideshow-black" data-trans="fade" data-autostart="1" data-gallery="[{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-7.jpg&quot;,&quot;id&quot;:&quot;1119&quot;,&quot;title&quot;:&quot;Add-drivers-7&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;},{&quot;src&quot;:&quot;https:\/\/kamilpro.com\/wp-content\/uploads\/2017\/05\/Add-drivers-6.jpg&quot;,&quot;id&quot;:&quot;1118&quot;,&quot;title&quot;:&quot;Add-drivers-6&quot;,&quot;alt&quot;:&quot;&quot;,&quot;caption&quot;:&quot;&quot;,&quot;itemprop&quot;:&quot;image&quot;}]" itemscope itemtype="https://schema.org/ImageGallery">
</div>

# Your image has just got better!

Deploying drivers had always been a mission before I’ve figured out the way I had shown you above. While there are some tools/scripts which can deliver the drivers in one way or another (and with the tools there is always a question of how reliable the tool is) having drivers injected to the image will always ensure the hardware is operational straight after deployment.

The 2 drawbacks of the whole process in the matter are: having a larger image and updating the drivers on the image. While the size and deployment time shouldn&#8217;t be a big deal nowadays, keeping up-to-date images is probably your responsibility after all. Not to mention the time saved for troubleshooting as this method works always, 100%, positive.

Did you enjoy reading the post or found it useful? I’m really keen to know! Let me know below in the comments section.

I’ve spent hours searching a way/tool capable of fully automatic drivers deployment to the computers of different hardware specs. I ended it up analysing and figuring out all this myself.

# Tips

  * If you can&#8217;t get the installation files of drivers, but you’ve got already a system where this hardware works, you can export by: 
      * open device manager identify the device you’d need the drivers for.
      * Double click on it, choose Details tab and choose “inf name” from property drop down menu, note down the OemXX.inf file name.
      * Create a folder, e.g. C:\Temp.
      * Then open elevated command prompt and type **pnputil /export-driver OemXX.inf C:\Temp **You can copy exported file to the missing computer and install them like we did before.
  * If you’ve added the wrong driver, use **PNPUTIL /delete-driver OemXX.inf /force** to remove it from the driver store.
  * If after the installations of the drivers you cannot find some folders in the installation files (e.g. basicrender.inf) then don’t worry &#8211; these files might be default for Windows and system might use the while installing new drivers.
  * Someone can say, that you could use PNPUTIL to add all files \*.INF to the driver store, but by the time when the actual process ends, you’ll notice that your driver store folder has swelled multiple times (in my case, that was 10GB after adding all \*.INF files from GeForce folder!), containing many drivers which you actually don’t need.

Main photo by [Jeff Sheldon][1].

 [1]: https://unsplash.com/@ugmonk

