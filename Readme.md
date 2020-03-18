# Over[Arch]ing issue #1
Disconnecting during an update seemed to cause an issue similar to the one mentioned [here](https://www.reddit.com/r/linuxquestions/comments/e2lr5k/arch_wont_boot_after_upgrade_vmlinuzlinux_missing/). As it states I get the following error on boot:
```
Loading Linux linux ...
error: file '/boot/vmlinuz-linux' not found.
Loading initial ramdisk ...
error: you need to load the kernel first.
```


## Boot to Live USB
I set mine up the cheap way using [this video](https://www.youtube.com/watch?v=uWO3vif7hTw)

After that's done simply bring up your boot menu (For my Dell I spam <kbd>Delete</kbd> until a "bring up single use boot menu"  thing pops up and then spam <kbd>F12</kbd>). Boot into the partition with the Arch USB.
## Mount Your Filesystem
Specifically for my system: `mkdir add; mount /dev/sda2 hdd/; mount /dev/sda1 hdd/boot/; mount /dev/sda3 hdd/home/`
## Downgrade packages
Note this didn't actually resolve the issue for me, but it's worth mentioning. First I copied the linux preset for mkinitcpio to my mounted copy of my system, then chroot and downgrade your linux to the last working one

```
cp /etc/mkinitcpio.d/linux.preset hdd/etc/mkinitcpio.d/linux.preset
arch-chroot hdd /bin/bash
cd /var/cache/pacman/pkg/
pacman -U linux-<version>-x86_64.pkg.tar.xz
```

I am not running a virtual machine, but an Arch Linux only so the downgrade is more barebones. Check [Downgrading Paackages Wiki](https://wiki.archlinux.org/index.php/Downgrading_packages#Downgrading_the_kernel) for more info. At this point, in theory, you can `exit` and `reboot`. This got me to an "Oh no! Something has gone wrong..." white screen and I could not access the virtual terminals.


## Connect to Wifi in Arch-chroot
If you haven't yet connected to the network (which if you're booting into live USB, you prob haven't): 

`> sudo wifi-menu`

You can add the `-o` tag to hide your password. Pick the network you want and save the name **without dashes**. For example, `wlan0meowtownguest`  since my *interface* is _**wlan0**_ and my *ESSID* is _**Meow Town Guest**_. Check the status with 

```diff
> sudo netctl status wlan0meowtownguest
● netctl@wlp9s0sktab.service - Automatically generated profile by wifi-menu
  Loaded: loaded (/etc/systemd/system/netctl@wlp9s0sktab.service; static; vendor preset: disabled)
+ Active: active (exited)+ since Sun 20120-03-18 18:51:06 IST; 20s ago
  Docs: man:netctl.profile(5)
 ... # etc
```
If instead you get 
```diff
- Active: failed (exited)+ since Sun 20120-03-18 18:51:06 IST; 20s ago
```
Try deleting the file and starting again with
```
> sudo systemctl stop dhcpcd.service
> sudo systemctl disable dhcpcd.service
> sudo rm -fr /var/lib/dhcpcd/dhcpcd-wlan0*
> sudo rm /etc/systemd/system/multi-user.target.wants/netctl*
> sudo rm -fr /etc/netctl/wlan0*
```
And repeating the process. To test your connection `ping 8.8.8.8` or similar, then we need to move our config file to the place we'll chroot and go ahead and chroot...
```
cp /etc/resolv.conf hdd/etc/resolv.conf
arch-chroot hdd /bin/bash
```

## Upgrading Packages AFTER Connecting to Wifi (See Above)
After chrooting, go ahead and `ping 8.8.8.8` to check your connection, then update whatever you need with `pacman`. Specifically, I updated everything with `pacman -Syyu` which upgraded me from the white "Oh no!" screen to a blank grey screen. Still no virtual terminals avaliable. (Removing gnome with `pacman -Rcns gdm gnome` and reinstalling thus far produced the same results)
