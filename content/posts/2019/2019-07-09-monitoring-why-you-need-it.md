---
title: Monitoring – why you need it
author: Kamil

date: 2019-07-09T06:45:00+00:00
url: /monitoring-why-you-need-it/
featuredImage: /wp-content/uploads/2019/07/artur-luczka-qhG6Xde9lCQ-unsplash.jpg
wp_last_modified_info:
  - 8th July 2019 @ 7:45 pm
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - IT
tags:
  - monitoring
  - nagios
  - readings
  - zabbix

---
It&#8217;s Friday. 5.20PM. So close to the long weekend. Monday is bank holiday &#8211; thus three days off. Very happy Friday.

You&#8217;re going to the server room to check some minor details. Upon opening the main door, you can feel that cold breeze and characteristic noise of hundreds of spinning fans. You quickly skim through all flashy lights, as it&#8217;s a good habit.

And then you notice there&#8217;s something wrong. An orange light &#8211; why is there an orange light? Hopefully it&#8217;s nothing serious. It&#8217;s just above one of the disks. Fuck, it&#8217;s the main file server &#8211; the one that was meant to be virtulised a long time ago, but for some reason it haven&#8217;t happened for over two years.

All finance data is on it. And HR. Not to mention sales and marketing guys. It looks like it already picked a hot spare.

Now it&#8217;s coming a reactive part of checking logs, grabbing part number, calling vendor. Depending on luck, you might leave office within one hour from discovery, but until the part arrives, the faulty disk is replaced and data copied &#8211; it&#8217;s kind of a bad dream.

## When did it happen?

Problem with manual checks is that they are only valid during the time of the check &#8211; all you have is a snapshot of this particular moment in time. If something requiring your attention breaks the moment you leave the premises &#8211; it won&#8217;t be discovered until the next check. Unless, if it&#8217;s very critical problem like whole server going down &#8211; then you&#8217;ll find out through you customers/users.

If your customers are telling you that they can&#8217;t access file shares and the Internet, because the only domain controller is down, then **this is a very bad position to be in**.

Therefore having a monitoring in place, even if it&#8217;s set as rarely as once per hour*, can give you a certain edge.

## Why did it happen?

Depending on how particular system is being used, what level of logging is enabled etc. there might be different number of logs to go through.

The problem is that after checking post fact, it might take time to drill through all the logs to find out which exact part and why it failed. It might be required by vendor&#8217;s support. It might as well be required by audit trail why certain service wasn&#8217;t delivered in accordance with SLA. The point is simple &#8211; going through logs is not fun.

What if instead &#8211; you had monitoring in place, which would answer that questions straight away?

On top of knowing (if you configure it so) almost instantly when something breaks, you will also know why it happened. Monitoring works pretty much like: retrieve status of the device, check what the status is, if it&#8217;s not in OK state, shoot out alarm.

It obviously depends of how well integrated it is, how documented etc. but the rule of thumb is &#8211; if you can retrieve the status, you&#8217;re able to monitor it.

## If it&#8217;s that great, can I stop performing manual checks all together?

This arguable question, and I say that &#8211; even when you trust 100% in your monitoring system, it&#8217;s still worth doing manual checks from time to time.

The reason is that, there&#8217;s always a tiny chance that with some firmware upgrade, or some completely new piece of kit, there was a change which changes how readings are done. It also depends on system you use &#8211; if it&#8217;s open source or commercial product.

Typical rules apply &#8211; free software requires more of your input, while commercial might be good to go out of the box.

## How is monitoring performed?

Obviously, the monitoring system will not be able to do much if it can&#8217;t authenticate to any of the systems. There are various ways, some most popular include:

  * SNMP &#8211; it&#8217;s a standard which has been around for a long time now. It&#8217;s very possible that your printer, router, server etc. supports it. This can be used for hardware as well as OS monitoring.
  * Agent &#8211; you install an agent compatible with your monitoring system, and that&#8217;s the way it can get reading from.
  * API &#8211; used with scripted checks. APIs are usually well documented, thus creating a script around it is pretty straight forward.
  * WMI &#8211; it used to be one way of monitoring HP Generation 5-9 servers. As WMI can give you a lot of granularity, there&#8217;s a lot of information which is hidden from user in graphical tools. You end up really scripting around it.

If there&#8217;s piece of business important kit that can&#8217;t be monitored, I would really question the sense of using it in production.

## Ok &#8211; this all sounds great &#8211; but must be expensive

Not at all. I mean it depends on your choice.

If at the moment you don’t have any monitoring in place, then there are two free solutions which you can use: [Nagios][1] and [Zabbix][2]

### Nagios Core

Nagios Core is an open source version of Nagios. It’s so open source that you need build it from the source code. There are also Docker containers available, but I haven’t used them personally.

The main advantage or disadvantage of Nagios is that once installed, you configure it enterly through the text files. It also gives you a lot of freedom of how these files are organised grouped etc. Your role is to point it where files are, and however you configure them is up to you.

You can have one massive file for all your computers, network switches etc. Or you can create a folder structure Per Site/Office Location/Particular location. Whatever floats your boat.

Once configured, all the readings are available through the website.

Nagios has also been on the market for a long time, thus there’s a strong community full of ready to go monitoring solutions. Nothing obviously stops you from creating your own scripts.

I’d say, if you like working in command line/code and have plenty of space of how to organise a solution, then Nagios Core will be appealing.

### Zabbix

Zabbix on the other hand is much more easy to deploy than Nagios. And once installed, it’s configured entirely through the website.

The very strong point of Zabbix are its discovery templates &#8211; e.g. if you point it to the switch, it will start monitoring all ports on it. If you point it to the ESXi server, it will start monitoring hardware through it, and do readings of all VMs in there. You can also install a Zabbix agent on monitored operating systems.

There are also community plugins available for Zabbix, thus extending its capabilities is easy.

You can either install Zabbix on Linux, or download a pre-configured appliance &#8211; this is really easy way to test the software.

## So where should you install the monitoring?

It should really be on its own physical box. If you have it as VM, and hyper visor dies &#8211; then you loose monitoring. Rather than knowing that hyper-visor is down.

For convenience, you can configure the monitoring solution as VM, and once happy with it, V2P it to the actual physical box.

And when configuring monitoring, you must always do it from monitoring’s point of view. E.g. if the monitoring is in network, can it get across to the other subnet? Where is the gateway, so that it can check if the Internet is working? Will be able to notify you if the Internet is actually down?

## Ready for a better night&#8217;s sleep?

I hope it&#8217;s a no brainer to you now that monitoring should be first place for your work.

Having a peace of mind that stuff you&#8217;re responsible for is being constantly check for it&#8217;s condition, can really improve your overall stress level.

And hey, you&#8217;ve also got more time as manual checks are not needed any more, what are you going to do about it?

*Depending on <g class="gr_ gr\_6 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Grammar only-ins replaceWithoutSep" id="6" data-gr-id="6">software</g> you use, you might be able to pull data once every five seconds. The point here is that there&#8217;s something in the background that does these checks <g class="gr_ gr\_8 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Punctuation only-del replaceWithoutSep" id="8" data-gr-id="8">constantly,</g> so that you don&#8217;t <g class="gr_ gr\_7 gr-alert gr\_gramm gr\_inline\_cards gr\_run\_anim Grammar only-ins replaceWithoutSep" id="7" data-gr-id="7">need</g> worry about it. Imagine knowing that something broke within one hour, rather than one month.

Photo by [Artur Łuczka][3] on [Unsplash][4]

 [1]: https://www.nagios.com/products/nagios-core/
 [2]: https://www.zabbix.com/
 [3]: https://unsplash.com/@artur_luczka?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
 [4]: https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText
