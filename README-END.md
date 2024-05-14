# Ordissimo installation

## Disclaimer

The Debianissimo project was made to let everyone use the Ordissimo software, which *should* respect the GNU/Linux License.

Debianissimo uses and is built on public Debian-based repositories, therefore there are no legal issues.

Debianissimo is not associated in any way to Ordissimo SA. All rights of Ordissimo SA and respective logos belong to the rightful owner.

The Debianissimo team **does not take any responsibility for any damage** caused by the installation or the use of the Ordissimo OS.

---

## Requirements
- [Debian 9 "stretch"](https://www.debian.org/releases/stretch/debian-installer/) The base to install Ordissimo, you should download the "netinst" version
    - [Direct link to the `netinst` .iso file](https://cdimage.debian.org/cdimage/archive/9.13.0/amd64/iso-cd/debian-9.13.0-amd64-netinst.iso)
- Internet
- A 64bits CPU (`AMD64` o `x86_64`), 32bit (`i386`, `x86`) and ARM CPUs aren't supported.
- 2GB of RAM
- Have base knowledge of how Debian works since you will have to manually edit some files, so it may bring issues if not done correctly

## Installation

We highly discourage the use of [Physical Hardware](#--physical-hardware), so we suggest to use a virtual machine via VirtualBox. For more infos, check out the [FAQ](#--vm-software)

1. Install Debian 9
    1. Use a disk with minimum [64 GB](#--disk) of space
        - Create a partition of at least 15 GB as primary with `ext4` format, label `root`, mount point `/` and bootable
        - Create a 2 GB partition as primary with type `linux swap`
            - The swap partition is optional and it can be as big as you want as long as you still have space for the other partitions.
        - Create a partition of at least 30 GB with `ext4` format, label `var` and mount point `/var`
        - Create a partiotion of at least 1 GB with `ext4` format, label `secours` and (custom) mount point `/mnt/secours`
        - Create a partition with the remaining space with `ext4` format, label `home` and mount point `/home`
    1. Use `ordissimo` as hostname
    1. Use `ordissimo` as domain name
    1. Use as password for the "root" user the password `root` or a password that you will remember
    1. Use `ordissimo` as username
    1. Use `ordissimo` as password
    1. During the setup, the installer will ask to set up the package manager, however it should also ask you if you want to configure another cd, press no, after that it will ask you to set your nation for the mirrors, at this point press no (and press continue if an error appears) and press yes when the installer asks you to continue __without__ a network mirror
        - These errors will be fixed later
    1. If it asks you to install a "Desktop Environment", DO NOT install it since it will cause issues with the Ordissimo installation (if it doesn't ask it's fine)
1. After the system's boot you will get into the `tty`, login with the user `root` by using the password you set before
    1. Using a text editor (nano and vi are already installed), edit the `/etc/apt/souces.list` file:
        - Add an `#` at the start of the lines that start with `deb cdrom:[` if not present already
        - Add a line with `deb http://archive.debian.org/debian stretch contrib main non-free`
    1. Run `apt update`
1. Download the `install.sh` script from the repo
    - You can download it from the terminal, if these tools aren't installed you can easily install them with `apt`:
        - curl: `curl https://raw.githubusercontent.com/Debianissimo/install-script/main/install.sh -o install.sh`
        - wget `wget https://raw.githubusercontent.com/Debianissimo/install-script/main/install.sh`
1. Run `chmod +x install.sh` to make the script runnable
1. Run the `install.sh` script with the root user, follow the instructions and accept the disclaimer (since the installer is not translated, get a translator to read it)
    - 2 GBs of download are requested to install
    - If it blocks while connecting to `substantielwww.dyndns.org` and after that it gives you an error try the following steps:
        1. Turn off the VM
        2. Restart your computer
        3. Start the VM again, login as root and run the script again with the `--skip-mirror` parameter, so the command will become `./install.sh --skip-mirror`
        - NOTE: we don't really know why this is needed, but they seem to work for now, if you're running into issues you can contact us on [Telegram](https://t.me/Debianissinauta) and we will try to help you
    - When it asks you to change the keymap, don't change it
    - If it asks you to replace configuration files, always reply with "yes"
    - When it asks questions about `rEFInd` reply with no
    - At the `lilo` questions reply with `Ok` or `Si` depending on the options
1. Add the `ordissimo` user to the `sudo` group and set the following rule, replacing the default `%sudo` rule with `%sudo   ALL=(ALL) NOPASSWD:ALL` in the file given by the `visudo` command
    - To use nano/vim/micro with `visudo`, use `EDITOR=<editor> visudo`
    - To add the ordissimo user to the sudo group, use `usermod -aG sudo ordissimo`
1. Installing a terminal could come in handy to, for example, set the screen resolution, check [FAQ](--changing-resolution) for more details
1. Restart, wait for the language selection screen, select a language, wait for the reboot and login with the user (which will be the ordissimo one in the Ordissimo UI)
    - The keyboard will automaticalle be in AZERTY, as far as we know, you **cannot** change it, therefore the password will be `ordissi;o` (the `;` button in the US layout is the `m` in AZERTY, generally it's the button on the right of the L)
    - You will need to repeat the login two times

---

## FAQ

### - **Read-Write Filesystem**

Ordissimo automatically sets the system in readonly (RO) for some unknown reason, but there are two ways to set it to Read-Write (RW)

To check if the system is in RO or RW, you can try to write a file in a directory like /usr (with sudo, or else you'll get a permission denied message) or by running `cat /proc/cmdline` in your terminal, which will show the parameters given to the kernel and also a `ro` or `rw` depending on the system's writeability state

#### - First method, not permanent

By using the following command in bash you can make the filesystem RW, this is not permanent and you will have to run it again after a reboot

1. Run the command `for m in $(ls /); do mount -o remount,rw /$m; done`
    - This command runs `ls /` and for every file / folder it tries to remount the folder with r/w perms, therefore when it will try to mount a file it will show an error though it can safely be ignored

- From our tests, this method works with no problems and succeeds at putting the file system in Read-Write almost every time

#### - Second method, permanent

By editing the lilo configuration file you can make it so that lilo will not set the filesystem to RO at boot, so the change to the config will be preserved even after a reboot

1. Make the following edits to`/etc/lilo.conf`
    - There are 3 `Read-Only` string in the file, for each of those change them to `Read-Write`
    - Save the file
1. Run `sudo lilo`
1. Restart
1. Check if the file system is RW with the method mentioned before

- This method doesn't seem to put the entire system in RW, so sometimes you will have to also run the non permanent method

### - **Installing the terminal**

> **Warning**:
> To install the terminal, you will need to put your file system in [Read-Write](#--read-write-filesystem)

The terminal is NOT installed by default on Ordissimo, so you will need to install it manually. The method changes based on the current state of the system.

#### - Pre-reboot system after executing install.sh

> **Warning**:
> This method does NOT work if you already rebooted after running the install script, in that case follow the section [below](#--post-reboot-system-after-executing-installsh)

1. Run `apt install xfce4-terminal`
1. Edit `/usr/share/applications/xfce4-terminal.desktop` to make the following change
    - Find the line that contains `Categories=GTK;System;TerminalEmulator;` and add `OrdissimoDefault;` at the end of it
    - Save the file
1. After restarting and doing the initial Ordissimo setup, you should find the terminal in the apps
    - To set a layout **on the terminal** use `setxkbmap <layout>` (for ex. the italian layout is `it`)
        - Optionally, you can put it in the `.bashrc`
            - You may need to set your file system to [Read-Write](#--read-write-filesystem)

#### - Post-reboot system after executing install.sh

> **Note**:
> This method also works before rebooting, but it's way longer

1. Run the Debian 9 installation iso
1. Go into the advanced menu
1. Go to `Rescue mode`
1. When it asks you to select a partition select `/dev/sda1`
    - If some warnings are shown, click `Yes`
1. Press `Execute a shell in /dev/sda1`
    - If some warnings are shown, press `Continue`
1. Run the following commands to install it:

```sh
mount /dev/sda5 /var
mount /dev/sda7 /home
apt install xfce4-terminal
```

1. After installing this you will have to edit `/usr/share/applications/xfce4-terminal.desktop` to make the following change
    - Find the line that contains `Categories=GTK;System;TerminalEmulator;` and add `OrdissimoDefault;` at the end of it
    - Save the file
1. After restarting you should find the terminal on your desktop/in the apps
    - To set a layout **on the terminal** use `setxkbmap <layout>` (for ex. the italian layout is `it`)
        - Optionally, you can put it in the `.bashrc`
            - You may need to set your file system to [Read-Write](#--read-write-filesystem)

### - **Changing resolution**

> **Warning**:
> To change screen size you will have to install the [terminal](#--installing-the-terminal) and you will need to have the file system in [Read-Write](#--filesystem-read-write)

To change the screen size (assuming you're in a VM) run `sudo nano /etc/X11/xorg.conf` and write:

<!-- markdownlint-disable-next-line -->
```
Section "Monitor"
 Identifier "Virtual1"
 Option "PreferredMode" "1360x768"
EndSection
```

This will change your screen size to `1360x760`. Obviously you can change it to your liking by editing the `1360x768` in the third line

### - **Error during the installation**

If you followed all the steps and you still get an error, you can come on [Telegram](https://t.me/Debianissinauta) so we can try to help you fix your issue(s)

Otherwise, if you didn't follow the entire guide and you didn't format the disk like described you will obviously get errors, Ordissimo is a very delicated system with little to no space for errors.

We will help you only if you followed the guide from start to end without skipping any steps.
 
### - **Updates**

In the Ordissimo settings there's a button to update the system that is well known to break the system or not do anything at all, so we highly recommend to not press it to not play russian roulette with your system.
If you still want to press it you will take all responsibilities and we will not help you fix your system.
 
### - **Dual booting with windows**

We never tried to dual boot with Ordissimo but keep in mind that ordissimo is a BIOS system, not UEFI like on all modern Windows computers, so you may have problems booting Windows, also we suggest only trying Ordissimo in a virtual machina as it's known to give issues on real hardware

We will **not** help you dual boot

### - **Raspepberry PI**

Raspberry PIs have ARM processors; like we said before, ARM CPUs are **NOT** supported due to critical packages missing, so the installation will not work.

As always, if you want to try this, we will **NOT** help you.

### - **Disk**

The Debianissimo installation **REQUIRES** 64GBs as your disk's size, but if you use a non preallocated disk, the final disk size will be ~10.2GB

Partitioning a smaller disk may cause issues since the disk may go out-of-space on certain partitions while installing the Ordissimo packages

If you want to try on smaller disks, we will **NOT** help you

### - **Physical Hardware**

Even if this is **__technically__** possible we don't recommend doing it due to many issues, like changing the BIOS setting or having installation issues of Debian and/or Ordissimo

We do **NOT** recommend doing this

### - **VM software**

We recommend the use of VirtualBox since it's the software that we used from the start to test Ordissimo

VmWare also works, but you will have to do more stuff to make it work:

- The disk used to install **HAS TO** be `SATA`
    - If you don't know how to set it to `SATA` you need to use Vmware Workstation Pro with the advanced setup during the VM's creation, you can also do it on Vmware Workstation Player but you will have to remove the disk and re-add it as `SATA`, it should be available in the options
- Make sure you have A LOT of patience because `lilo` doesn't seem to like VmWare, so the boot may take over **__20 minutes__** 

### - **Guest additions**

The VirualBox guest additions or the VmWare tools are NOT necessary for the VM to work and they're not even recommended since they may have/cause issues or might not work at all.

Another reason of why you might want to install the guest additions/VmWare tools if for the screen size, this is better fixed by manually changing the size with this [FAQ entry](#--changing-resolution)
