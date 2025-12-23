---
title: "Duel booting Gentoo and Windows 11 with Secure Boot"
date: 2025-12-23
tags: 
  - posts
layout: post.njk
---

Supposed you as a Linux user wants to play some Battlefield 6. Won't run 
on linux. You know, or whatever game that requires that TPM2.0 plus secureboot. 
Clearly a VM won't cut it at this rate, and you still want Gentoo, so whats left 
for you is duel booting (or WSL, we don't talk about WSL). Then you ask yourself, 
how do I do that? 

In this guide, we will be installing Gentoo with the following layout:
- `gentoo-kernel-bin`, because nobody has time to wait for the compilation 
- dracut
- grub, it's old and bloated, but requires the least hassle on my end
- one efi partition, already created by windows
- you use UEFI (duh)
- recommended: you have already installed Gentoo before and knows the gist of installation, 
knows how to install package with `emerge` and knows the basics of tweaking `make.conf`

This guide is quick and dirty, if you want explinations on what these commands means,
look it up with a search engine.


## Install Windows 

Skip this step if you have already installed Windows.

Microsoft doesn't envision you installing another operating system. If you install
gentoo first they would just mess up the bootloader or do some equally bad things. 
Download Windows media creation tool from [here](https://www.microsoft.com/en-us/software-download/windows11).
Plug your USB into your box and run it. Follow the instructions in the app to make a 
bootable media. Then open UEFI settings, disable secure boot and then boot the Windows 
installation medium. Afterwards, just install windows as you normally do. 

Don't try to use something like Etcher to flash the iso downloaded from the Microsoft
site into your USB. Won't work. Don't ask me why.


## Shrink Windows partition 

Press the Windows key and search up `Create and format hard disk partitions`. Open it.
In the bottom of Disk Management, you should see 3 partitions for the disk you installed 
windows on. `EFI System Partition`, `Windows (C:)` and `Recovery Partition`. Right click 
`Windows (C:)` and `Shrink Volume...` Shrink it to make an empty partition of least 3GB.

## Downloading and booting the Arch ISO 

Yes, we are using the Arch Linux ISO to install Gentoo. It is much more user friendly than 
the Gentoo minimal installation ISO. Download it [here](https://archlinux.org/download/).  
Flash it into your USB (this time, I recommend [Etcher](https://etcher.balena.io/)). Open 
UEFI and boot it.

Alternatively, do the same but with [Gentoo ISO](https://www.gentoo.org/downloads/). 

Another alternative is to just get whatever GUI Linux ISO out there.

In this guide, I will be using the Arch ISO.

Since you are on a non-gui ISO, you can't copy and paste commands. Set root password by running 
`passwd`, then activate `sshd` if not already. `ssh` in from another device. You can now copy 
and paste commands. In fact, you can use an Android phone with Termux to do the ssh connection.

## Connecting to WiFI

Goes without saying, skip if you use ethernet.

1. Run `iwctl`
1. Find your wireless adapter name by running `device list`, in my case it is `wlan0`
1. Run `station wlan0 scan`
1. Run `station wlan0 get-networks`, look for your WiFi
1. Run `station wlan0 connect "Your_Network_SSID"`, enter your password and press enter
1. exit by pressing `control+d` on your keyboard
1. `ping gnu.org` to see if its working, press `control+c` to stop pinging
1. Profit

With the Gentoo minimal installation ISO you will need `wpa_supplicant`.
Guide [here](https://www.youtube.com/watch?v=QGyHDIYlLFA). I have to confess 
the main reason I like to use Arch ISO is because I hate `wpa_supplicant`. 

## Prepare the disk and mount

1. Run `lsblk`. Find your main disk. In my case it is `nvme0n1`. 
1. `cfdisk /dev/nvme0n1`. 
1. Create a singular partition to replace the empty hole you made after shrinking `C drive`.
1. Write and quit. Don't make any more partitions. (Swap is an exception if you need it)
1. Run `lsblk` again and note down the name of said partition. In my device, `/dev/nvme0n1p5`.
It will be the root partition.
1. Also note down the Windows created EFI partition name. In my case, `/dev/nvme0n1p1`
1. Format the root partition. I will be using xfs in this case. `mkfs.xfs /dev/nvme0n1p5`
1. Create the mount directory. `mkdir -p /mnt/gentoo`
1. Mount the root. `mount /dev/nvme0n1p5 /mnt/gentoo` 
1. Create the EFI directory. `mkdir /mnt/gentoo/efi`
1. Mount the EFI partition. `mount /dev/nvme0n1p1 /mnt/gentoo/efi`

## Installing Gentoo

From this point on you will want to read the [Gentoo handbook installing stage file section](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage).

Now get the stage file by `cd /mnt/gentoo` then `curl -O link-to-stage3-tarball`. Find the 
link by copying and pasting the link to the stage3 tarball you desire in [here](https://www.gentoo.org/downloads/).
Optionally verify the tarball as per instruction on the handbook.

Extract the tarball: `tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo`
Note that you must be in the same directory as the tarball to make this work.

Now follow the `Configuring compile options` on the Gentoo handbook and onwards. Stop when you
reaches the `Configuring the Linux kernel` stage. 

## Configuring secure boot

Now for the secure boot part. 

In the `/etc/portage/make.conf` file you will make the following changes:
1. Add `dist-kernel networkmanager secureboot` to the USE flags
1. Add these lines:
```bash
SECUREBOOT_SIGN_KEY="/root/secureboot/MOK.key"
SECUREBOOT_SIGN_CERT="/root/secureboot/MOK.pem"

# if you use the modules-sign use flag, include the following
# gentoo-kernel-bin is presigned so you can skip them
MODULES_SIGN_KEY="/root/secureboot/MOK.key"
MODULES_SIGN_CERT="/root/secureboot/MOK.pem"

GRUB_PLATFORMS="efi-64"
```

Make the file `/etc/portage/package.use/installkernel` then add the line 
```bash
sys-kernel/installkernel dracut grub
```

Make another file `/etc/portage/package.use/grub` then add the line 
```bash
sys-boot/grub mount
```

Then edit `/etc/dracut.conf.d/secureboot.conf` (make the directory and the file if they doesn't 
don't exist), insert these lines:
```bash
uefi_secureboot_cert="/root/secureboot/MOK.pem"
uefi_secureboot_key="/root/secureboot/MOK.key"
```

Following that, install `sys-kernel/linux-firmware` and the microcode of your choice.

Install the secure boot utilities of `sys-boot/shim sys-boot/mokutil sys-boot/efibootmgr`.

Make/edit `/etc/env.d/99grub`, add the line:
```bash
GRUB_CFG=/efi/EFI/Gentoo/grub.cfg
```
Then run `env-update`

We are now going to make the secure boot keys thing. Run the following commands:
```bash
mkdir -p /root/secureboot 

# replace alice with your name
openssl req -new -nodes -utf8 -sha256 -x509 -outform PEM -out /root/secureboot/MOK.pem -keyout /root/secureboot/MOK.key -subj "/CN=alice/"
openssl x509 -in /root/secureboot/MOK.pem -outform DER -out /root/secureboot/MOK.der

# install grub and installkernel
# in case you somehow emerged grub before changing the useflags, pass `--update --newuse` flags to `emerge`.
emerge -a sys-kernel/installkernel sys-boot/grub

# install grub 
grub-install --efi-directory=/efi
```

Now import the keys:
```bash
# you will need to set the password 
# both command will ask you to set the password, set both password to the same password
# to avoid complications

mokutil --ignore-keyring --import /root/secureboot/MOK.der
# replace the version with your kernel version
mokutil --ignore-keyring --import /usr/src/linux-6.12.58-gentoo-dist/certs/signing_key.x509
```

Also install os-prober to detect windows: `emerge -a sys-boot/os-prober`
Enable os-prober by setting `GRUB_DISABLE_OS_PROBER=false` in `/etc/default/grub`. Change the line if exist,
append the line if it doesn't exist.

Run the following:
```bash
cp /usr/share/shim/BOOTX64.EFI /efi/EFI/Gentoo/shimx64.efi
cp /usr/share/shim/mmx64.efi /efi/EFI/Gentoo/mmx64.efi
cp /usr/lib/grub/grub-x86_64.efi.signed /efi/EFI/Gentoo/grubx64.efi

# replace /dev/nvme0n1 with your boot disk 
# replace the number 1 in --part 1 with the boot partition id, should just be 1 because 
# our efi partition is /dev/nvme0n1p1
efibootmgr --create --disk /dev/nvme0n1 -disk --part 1 --loader '\EFI\Gentoo\shimx64.efi' --label 'GRUB via Shim' --unicode

# install the kernel
emerge -a sys-kernel/gentoo-kernel-bin

# make the config
grub-mkconfig -o /efi/EFI/Gentoo/grub.cfg
```

You should be golden.
Now follow the `Configuring the system` and the `Installing tools` section 
of the handbook.

Note that for the internet package, I recommend installing networkmanager with the 
`iwd` USE flag. That way you can connect to the internet after rebooting. Avoid 
`wpa_supplicant` because I really don't like it.

## Finalizing

Run `passwd` to set root password.

Run `exit` to quit chroot.  
Run `cd` then run `umount -R /mnt/gentoo`.
Run `reboot`.
Unplug the USB.

Your box should boot to grub with shim, if not, edit UEFI settings to push it to the 
front of the boot order.

You will be met with shim UEFI key management. It should say Press any key to perform 
MOK management. Press any key. Use arrow keys to move. Select Enroll MOK, press enter. 
View all the keys. Then press continue. Select yes when asked to enroll the key(s). Enter 
the password you set while importing the keys with `mokutil` command (you can now safely 
forget that password) and reboot.

Enter UEFI settings and turn secure boot on.
Pray that you boot into the the grub menu. 
Pray that grub boots your kernel.

If nothing goes wrong, enjoy Gentoo and Windows duel boot. Now you can play Battlefield 6 
and occationally use your box to write code or do Linux things in general.

## When things go wrong 

Install `net-irc/weechat` or whatever IRC client that runs, connect to Libera 
by `irc.libera.chat:6697`, then ask for help on `#gentoo`.

You may be asked to paste logs. Install `app-text/wgetpaste` to upload your logs 
to the internet and send them the output url. Good luck.
