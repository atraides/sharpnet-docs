# ZFS on Ubuntu

## Install ZFS utils

``` sh
sudo apt install zfsutils-linux
```

## Import already created pools

``` sh
sudo zpool import
sudo zpool import <poolname>
```

## Upgrade pool

``` sh
sudo zpool upgrade <poolname>
```
