ARCH-INSTALL
=================

Basic Arch Installation and Configuation Tutorial for Raspberry Pi 2/3.

## INSTALATION

Source: [https://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2](https://archlinuxarm.org/platforms/armv7/broadcom/raspberry-pi-2)

###  Partition the microSD card

* First partition to type W95 FAT32 (LBA) with 100M size, second one to type Linux (ext4) with the rest of volume.

Fdisk:
```
$ fdisk /dev/sdX
(key commands in fdisk)
o > p > n > p > 1 > enter > +100M > t > c (create the first partition)
n > p > 2 > enter > enter (create the second one)
w (finally, update changes)
```

Or with [Parted](https://wiki.archlinux.org/index.php/GNU_Parted):
```
$ parted /dev/sdX
(parted) mkpart primary fat32 1MiB 100MiB
(parted) set 1 boot on
(parted) mkpart primary ext4 100MiB -1s
```

### Setup the Filesystems

* Create and mount the FAT filesystem:
```
$ mkfs.vfat /dev/sdX1
$ mkdir boot
$ mount /dev/sdX1 boot
```

* Create and mount the ext4 filesystem:
```
$ mkfs.ext4 /dev/sdX2
$ mkdir root
$ mount /dev/sdX2 root
```

### Download and extraction root filesystem

* Type the following commands as root, not via sudo:
```
$ wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
$ bsdtar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root
$ sync
```

* Move boot files to the first partition:
```
$ mv root/boot/* boot
```

* Unmount the two partitions:
```
$ umount boot root
```

* Finally, plug the microSD and boot.

## BOOT

### Login

Use the serial console or SSH to the IP address given to the board by your router.
Login as the default user `alarm` with the password `alarm`.
The default `root` password is `root`.

* Initialize the pacman keyring and populate the Arch Linux ARM package signing keys:

```sh
$ pacman-key --init
$ pacman-key --populate archlinuxarm
```
* And then upgrade the system:

```
$ pacman -Syu
```

### Choose your terminal and editor

* Afterwards change $TERM ('backspace' key may be a thorn in your foot):
```
$ echo "export TERM=xterm" > $HOME/.bashrc
$ su (passwd = 'root')
$ reboot
```

* Installing your favorite editor:
```
$ su (root)
$ pacman -S vim
$ (quit root mode, Ctrl-D)
$ echo "export EDITOR=vim" >> ~/.bashrc (optional)
```

### Language and Time

Source: [https://wiki.archlinux.org/index.php/Locale](https://wiki.archlinux.org/index.php/Locale)
Source: [https://wiki.archlinux.org/index.php/Time](https://wiki.archlinux.org/index.php/Time)

**Language:**

* Edit `/etc/locale.gen` and uncomment en_US.UTF-8 UTF-8 for American-English:
```
...
#en_SG ISO-8859-1
en_US.UTF-8 UTF-8
#en_US ISO-8859-1
...
```

* Save the file, and generate the locale:
```
$ locale-gen
```

* Set the system locale (in root) ; Use the first part of the previous uncomment line to set the `LANG` variable:
```
$ echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

**Time:**

* Check the current zone defined for the system:
```
timedatectl
```

* If necessary, set the time zone of your system :
```
timedatectl set-timezone Zone/Subzone
(example) timedatectl set-timezone Europe/Paris
```
Get your information: `/usr/share/zoneinfo/<Zone>/<Subzone>`


* The systemd-timesyncd service is available with systemd. To start and enable
it, simply run:

```sh
$ timedatectl set-ntp true
```

### Disable root

```
$ su (root)
$ EDITOR=vim visudo
```
Uncomment this line '%wheel ALL=(ALL) ALL' to allow members of group wheel to execute any command.

* Now, install *sudo*.
```
$ pacman -S sudo
```

* Add a new user, add it in the 'wheel' group:
```
$ useradd -m -G wheel -s /bin/bash <username>
```

- Quit the root-mode and  lock the root-user with the following command:<br>
```
$ sudo passwd -l root
```

If the message "passwd: password expiry information changed". Don't worry.

* Finally, delete the useless user:
```
$ sudo userdel alarm
```

## SSH Server

Source: [https://wiki.archlinux.org/index.php/Secure_Shell](https://wiki.archlinux.org/index.php/Secure_Shell)

### Authentification

* To permit only authentification by authorized keys, edit `/etc/ssh/sshd_config`:
```
PermitRootLogin no
AuthorizedKeysFile .ssh/authorized_keys
PasswordAuthentification no
ChallengeResponceAuthentication no
UsePAM no
PrintMotd no
```

* (optional) A bit more secured to change the port number, for instance:
```
Port 24326 (setting router redirections accordingly)
```

* Restart the SSH deamon:
```
sudo systemctl restart sshd.service
```

### Static IP Address

* Source: [https://wiki.archlinux.org/index.php/Network_configuration#Static_IP_address](https://wiki.archlinux.org/index.php/Network_configuration#Static_IP_address)

### Screen (multiple terminal sessions - for ssh usage)

Screen is like a window manager for your console.<br>
It will allow you to keep multiple terminal sessions running and easily switch between them.<br>
Moreover, it's useful to keep your SSH session running when you disconnect.

* Installation the package:
```
$ sudo pacman -S screen
```

* Useful shortcuts:
  \- Create a window: Ctrl-A and C
  \- Switch windows: Ctrl-A and Ctrl-A
  \- Detach: Ctrl-A and Ctrl-D or `$ screen -d`
  \- Reconnect to a session: `$ screen -r`

## Video and Sound

Source: [http://elinux.org/RPiconfig](http://elinux.org/RPiconfig)<br>
Source: [/boot/overlays/README](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README)

### Enable HDMI

You need to add some line to `/boot/config.txt`.<br>
Just in case, check if the lines below already exist.

* Force the monitor to HDMI mode so that sound will be sent over HDMI cable:

```
$ sudo echo "hdmi_drive=2" >> /boot/config.txt
```

* Use HDMI mode even if no HDMI monitor is detected:

```
$ sudo echo "hdmi_force_hotplug=1" >> /boot/config.txt
```

* Reboot (HDMI will be turned on if the wire is plugged-in).

* Useful commands (not in sudo):
```
$ /opt/vc/bin/tvservice -o (turn off HDMI)
$ /opt/vc/bin/tvservice -p (turn on HDMI with preferred settings)
```

[For more details on /boot/config.txt]([/boot/overlays/README](https://github.com/raspberrypi/firmware/blob/master/boot/overlays/README))

### Enable Sound

Sources: [alsa-project](www.alsa-project.org/main/index.php/Asoundrc), [wiki-arch_1](https://wiki.archlinux.org/index.php/Raspberry_Pi), [wiki-arch_2](https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture#ALSA_Utilities)

* Installation and enable the audio module to the kernel :
```
$ pacman -S alsa-utils pulseaudio pulseaudio-alsa
$ modprobe snd_bcm2835
```

* Set the boot config file and set the card control with amixer:
```
$ echo "dtparam=audio=on" > /boot/config.txt
```

* Select an audio source for output (*Where x corresponds to: 0:Auto, 1:Analog out, 2:HDMI*):
```
$ amixer cset numid=3 x
```

* Make a file called .asoundrc in your home and/or root directory:
```
$ vim /home/xxx/.asoundrc
```

* Copy and paste the following into the file then save it:
```
pcm.!default {
    type hw
    card 0
}

ctl.!default {
    type hw
    card 0
}
```
* Reboot

* Make a test:

```sh
$ sudo aplay /usr/share/sounds/alsa/Front_Center.wav
```

* If it's not loudly enough, try the following command and increase the volume:
```
$ sudo alsamixer
```

## Finally

* These following packages will be quasi indispensable for future usages:
```
$ sudo pacman -S base-devel git wget
```
