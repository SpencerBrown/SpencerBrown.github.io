---
layout: post
title: "Dual booting OS X and Linux"
date: 2016-02-15
---

I've successfully set up my MacBook Pro Retina (mid-2012) with a clean install of OS X El Capitan, dual booting to Arch Linux.
Other blog posts, and the Arch Linux Wiki itself, are misleading, confusing, or leave out important parts.
Here's the straight information. This recipe should work (with some obvious adjustments) for other Mac computers or Linux distributions.

**Why would you want to dual boot?**

If your work on this machine is mostly OS X with a bit of Linux from time to time, it is best *not* to use dual boot.
Instead, set up Linux as a virtual machine under OS X. You can use the [free/open-source VirtualBox](http://virtualbox.org),
or the commercial [VMware Fusion](https://www.vmware.com/products/fusion) or [Parallels Desktop](http://www.parallels.com/products/desktop/).

If your work on this machine will be mainly Linux, it makes sense to set it up as a dual boot machine. 
When booted natively in Linux, you will get the best performance and simplest operation.

Why not make your machine completely Linux? Because you need OS X for support/maintenance and updates.

**You need the following:**

1. A Mac computer. It's best if this is an "extra" machine that you want to use mainly for Linux.
2. A decent Internet connection, as you will be downloading multiple gigabytes of code.
3. A USB flash drives that you can erase. It need to be at least 8GB, preferably 16GB. A USB3 flash drive will make this go much faster.

**We do this in five steps:**

1. Back up any data that you need
1. Clean install El Capitan
2. Prepare for Arch Linux install
3. Install Arch Linux
4. Set up dual boot

## Back up data

**All data on your Mac will be erased!** So back up whatever you need. Use Time Machine, copy the files, make an image copy, whatever works for you.

You can skip the "clean install El Capitan" step, and keep your data, but it's recommended to start fresh.

## Clean install El Capitan

This step is optional, but recommended, to start the life of your dual boot system with a clean OS X install.

#### Create El Capitan installation USB flash drive

1. Open the App Store on your Mac, go to the Purchased tab, and download OS X El Capitan. **warning** this is a 6GB download!
2. If you see it, cancel the dialog asking you to install.
3. Insert your USB flash drive. **warning** All data on this drive will be erased.
4. Open the Disk Utility application.
5. Navigate to your USB drive (not a partition) and click the Erase tab.
6. Make sure the format is "Mac OS Extended (Journaled)" and the name is "Untitled". Click Erase.
7. Open a Terminal window and run the following command (may take several minutes). Then use OS X to eject the flash drive, to ensure all data has been written. 

```bash
sudo /Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app --nointeraction
```

#### Boot from USB flash drive and clean install OS X El Capitan

1. On the Mac that you want to clean install, shutdown (power off), then insert the El Capitan install USB flash drive that you created in the previous section.
2. Hold down the Option key, then press and release Power. Continue to hold the Option key until you see some icons.
3. You should see two large icons: one for OS X, and one that probably says "EFI Boot" or "Install El Capitan". Use your arrow keys to navigate to the latter, then Enter.
4. You should see an "OS X Utilities" window. Run Disk Utility.
5. Select your main Mac hard drive (not a partition) and click Erase. **warning** This will erase all of your OS X data!
6. Close Disk Utility, and run "Install OS X" from the OS X Utilities menu.
7. Continue through the OS X install process, selecting options as you wish. Then shut down the Mac and remove the USB flash drive.

## Prepare for Arch Linux install

#### Resize the OS X partition

1. Start up your Mac and run the Disk Utility application.
2. Select your Mac hard drive (not a partition) and click the Partition tab.
3. Note the size of your main partition, and decide how much you want to leave for OS X versus how much to give to Linux. If this will be a minimal OS X install used only for maintenance, 20-30 GB is fine.
4. Change the "Size:" number to the desired size of your OS X partition, and hit Tab. 
5. Click the Apply button to resize the drive.

#### Create the Arch Linux installation USB flash drive

1. Download the latest Arch Linux `.iso` file from [the RackSpace mirror](http://mirror.rackspace.com/archlinux/iso/latest/).
2. Insert your USB flash drive, and eject it if it is mounted.
3. Open Terminal, and run 'diskutil list'. Figure out which is your USB device, for example, `/dev/disk2`. 
4. Make sure all partitions of your USB disk are unmounted. Use `diskutil unmountDisk /dev/disk2`, for example.
5. **Warnings** The next step will erase everything on your flash drive. Also be **completely sure** that you have the right device name, or you could destroy your OS X installation.
6. Write the .iso image to the USB flash drive: `dd if=~/Downloads/<name-of-image>.iso of=/dev/rdisk2 bs=1m` for example. Be sure to use `rdiskN` instead of `diskN`.
7. Eject the drive: `diskutil eject /dev/disk` for example.
8. Shut down your Mac and remove the flash drive.

## Set up Arch Linux

#### Prepare for Arch Linux installation
 
1. Insert the Arch Linux flash drive created in the previous section.
1. Hold down the Option key, and press and release Power. Continue to hold down Option until you see icons.
2. Using the arrow keys, navigate to "EFI Boot" or similar and hit Enter to boot the flash drive.
3. You should now be booted into Arch Linux, running as root, working entirely from RAM. 
4. Run `cgdisk` and delete the partition previously created under OS X. 
5. **note** some recommend adding an extra dummy 128MB partition in front of the main one created in the next step.
5. Now add a partition for Linux, using that free space, and mark it for Linux file system. Let's say it's `/dev/sda4` for example.
6. Format the partition using `mkfs.ext4 /dev/sda4` for example.
7. Mount the partition using `mount /dev/sda4 /mnt` for example.

#### Install Arch Linux

8. Go through normal install steps but do not install a bootloader. We will use the EFI bootloader that is built into the Linux kernel.
9. Umount the Arch Linux partition, for example `umount /dev/sda4`.
10. Shut down the system using `systemctl poweroff` and remove the USB flash drive.

## Set up dual boot

1. Power on the Mac and login to OS X. 
2. Download and unzip the rEFInd binary zip package from [Rod Smith's project](http://www.rodsbooks.com/refind/getting.html). Make sure the unzipped directory is in your Downloads directory.
3. Shutdown (power off) the Mac.
4. Hold down the Command and R keys and press and release Power. This should boot your Mac into Recovery mode.
5. If you enabled FileVault2 encryption of your OS X hard drive, unlock the drive by running Disk Utility and choosing File/Unlock Disk from the menus, then quit Directory Utility.
6. Run Terminal from the Utilities menu. 
7. Run `df -h` and find where the main hard drive is mounted (it will be something like `/Volumes/Macintosh HD`.
8. Change to the rEFInd directory, something like `cd /Volumes/Macintosh HD/Users/<your-account>/Downloads/refind-bin-0.10.2`.
9. Run `./refind-install`. You should see messages indicating success.
10. Reboot your Mac. If all is well, you should see a rEFInd dual boot screen with icons for OS X and Arch Linux. Test this by booting both in turn.

## References/Source Material

Credit goes to these sites for providing source material for this blog post.
You can reference these for further information.

* [Mashable: how to clean install El Capitan](http://mashable.com/2015/10/01/clean-install-os-x-el-capitan)
* [Arch Linux: Macbook Installation](https://wiki.archlinux.org/index.php/MacBook#OS_X_with_Arch_Linux)
* [Arch Linux: USB flash installation media in Mac OS X](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Mac_OS_X)
* [The rEFInd Boot Manager](http://www.rodsbooks.com/refind)

A special thanks to Mashable, and the Rod Smith site, for excellent explanations and instructions.

If you really want to dig into how things work during boot, an excellent read is Rod Smith's [Managing EFI Boot Loaders for Linux](http://www.rodsbooks.com/efi-bootloaders/index.html).