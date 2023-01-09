# Install Ubuntu 22.04 on ZFS root filesystem

Original documentation [here](https://openzfs.github.io/openzfs-docs/Getting%20Started/Ubuntu/Ubuntu%2022.04%20Root%20on%20ZFS.html).

## Preparation

1. Boot to a live environment
2. Enable ssh access

    ```sh
    passwd
    # There is no current password.
    sudo apt install --yes openssh-server vim
    ```

3. Install required packages

    ```sh
    sudo -i
    apt install --yes debootstrap gdisk zfsutils-linux
    systemctl stop zed
    ```

4. Select disks for the root pool

    ```sh
    export DISK1="/dev/disk/by-id/wwn-0x50014ee65d5560f6"
    export DISK2="/dev/disk/by-id/wwn-0x50014ee6080048c6"
    ```

5. Turn off swap

    ```sh
    swapoff --all
    ```

6. If re-using disks from an old array, format them

    ```sh
    apt install --yes mdadm

    # See if one or more MD arrays are active:
    cat /proc/mdstat
    # If so, stop them (replace ``md0`` as required):
    mdadm --stop /dev/md0

    # For an array using the whole disk:
    mdadm --zero-superblock --force $DISK1
    # For an array using a partition (e.g. a swap partition per this HOWTO):
    mdadm --zero-superblock --force ${DISK1}-part2
    ```

7. Clear the partition table

    ```sh
    sgdisk --zap-all $DISK1
    sgdisk --zap-all $DISK2
    ```

8. Create bootloader partitions

    ```sh
    sgdisk     -n1:1M:+512M   -t1:EF00 $DISK1
    sgdisk     -n1:1M:+512M   -t1:EF00 $DISK2

    # For legacy (BIOS) booting:
    # sgdisk -a1 -n5:24K:+1000K -t5:EF02 $DISK
    ```

9. Create swap partitions

    ```sh
    sgdisk     -n2:0:+4G    -t2:FD00 $DISK1
    sgdisk     -n2:0:+4G    -t2:FD00 $DISK2
    ```

10. Create boot pool partitions

    ```sh
    sgdisk     -n3:0:+2G      -t3:BE00 $DISK1
    sgdisk     -n3:0:+2G      -t3:BE00 $DISK2
    ```

11. Create root pool partitions

    ```sh
    sgdisk     -n4:0:0        -t4:BF00 $DISK1
    sgdisk     -n4:0:0        -t4:BF00 $DISK2
    ```

12. Create the boot pool

    ```sh
    zpool create \
    -o ashift=12 \
    -o autotrim=on \
    -o cachefile=/etc/zfs/zpool.cache \
    -o compatibility=grub2 \
    -o feature@livelist=enabled \
    -o feature@zpool_checkpoint=enabled \
    -O devices=off \
    -O acltype=posixacl -O xattr=sa \
    -O compression=lz4 \
    -O normalization=formD \
    -O relatime=on \
    -O canmount=off -O mountpoint=/boot -R /mnt \
    bpool mirror \
    ${DISK1}-part3 \
    ${DISK2}-part3
    ```

13. Create the root pool

    ```sh
    zpool create \
    -o ashift=12 \
    -o autotrim=on \
    -O acltype=posixacl -O xattr=sa -O dnodesize=auto \
    -O compression=lz4 \
    -O normalization=formD \
    -O relatime=on \
    -O canmount=off -O mountpoint=/ -R /mnt \
    rpool mirror \
    ${DISK1}-part4 \
    ${DISK2}-part4
    ```

## System installation

1. Create filesystem datasets

    ```sh
    zfs create -o canmount=off -o mountpoint=none rpool/ROOT
    zfs create -o canmount=off -o mountpoint=none bpool/BOOT
    ```

2. Create filesystem datasets for root and boot

    ```sh
    UUID=$(dd if=/dev/urandom bs=1 count=100 2>/dev/null | tr -dc 'a-z0-9' | cut -c-6)

    zfs create -o mountpoint=/ \
        -o com.ubuntu.zsys:bootfs=yes \
        -o com.ubuntu.zsys:last-used=$(date +%s) rpool/ROOT/ubuntu_$UUID

    zfs create -o mountpoint=/boot bpool/BOOT/ubuntu_$UUID
    ```

3. Create datasets

    ```sh
    zfs create -o com.ubuntu.zsys:bootfs=no -o canmount=off \
        rpool/ROOT/ubuntu_$UUID/usr
    zfs create -o com.ubuntu.zsys:bootfs=no -o canmount=off \
        rpool/ROOT/ubuntu_$UUID/var
    zfs create rpool/ROOT/ubuntu_$UUID/var/lib
    zfs create rpool/ROOT/ubuntu_$UUID/var/log
    zfs create rpool/ROOT/ubuntu_$UUID/var/spool

    zfs create -o canmount=off -o mountpoint=/ \
        rpool/USERDATA
    zfs create -o com.ubuntu.zsys:bootfs-datasets=rpool/ROOT/ubuntu_$UUID \
        -o canmount=on -o mountpoint=/root \
        rpool/USERDATA/root_$UUID
    chmod 700 /mnt/root
    ```

4. Create additional datasets

    ```sh
    zfs create rpool/ROOT/ubuntu_$UUID/var/cache
    zfs create rpool/ROOT/ubuntu_$UUID/var/lib/nfs
    zfs create rpool/ROOT/ubuntu_$UUID/var/tmp
    chmod 1777 /mnt/var/tmp
    zfs create -o com.ubuntu.zsys:bootfs=no \
    rpool/ROOT/ubuntu_$UUID/srv
    zfs create -o com.ubuntu.zsys:bootfs=no bpool/grub
    zfs create -o com.ubuntu.zsys:bootfs=no \
    rpool/ROOT/ubuntu_$UUID/tmp
    chmod 1777 /mnt/tmp
    ```

5. Mount tmpfs at /run

    ```sh
    mkdir /mnt/run
    mount -t tmpfs tmpfs /mnt/run
    mkdir /mnt/run/lock
    ```

6. Install the minimal system

    ```sh
    debootstrap jammy /mnt
    ```

7. Copy in the zpool.cache

    ```sh
    mkdir /mnt/etc/zfs
    cp /etc/zfs/zpool.cache /mnt/etc/zfs/
    ```

## Configure the system

### Setup hostname

```sh
hostname smilpdch4000.sharpnet.sdac
hostname > /mnt/etc/hostname
cat >> /mnt/etc/hosts <<-EOF
127.0.1.1   smilpdch4000.sharpnet.sdac
EOF
```

### Setup networking

```sh
cat > /mnt/etc/netplan/01-netcfg.yaml <<-EOF
network:
  ethernets:
    eno49:
      addresses:
        - 10.42.0.11/24
      critical: true
      dhcp-identifier: mac
      gateway4: 10.42.0.1
      nameservers:
        addresses:
          - 10.42.0.1
        search:
          - sharpnet.sdac
          - mgmt.sharpnet.sdac
    eno50:
      addresses:
        - 172.23.0.1/30
      nameservers:
        addresses: []
        search: []
  version: 2
EOF
```

### Configure the package sources

```sh
cat > /mnt/etc/apt/sources.list <<-EOF
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-updates main restricted universe multiverse
deb http://archive.ubuntu.com/ubuntu jammy-backports main restricted universe multiverse
deb http://security.ubuntu.com/ubuntu jammy-security main restricted universe multiverse
EOF
```

### 'Login' to the new system with chroot

```sh
mount --make-private --rbind /dev  /mnt/dev
mount --make-private --rbind /proc /mnt/proc
mount --make-private --rbind /sys  /mnt/sys
chroot /mnt /usr/bin/env DISK1=$DISK1 DISK2=$DISK2 UUID=$UUID bash --login
```

### Configure basic system

```sh
apt update
```

!!! note
    Even if you prefer a non-English system language, always ensure that en_US.UTF-8 is available

```sh
dpkg-reconfigure locales tzdata keyboard-configuration console-setup
apt install --yes vim
```

### Create the EFI filesystem

!!! note
    The -s 1 for mkdosfs is only necessary for drives which present 4 KiB logical sectors (“4Kn” drives) to meet the minimum cluster size (given the partition size of 512 MiB) for FAT32. It also works fine on drives which present 512 B sectors.

!!! warning
    For a mirror or raidz topology, repeat the mkdosfs for the additional disks, but do not repeat the other commands.

```sh
apt install --yes dosfstools

mkdosfs -F 32 -s 1 -n EFI ${DISK1}-part1
mkdosfs -F 32 -s 1 -n EFI ${DISK2}-part1
mkdir /boot/efi
echo /dev/disk/by-uuid/$(blkid -s UUID -o value ${DISK1}-part1) \
    /boot/efi vfat defaults 0 0 >> /etc/fstab
mount /boot/efi
```

### Install GRUB

#### Legacy (BIOS) boot

```sh
apt install --yes grub-pc linux-image-generic zfs-initramfs zsys
```

#### UEFI boot

```sh
apt install --yes \
    grub-efi-amd64 grub-efi-amd64-signed linux-image-generic \
    shim-signed zfs-initramfs zsys
```

### Remove OS prober

!!! notice
    os-prober is only required for dual boot environments

```sh
apt purge --yes os-prober
```

### Set a root password

```sh
passwd
```

### Create swap

```sh
apt install --yes mdadm

# Adjust the level (ZFS raidz = MD raid5, raidz2 = raid6) and
# raid-devices if necessary and specify the actual devices.
mdadm --create /dev/md0 --metadata=1.2 --level=mirror \
    --raid-devices=2 ${DISK1}-part2 ${DISK2}-part2
mkswap -f /dev/md0
echo /dev/disk/by-uuid/$(blkid -s UUID -o value /dev/md0) \
    none swap discard 0 0 >> /etc/fstab
```

### Mount a tmpfs

```sh
cp /usr/share/systemd/tmp.mount /etc/systemd/system/
systemctl enable tmp.mount
```

### Create additional groups

```sh
addgroup --system sharpadm
addgroup --system docker
addgroup --system sambashare
```

### Install openssh-server

```sh
apt install --yes openssh-server
```

### Create admin user

```sh
useradd -m -g sharpadm sharpadm
passwd sharpadm
mkdir -p ~sharpadm/.ssh
chown sharpadm:sharpadm ~sharpadm/.ssh
chmod 0700 ~sharpadm/.ssh
cat > /home/sharpadm/.ssh/authorized_keys <<-EOF
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFV3lop3Yp2eiNsmQjBw0fe3+vqXBkIr2LmtEml/sjSd
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPf25WfayakmxiBDz0JV2R6JsuehfEXftrCRZcWfcjED
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHDOs9+sk4vmRakxW7B4u9wsCvHnctdnlVkSjSEMXbP0 weltar@gmail.com
EOF
```
