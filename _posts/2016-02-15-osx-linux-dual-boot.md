---
layout: post
title: "Dual booting macOS and Linux"
date: 2016-02-15
---

I've successfully set up my MacBook Pro Retina (mid-2012) with a clean install of macOS Sierra, dual booting to Arch Linux.
Other blog posts, and the Arch Linux Wiki itself, are misleading, confusing, or leave out important parts.
Here's the straight information. This recipe should work (with some obvious adjustments) for other Mac computers or Linux distributions.

**Why would you want to dual boot?**

If your work on this machine is mostly Mac software with a bit of Linux from time to time, it is best *not* to use dual boot.
Instead, set up Linux as a virtual machine under macOS. You can use the [free/open-source VirtualBox](http://virtualbox.org),
or the commercial [VMware Fusion](https://www.vmware.com/products/fusion) or [Parallels Desktop](http://www.parallels.com/products/desktop/).

If your work on this machine will be mainly Linux, it makes sense to set it up as a dual boot machine. 
When booted natively in Linux, you will get the best performance and simplest operation.

Why not make your machine completely Linux? Because you need macOS for support/maintenance and updates.

**You need the following:**

1. A Mac computer new enough to run macOS Sierra. It's best if this is an "extra" machine that you want to use mainly for Linux.
2. A decent Internet connection, as you will be downloading multiple gigabytes of code.
3. A USB flash drives that you can erase. It need to be at least 8GB, preferably 16GB. A USB3 flash drive will make this go much faster.

**We do this in several steps:**

1. Back up any data that you need
2. Clean install macOS Sierra
3. Prepare for Arch Linux install
4. **or** Prepare for Arch Linux encrypted install
5. Install Arch Linux
6. Set up dual boot

## Back up data

**All data on your Mac will be erased!** So back up whatever you need. Use Time Machine, copy the files, make an image copy, whatever works for you.

You can skip the "clean install macOS Sierra" step, and keep your data, but it's recommended to start fresh.

## Clean install macOS Sierra

This step is optional, but recommended, to start the life of your dual boot system with a clean macOS install.

### Create macOS Sierra installation USB flash drive

1. Open the App Store on your Mac, go to the Purchased tab, and download macOS Sierra. **warning** this is a 6GB download!
2. If you see it, quit the application asking you to install.
3. Insert your USB flash drive. **warning** All data on this drive will be erased.
4. Open the Disk Utility application.
5. Navigate to your USB drive (not a partition) and click the Erase tab.
6. Make sure the format is "Mac OS Extended (Journaled)" and the name is "Untitled". The partition scheme should be "Master Boot Record". Click Erase.
7. Open a Terminal window and run the following command (may take several minutes).

```bash
sudo /Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ macOS\ Sierra.app
```

When it completes, use Finder to eject the flash drive, to ensure all data has been written.

### Boot from USB flash drive and clean install macOS Sierra

1. On the Mac that you want to clean install, shutdown (power off), then insert the macOS Sierra install USB flash drive that you created in the previous section.
2. Hold down the Option key, then press and release Power. Continue to hold the Option key until you see some icons.
3. You should see an icon that says something similar to "Install macos Sierra". Use your arrow keys to navigate to the latter, then Enter.
4. You should see "macOS Utilities" window. Run Disk Utility.
5. Select your main Mac hard drive (not a partition) and click Erase. **warning** This will erase all of your data!
6. Close Disk Utility, and run "Install macOS" from the macOS Utilities menu.
7. Continue through the macOS install process, selecting options as you wish. Then shut down the Mac and remove the USB flash drive.

## Prepare for Arch Linux install

### Resize the macOS partition

1. Start up your Mac and run the Disk Utility application.
2. Select your Mac hard drive (not a partition) and click the Partition tab.
3. Note the size of your main partition, and decide how much you want to leave for macOS versus how much to give to Linux. If this will be a minimal macOS install used only for maintenance, 30 GB is fine.
4. Change the "Size:" number to the desired size of your macOS partition, and hit Tab. 
5. Click the Apply button to resize the drive.

### Create the Arch Linux installation USB flash drive

1. Download the latest Arch Linux `.iso` file from [the RackSpace mirror](http://mirror.rackspace.com/archlinux/iso/latest/).
2. Insert your USB flash drive, and eject it if it is mounted.
3. Open Terminal, and run 'diskutil list'. Figure out which is your USB device, for example, `/dev/disk2`. 
4. Make sure all partitions of your USB disk are unmounted. Use `diskutil unmountDisk /dev/disk2`, for example.
5. **Warnings** The next step will erase everything on your flash drive. Also be **completely sure** that you have the right device name, or you could destroy your macOS installation.
6. Write the .iso image to the USB flash drive: `dd if=~/Downloads/<name-of-image>.iso of=/dev/rdisk2 bs=1m` for example. Be sure to use `rdiskN` instead of `diskN`.
7. Eject the drive: `diskutil eject /dev/disk2` for example.
8. Shut down your Mac and remove the flash drive.

## Prepare for Arch Linux installation

1. Ensure you have a hard-wired Ethernet connection, preferably using the Thunderbolt Gigabit Ethernet adapter.
2. Insert the Arch Linux flash drive created in the previous section.
3. Hold down the Option key, and press and release Power. Continue to hold down Option until you see icons.
4. Using the arrow keys, navigate to "EFI Boot" or similar and hit Enter to boot the flash drive.
5. You should now be booted into Arch Linux, running as root, working entirely from RAM.
6. Run `lsblk` and `lsblk -f` and review the existing disks and partitions.
7. For this example, we are assuming the disk device you are working with is `/dev/sda`, adjust these steps as needed.
8. Run `cgdisk /dev/sda` and delete the partition you created under macOS for Arch Linux.
9. Still under `cgdisk`, add a partition for Linux, using the rest of the free space, label it ARCH, and mark it for Linux file system. Let's say it's `/dev/sda4` in this example.
10. Write the partition table and exit `cgdisk`.

If you do **not* wish to encrypt your Arch Linux root filesystem, just do the following two steps:

1. Format the partition using `mkfs.xfs /dev/sda4`.
2. Mount the partition using `mount /dev/sda4 /mnt`.

If you *do* wish to encrypt your Arch Linux root filesystem, perform the steps in the following section.

### Set Up for Encrypted Arch Linux

Use this section to encrypt the Arch Linux partition. Skip this section if you want your Arch Linux installation to be unencrypted.

Wipe the Linux partition with dummy encrypted data:

```bash
cryptsetup open --type plain /dev/sda4 container --key-file /dev/random
# The following command will end with a "No space left on device" message.
dd if=/dev/zero of=/dev/mapper/container status=progress
cryptsetup close container
```

Set up encryption for the Linux partition, which will now be accessed as `/dev/mapper/archlinux`:

```bash
cryptsetup -v -s 512 luksFormat /dev/sda4
# Please remember the passphrase you entered! You will need it to access your Arch Linux system. 
cryptsetup open --type luks /dev/sda4 archlinux
```

Back up the LUKS header:

```
# ???
```

Format and mount the Linux partition:

```bash
mkfs.xfs /dev/mapper/archlinux
mount /dev/mapper/archlinux /mnt
```

## Install Arch Linux

This installs Arch Linux to the partition we created in the previous step. No separate bootloader is installed because we are installing the `systemd-boot` boot manager, the kernel, and the `initramfs` into the EFI System Partition, which allows us to boot the kernel from there.

1. Mount the ESP (EFI System Partition) as the boot partition by running `mkdir /mnt/boot` and `mount /dev/sda1 /mnt/boot`.
2. Sync time: run `timedatectl set-ntp true`
3. Set up the mirror list by editing `/etc/pacman.d/mirrorlist` and commenting out all but the RackSpace entry.
4. Bootstrap the install: run `pacstrap /mnt base base-devel vim`, which will take several minutes.
5. Generate the fstab: run `genfstab -p /mnt >> /mnt/etc/fstab`
6. Switch to the installed system: run `arch-chroot /mnt`
7. Set the hostname: run e.g. `echo starship-enterprise > /etc/hostname`
8. **optional** Set the local timezone to e.g. Central: `ln -s /usr/share/zoneinfo/US/Central /etc/localtime` (note: if you skip this step the system will run on UTC time)
9. Set up your locale: edit `/etc/locale.gen` and uncomment your desired locales (e.g. `en_US.UTF-8 UTF-8`)
10. Create your locale: run `locale-gen`
11. Set your locale current configuration: run e.g. `echo LANG=en_US.UTF-8 > /etc/locale.conf`
12. Set up the network: use `ip link` to discover your network interface name, then run e.g. `systemctl enable dhcpcd@enp0s3.service`
13. Set the system clock: run `hwclock --systohc --utc`
14. Set the root user password: run `passwd`
15. **IF you encrypted your Linux root filesystem**: `vim /etc/mkinitcpio.conf` and change the `HOOKS=` line to `HOOKS="systemd autodetect modconf block sd-encrypt filesystems keyboard fsck"`.
16. Run `mkinitcpio -p linux` to generate a new Linux initramfs.
17. Set up the `systemd-boot` boot manager: `bootctl --path=/boot install`
18. Create a boot entry for Arch Linux by creating a file `/boot/loader/entries/arch.conf` and changing `/boot/loader/loader.conf` to point to it. Here are examples to use:

**`/boot/loader/loader.conf`**:

```
timeout 3
default arch
editor  0
```

For a non-encrypted root filesystem, create **`/boot/loader/entries/arch.conf`** like this:

```
title       Arch Linux
linux       /vmlinuz-linux
initrd      /initramfs-linux.img
options     root=PARTUUID=<partition-uuid> rw
```

Tip: To start the `arch.conf` file, seed it with `blkid -s PARTUUID -o value /dev/sda4` > arch.conf` which creates a file with the correct Partition UUID in it.

For an encrypted root filesystem, create **`/boot/loader/entries/arch.conf`** like this:

```
title       Arch Linux
linux       /vmlinuz-linux
initrd      /initramfs-linux.img
options     luks.name=<device-uuid>=archlinux root=/dev/mapper/archlinux rw
```

Tip: To start the `arch.conf` file, seed it with `blkid -s UUID -o value /dev/sda4` > arch.conf` which creates a file with the correct Device UUID in it.


## Complete installation and boot system

Finally, run `exit` to leave the installed system environment, and `systemctl poweroff` to shut down the machine. Remove the USB flash drive and reboot the machine. You should be able to select Arch Linux or macOS, and boot either successfully.

## References/Source Material

Credit goes to these sites for providing source material for this blog post. You can reference these for further information.

* [OS X Daily: how to clean install macOS Sierra](http://osxdaily.com/2016/09/26/clean-install-macos-sierra/)
* [Arch Linux: Macbook Installation](https://wiki.archlinux.org/index.php/MacBook#OS_X_with_Arch_Linux)
* [Arch Linux: USB flash installation media in Mac OS X](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Mac_OS_X)
* [Arch Linux: using dm-crypt to encrypt devices](https://wiki.archlinux.org/index.php/Dm-crypt)

If you really want to dig into how things work during boot, an excellent read is Rod Smith's [Managing EFI Boot Loaders for Linux](http://www.rodsbooks.com/efi-bootloaders/index.html).
