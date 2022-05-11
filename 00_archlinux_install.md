# __Archlinux Install for Mibook-Air__

 2022.05.08 by <u>SkoGaR</u>  
 This installation guide base on <u>Archlinux</u> wiki

---
## __Pre-installation__

### __Download the installation image__
 Download the image in operating system from <font color=powderblue>__Archlinux__</font> website.

  > __Note__: When you download the image, you should <font color=powderblue>__verifying__</font> it with:  
  > `$ gpg --keyserver-options auto-key-retrieven --verify archlinux-version-x86_64.iso.sig`

### __Make a Bootable USB Derive__
 Useing <font color=powderblue>Rufus</font> to make a bootable USB Live Image.

### __Boot the live environment__
  > __Note__: Archlinux installation image do not support Secure Boot. Make sure <font color=powderblue>disable Secure Boot</font> in UEFI. Power on the <font color=powderblue>Mibook Air</font> and press <font color=powderblue>F2</font> during the <font color=powderblue>POST</font>.

   1. Select current USB device to boot image. 
   2. When the installation medium's boot loader menu appears,select _Arch Linux install medium_ and enter the insrallation environment.
  
  | Selection | Explantion |
  | :---: | :--- |
  | Arch Linux install medium | enter the USB installation environment |
  | Arch Linux install medium (copy to RAM) | copy installation environment in RAM and enter (free your USB BUS) |

   3. You will be logged in on the first <font color=powderblue>virtual console</font> as the root user, and presented with a <font color=powderblue>__ZSH__</font> shell prompt.

### __Set the console font__
 <font color=powderblue>Console fonts</font> are located in `/usr/share/kbd/consolefonts/` and choose stype like <font color=powderblue>_LatGrkCyr-12x22.psfu.gz_</font>.
 ```
 # setfont /urs/share/kbd/consolefonts/LatGrkcyr-12x22.psfu.gz
 ```

### __Verify the boot mode__
 To verify the boot mode, list the efivars directory:
 ```
 # ls /sys/firmware/efi/efivars
 ```
 
### __Connect to internet__
 To set up a network connection in the live environment, go through the following steps:
 * set the network interface(wlan0) up.
  ```
  # ip link
  # ip link set interface_name up
  ```
 * set your wi-fi <font color=powderblue>SSID</font> and  <font color=powderblue>password</font> to `wpa_supplicant.conf`.
  ```
  # wpa_passphrase SSID passwd > wpa_supplicant.conf
  ```
 * runing `wpa_supplicant` to background.
 ```
 # wpa_supplicant -i interface_name -c wpa_supplicant.conf &
 ```
 * The connection verified with `ping`:
  ```
  # ping www.baidu.com -c 3
  ```

### __Update the system clock__
 Use `timedatectl` to ensure the system clock is accurate:
 ```
 # timedatectl set-ntp true
 ```
 To check the service status, use `timedatectl status`.

### __Partition the disks__
 When recogbized by the live system, disks are assigned to a block device such as `/dev/sda` or `/dev/sdax`. To identify these devices, use <font color=powderblue>fdisk</font>.
 ```
 # fdisk -l
 ```
 Results ending in `rom`, `loop`, or `airoot` may be ignored.
 The following partition are __required__ for a chosen device:
 
 * One partition for the <font color=powderblue>root directory</font> `/`.
 * For booting in UEFI mode: an <font color=powderblue>EFI system partition</font>.
  
In Mibook-Air, example layouts

  | Mount point | Partition | Partition type | suggested size |
  | :---: | :---: | :---: | :---: |
  | `/mnt/boot` | `/dev/efi_system_partition` | EFI system partition | 512MiB |
  | `/mnt` | `/dev/root_partition` | Linux x86-64 root (/) | Remainder of the device |

### __Format the partitions__
 Once the partitions have been created or appointed, each partition must be formatted with an appropriate file system.
 ```
 # mkfs.ext4 /dev/root_partition
 # mkfs.fat -F 32 /dev/efi_system_partition
 ```

### __Mount the file systems__
 Mount the root volume to `/mnt` with the following steps:

 * _root_partition_ to `/mnt`
  ```
  # mount /dev/root_partition /mnt
  ```
 * Create a directory fot mounting _EFI_system_partition_.
  ```
  # mkdir /mnt/boot
  ```
 * _EFI_system_partition_ to `/mnt/boot`
  ```
  # mount /dev/efi_system_partition /mnt/boot
  ```

---
## __Installation__

### __Seletc the mirrors__
 Packages to be installed must be downloaded from <font color=powderblue>__mirror servers__</font>, which are defined in  `/etc/pacman.d/mirrorlist`. Edit <font color=powderblue>__mirrorlist__</font>, comment the other servers, choosing the home server like <font color=powderblue>__tuna.tsinghua.edg.cn__</font> source. (Adding server URL depend on the server pubilsh)
 
 > Adding the following URL at the TOP in `/etc/pacman.d/mirrorlist`
 > ```
 > Server = https://mirror.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
 > ```

 Then refrash the sourve.
 ```
 # pacman -Syy
 ```

### __Easy to build the system__
 Useing bluetooth keyboard 
    
 `# pacman -S bluez bluez-utils`   
 `# systemctl start bluetooth.service`   
 `# bluetoothctl`   
 
 > 1. Enter `power on` to turn the power to controller on. It is off by default and will off again each reboot.
 > 2. Enter device  discovery mode with `scan on` command if device is not yet on the list.
 > 3. Turn the agents on with `agant on` or choose a specific agent likes `agent KeyboardOnly`.
 > 4. Enter `pair MAC_address` to do the pairing and enter the `PIN`.(tab completion works)
 > 5. Enter `trust MAC_address`.
 > 6. Enter `connect MAC_address` to establish a connection.

 Colorful pacman
 
 > * In `/etc/pacman.conf`, editing to uncomment `Color` can make the pacman easy to look.
 > * And also editing <font color=powderblue>__pacman.conf__</font> to setting the looking.

### __Install essential packages__
 Use the <font color=powderblue>__pacstrap__</font> script to install the <font color=powderblue>__base__</font> package, Linux <font color=powderblue>__kernel__</font> and firmwake for common hardware:
 ```
 # pacstrap /mnt base linux linux-firmware intel-ucode
 # pacstrap /mnt base-devel man wpa_supplicant dhcpd bluez bluez-utils acpid
 ```

---
## __Configure the system__


### __Fstab__
 Generate an <font color=powderblue>__fstab__</font> file (use `-U` to define by <font color=powderblue>__UUID__</font> seccessful, or `-L` to define by ~~__labels__~~ never seccessful yet)

 > __NOTE__: Check the resulting `/mnt/etc/fstab` file, and edit it in case of errors.

### __Chroot__
 <font color=powderblue>__Change root__</font> into the new system.
 ``` 
 # arch-chroot /mnt
 ```

### __Time zone__
 Set the <font color=powderblue>__time zone__</font>:
 ```
 # ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
 ```
 Run `hwclock` to generate `/etc/adjtime`, to administer the Hardware Clock things.
 ```
 # hwclock --systohc
 ```
 This command assumes the hardware clock is set to <font color=powderblue>__UTC__</font>.
 
### __Localizaion__
 Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed <font color=powderblue>__locales__</font>. Generate the locales by running:

 ```
 # locale-gen
 ```

 And <font color=powderblue>__create__</font> the `/etc/locale.conf` file, and <font color=powderblue>__set the LANG variable__</font> accordingly:

 > LANG=en_US.UTF-8
  

 It can <font color=powderblue>__set the console font__</font>,make the changes persistent in `/etc/vconsole.conf`

 > FONT=/usr/share/kbd/consolefonts/LatGrkCyr-12x22.psfu.gz

### __Network configuration__
 * First <font color=powderblue>__create__</font> the `/etc/hostname` file.
 
 > _myhostname_
 
 * Then with the following command to exit the chroot environment and copy the <font color=powderblue>__network configuration__</font> to new system.
  ```
  # exit
  # cp wpa_supplicant.conf /mnt/etc/wpa_supplicant/wpa_supplicant.conf
  # arch-chroot /mnt
  ```

### __Root password__
 When installed new system, <font color=powderblue>__root__</font> have nonpassword, set the root <font color=powderblue>__password__</font>.
 ```
 # passwd
 ```

### __Boot loader__
 First, install the packages `grub` and `efibootmgr`: _GRUB_ is the bootloader while _efibootmgr_ is used by the GRUB installation scirpt to write boot entries to NVRAM.   

 Then follow the below steps to install GRUB to your disk:
 1. Mount the <font color=powderblue>__EFI system partition__</font> and in the remainder fo the section, substitute _`esp`_ with its mount point, which is already mounted in `/boot` for `/dev/sda1` in this case.
 2. Choose a bootloader identifier, hera named `GRUB`. A directory of that name will be created in `esp/EFI/`to store the EFI binary and this is the name that will appear in the UEFI boot menu to identify the GRUB boot entry.
 3. Excute the following command to install the GRUB EFI application `grubx64.efi` to `esp/EFI/GRUB/` and install its modules to `/boot/grub/x86_64-efi/`.
 > __NOTE__: Make sure to install the packages and run the `grub-install` command from the system in which GRUB will be installed as the boot loader. That means if you are booting from the live installation environment, you need to be inside the chroot when running `grub-install`. If for some reason it is necessary to run `grub-install` from outside of the installed system, append the `--boot-directory=` option with the path to the mounted `/boot` directory, e.g `--boot-directory=/mnt/boot`.
 ```
 # grub-install --target=x86_64-efi --efi-directory=esp --bootloader-id=GRUB
 ```

---
## __Reboot__


---
## __Post-installation__

