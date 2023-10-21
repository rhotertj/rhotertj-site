+++
title = 'Discovering EOL behavior of SD cards'
date = 2023-10-21T12:33:35+02:00
draft = false
+++

This website, my cloud and my darts club website are hosted on a Raspberry Pi. This went well for several years until it suddenly died last week.
After ordering a mini hdmi cable to see what was going on, I learned that my machine wouldn't boot because of a file system _inconsistency_.

```
/dev/mmcblk0p2: UNEXPECTED INCONSISTENCY; RUN fsck MANUALLY.  
The root filesystem is currently mounted in read-only mode.  
A maintenence shell will now be started.  
After performing system maintenance, pleass CONTROL-D to terminate the maintenance shell and restart the system.
```

First lesson learned: 
Always have a way to access your devices even if they are not available through ssh.

After some googling, it turned out to be the death of my SD card. I do store all important data on an external hard drive, which is backed up frequently, so I was not worried about data loss, but rather the hassle of reinstalling the entire operating system. Although most of it is Docker containers, with definitions stored in a remote git repository, getting it all up and running would probably take an afternoon better spent learning something like NixOS.

I did get lucky though: [As it turns out](https://raspberrypi.stackexchange.com/a/88089), SD cards that are reaching their end of life protect themselves by going read-only.
This is part of a clever wear leveling system by the manufacturer and allows for copying the data to a fresh medium. Thank you SanDisk! 
As luck would have it, my laptop has an SD card reader. I could simply use `dd` to get an image of the now read-only SD card with my Ubuntu Server 20.10 installation, and copy it onto my new SD card. 

After that, I could simply reboot, update all packages and was back online!
Crisis averted!

