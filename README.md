# nobara_tweaks
***THIS REPOSITORY IS A WIP AND IS CURRENTLY UNFINISHED, DO NOT FOLLOW THESE GUIDES/SCRIPTS IF YOU SEE THIS WARNING, WAIT UNTIL THE SCRIPTS ARE FINISHED*** Thank you for your patience until I've finished writing these scripts and guides.

My own tweaks and scripts to/for nobara to use sway in place of KDE, unlock a luks encrypted root with a usb key, add better multi-monitor support, add secure boot (where supported), and add various keyboard shortcuts for easy switching between desktop/gaming mode. ***Please note these scripts and guides come with no warranty, support, or guarantees and is in no way associated with the nobara linux project or glorious eggroll's other work.***

## TL;DR
If you don't want the detailed explanation 

1. Git clone this repository onto a fresh nobara Steam-HTPC install in desktop mode.
> git clone https://github.com/Quietek/nobara_tweaks.git
2. Open konsole, then cd into the cloned directory.
> cd nobara_tweaks
3. Run the scripts for the tweaks you want.
> ./nobara_sway_fix
4. Follow/answer the prompts.  


## Detailed Breakdown
Everything from this point on is a detailed breakdown of the stepstaken by the included setup scripts that are included in this repository. If you want to do this manually, or you're attempting something similar on another distribution, you may find this helpful, but ***It should not be necessary to take these steps if you are using the included setup scripts on nobara.***

### Prerequisites
We need to take some steps prior to even installing nobara. You should:
1. Disable Secure Boot (This can be re-enabled later, though is UEFI/BIOS dependant).
    1. Most motherboards have this under a "Security" setting after booting into their BIOS (Usually accessed by pressing the "Delete" key or "F2" key during boot)
2. If dual booting, install windows ***before*** proceeding to the next steps.
    1. I recommend installing windows and nobara on ***different physical ssds/drives***. 
    2. When creating your windows installation media, make sure to use a ***GPT partition table*** or the option that is ***explicitly for booting in UEFI mode***
    3. Additionally, I would encourage you to ***disconnect every pysical ssd/drive that is not explicitly for Windows during the Windows installation process***. 
3. Disconnect all but one monitor prior to install.

Steps 1 and 2 are necessary because (at least in the past) microsoft has written their EFI boot partitions to other disks it detects with bootable partitions, overwriting boot partitions for other operating systems. This is also why if you dual-boot on a single disk, the EFI partition will likely be shared and you run the risk of windows overwriting your nobara bootloader during an update. Step 3 is necessary in case Steam's gaming mode gets confused with which connected monitor to prioritize.

### Installation

1. Download the Steam-HTPC nobara iso from [here](https://nobaraproject.org/download.html#standard) then follow the [nobara new user guide](https://wiki.nobaraproject.org/en/new-user-guide-general-guidelines) to create your installation media. 
2. Continue through the nobara setup process as normal until you reach the "Partition" section.
3. During this step, select "Erase disk" and "Swap (with Hibernate)" and "btrfs". ***Double check and make sure you select the correct disk you wish to use for nobara in the dropdown, all data on the selected disk will be deleted***
4. I strongly recommend enabling LUKS full disk encryption. Set a strong password that is either ***very*** memorable to you or you write down somewhere safe that you won't lose track of.
5. Continue through the installation as normal, until prompted to reboot/restart. Then reboot/restart and remove the installation USB from your computer.

### First Boot

1. You will be prompted with the steam deck start-up splash screen. This is normal. 
    1. If you enabled luks, nothing else will seem to happen. Type your luks password, then press enter. As a note, you can optionally press the "Escape" key during this screen and it will show you an actual prompt for your password and the console output during the bootup process. 
2. You will eventually be greeted by a SteamOS setup screen. Login to your steam account as normal, then press the "Escape" key when you get to the steam big picture mode screen. 
3. Click "Power Options", then "Switch to Desktop". You'll now be in your KDE desktop environment.
4. Click the icon in the bottom left of your monitor or press the "Windows" key on your keyboard.
5. Search for "Konsole" and press enter.
6. In the newly opened window, type "passwd", then set a new password for the default user account.
4. A "Nobara Welcome App" should automatically open on your desktop. Click "First Steps".
5. Click "Launch" next to "Update My System". Install any available updates.
6. Click "Launch" next to "Install media codec packages".
7. Click "Launch" next to "Open Driver Manager".
    1. Click the "Gear" icon in the top right of the newly opened window. 
    2. (Nvidia only) Click on "NVIDIA Graphics Drivers for Linux (Production) and install. 
    3. (Optional, Nvidia GPU only) Click on "Nvidia additional CUDA packages" and install. 
    3. (Optional, AMD GPU only) Click on "AMD ROCm Compute Driver Stack" and install.
    4. (Optional, Xbox accessories only) Click on "Linux kernel driver for Xbox One and Xbox Series X|S accessories" and install. This driver has been kinda finnicky for me, I found it caused me more problems than it solved when using my controller over bluetooth.
8. (Optional) install any desired additional software in the "Optional Steps" and "Recommended Additions" sections of the Nobara welcome app.
8. Close out of the "Nobara Welcome App"

### Automatic LUKS USB Unlock

I recommend doing this first after you've installed your system and necessary drivers, as this step is most likely to cause an unbootable system if you do mess something up. ***You will be making changes to system configuration files with elevated permissions, proceed with caution.***

1. If you haven't yet, insert the USB drive you wish to use to unlock your system into any open usb port.
2. Press the icon in the bottom left of your screen or the "Windows" key on your keyboard, search for "Konsole" and then open the "Konsole" application.
3. In the newly opened window type or paste the following then press enter:
    1. This command elevates you to the "root" user, with highly elevated permissions. ***Be very careful when following the next steps.***
> sudo su - 
4. Enter your password. Then press enter. 
5. Type or paste the following then press enter:
    1. This command simply lists all media disks attached to your computer.
> fdisk -l
7. You should now see a list of disks that are attached to your computer, scroll through the results until you find a "Disk /dev/sdX" entry (where X is any lower case letter) that has details that match the USB drive you inserted (e.g. size, vendor, disk model etc.)
9. Type or paste the following, replacing /dev/sdX with the entry you found in step 7, then press enter:
    1. ***All data on the USB drive is about to be erased. Make sure anything important on the usb drive is backed up prior to this!***
> format the usb in fat32 here
10. Type or paste the following to generate and add a keyfile for your luks partition, typing your current luks password when prompted.
    1. Replace /dev/nvmeXnpX with your root luks partition. In my case this was /dev/nvme0n1p3.
> dd if=/dev/urandom of=/root/luks-unlock.key bs=1 count=4096
> cryptsetup luksAddKey /dev/nvmeXnpX luks-unlock.key --key-slot 1
11. Mount your newly formatted USB disk.
    1. Replace /dev/sdXX with the fat32 partition of your usb. In my case it was /dev/sdc1
> mkdir -p /mnt/luks-usb/
> mount /dev/sdXX /mnt/luks-usb/
> mv luks-unlock.key /mnt/luks-usb/key
12. Add the new keyfile to your crypttab.
> add keyfile to crypttab here
13. rebuild your boot image.
> dracut -f
