USEFUL
===============

## Free up space on microSD Card

### Pacman cache

Source: [https://wiki.archlinux.org/index.php/Pacman](https://wiki.archlinux.org/index.php/Pacman)

Only do this when certain that previous package versions are not required:
```
$ pacman -Sc
```

It is possible to empty the cache folder fully with:
```
$ pacman -Scc
```

You can also define how many recent versions you want to keep:
```
$ paccache -rk 1
```

To remove all cached versions of uninstalled packages, re-run paccache with:
```
$ paccache -ruk0
```

### Reduce log size

Edit `/etc/systemd/journald.conf` and set the `SystemMaxUse` variable according to your need:
```
[Journal]
SystemMaxUse=50M
SystemMaxFileSize=10M
```

## Increase the microSD card lifetime

Edit `/etc/fstab`:
```
# <file system>	<dir>	<type>	<options>	<dump>	<pass>
/dev/mmcblk0p2  /  ext4  defaults,nodiratime,noatime,discard  0  0
```
(`/dev/mmcblk0p2` can be replaced according to your configuration)
