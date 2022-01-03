# Mastering PNPUTIL – how to add drivers to the Windows image

Drivers &#8211; that&#8217;s what makes a harmony between hardware and software. Appropriate&nbsp;drivers can make a huge difference between computer&#8217;s performance, stability and even temperature. Therefore it&#8217;s super important to make sure you&#8217;ve got up-to-date files included with your image or Windows installation.

We are going to review PNPUTIL tool &#8211; the versatile tool for handling drivers within the Windows.

# Why do you need PNPUTIL?

Yes, why bother with some command line tool (PNPUTIL is command line only) when there are dedicated installers and device manager? Not to&nbsp;mention that Windows update can download pretty much every single driver missing from the Internet.

Well, it&#8217;s not that wonderful when there is a need to prepare a couple of dozens of computers.

While all sort of installers need hardware to be present during the installation, PNPUTIL doesn&#8217;t. In fact, as long as you can provide driver files &#8211; *.inf&nbsp;to be more specific &#8211; you can add them to the image, and Windows will use them when it has detected the hardware.

That allows you to supply drivers before installing Windows on the target computer, which saves time after deployment. Drivers will be therefore installed automatically during Windows installation &#8211; fully automatically.

# What is the Driver Store

Windows needs to store all the drivers in one location. Whenever you add, install, update or remove drivers, the OS will use the Driver Store as well as Windows Update to make your devices work properly.

Take any driver, and you&#8217;ll find (among other files) at least one *.inf file. Windows will use the file to install the driver, using reference in the file to make sure that everything has been installed properly.

Another important task of adding new drivers to the driver store is security check of the file, making sure that the driver hasn&#8217;t been changed since developer released it.

You can find driver store in:&nbsp;C:\Windows\System32\DriverStore.

# Adding vs Installing

While these words are usually used interchangeably, they have a different meaning in Microsoft terminology.

Adding means copying the driver and associated with it files to the Driver Store. During that process, all files will be checked for integrity and security. Adding the driver will not make your device working straight away, Windows will need to install it.

Installing means using the&nbsp;_added_ driver for the actual hardware. Whenever computer detects a new hardware connected, it will go through the Driver Store/Windows Update and set up the device, provided there is a matching driver available.

You can think about it like having spices in your kitchen. When you go to shops you buy pepper, salt, sage, paprika&nbsp;and rosemary. This can be compared to downloading the drivers. Then you unpack these spices and put them in your cupboard. This is adding drivers to the store. Finally, you cook chicken and realise the salt, pepper and paprika will be necessary for it, so you take these spices out of the cupboard and season your chicken. This is installing the drivers.

I&#8217;m getting hungry now, so let&#8217;s better move on to the next step.

# PNPUTIL syntax

There are only 4 main syntaxes in PNPUTIL, which allow you to add, remove, export and list all the drivers currently in the driver store.

When you&#8217;ll open the command prompt, type PNPUTIL and see:

<pre><strong>/add-driver</strong> &lt;filename.inf | *.inf&gt; [/subdirs] [/install]

Add driver package(s) into the driver store.
 /subdirs - traverse sub directories for driver packages.
 /install - install/update drivers on any matching devices.

<strong>/delete-driver</strong> &lt;oem#.inf&gt; [/force]

Delete driver package from the driver store.
 /force - delete driver package even when it is in use by devices.

<strong>/export-driver</strong> &lt;oem#.inf | *&gt; &lt;target directory&gt;

Export driver package(s) from the driver store into a target directory.

<strong>/enum-drivers</strong>

Enumerate all 3rd party driver packages in the driver store.</pre>

## Add

**/add-driver**&nbsp;option allows you to add a driver to the store. If you provide specific INF file, only one driver will be copied, however using a wildcard *.inf will install all drivers within the current directory. You can also use&nbsp;**/subdirs** to do bulk load to all files in the child folders. &nbsp;Last option&nbsp;**/install** will install or update the drivers, however, Windows should perform it automatically once the devices have been detected.

## Delete

Sometimes you&#8217;d like to remove a certain device from the device manager, but it keeps coming back with maniac&#8217;s resistance. The reason is the device is being only installed, but the actual files are present in the driver store.

That&#8217;s when you&#8217;d like to use **/delete-driver&nbsp;**with&nbsp;&nbsp;/**force&nbsp;**option. It will uninstall and purge the files from the driver store, even if the device is currently used.

## Export

**/export-driver** is especially useful when you need to copy drivers from one computer to another, while not having the original driver source available. PNPUTIL will create an exact copy of the driver being used in the current system, which you&#8217;ll be able to add to another computer later on.

## Enumerate

**/enum-drivers** will show you the list of all drivers in currently being kept in the driver store. You&#8217;ll be able to find the type, vendor, version and name of the driver.

All drivers will have the OemXX.inf naming convention, while XX is the unique number of the each driver. You can use that tool to identify drivers which you&#8217;d like to remove or export.

# Hints/More stuff to read

  * You can use command **PNPUTIL /enum-drivers > C:\temp\drivers.txt** to create a file with the list of all current drivers.
  * You can find names of drivers in Device Manager. Simply click the device you&#8217;d need the drivers for, pick details tab and choose &#8220;inf name&#8221; from the drop down menu.
  * I did write more about creating the Windows 10 image here &#8211;&nbsp;<a href="https://kamilpro.com/prepare-windows-10-1607-image/" target="_blank" rel="noopener noreferrer">https://kamilpro.com/prepare-windows-10-1607-image/</a>.
