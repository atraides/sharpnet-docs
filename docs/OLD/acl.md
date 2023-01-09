# ACL setup

``` sh
sudo apt install acl
```

Update /etc/fstab

``` sh
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/disk/by-id/[...]           /       ext4    defaults,acl    0   1
/dev/disk/by-uuid/[...]         /boot   ext4    defaults        0   1
/dev/disk/by-uuid/[...]         /boot   ext4    defaults        0   1
/swap.img                       none    swap    sw              0   0
```
