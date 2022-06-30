# Remove snapd from Ubuntu

``` sh
snap list
sudo snap remove each_item # (by dependency order)
sudo umount /snap/core/xxxx # On 20.04, on 20.10 /var/snap
sudo apt purge snapd
sudo apt-mark hold snapd
```

Clear various files at `/home/*/snap`, `/usr/lib/snap` and alike

``` sh
rm -rf ~/snap
sudo rm -rf /snap
sudo rm -rf /var/snap
sudo rm -rf /var/lib/snapd
sudo rm -rf /var/cache/snapd/
```
