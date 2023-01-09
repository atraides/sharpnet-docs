# How to disable systemd-resolver under Ubuntu

```sh
sudo systemctl disable systemd-resolved
sudo systemctl stop systemd-resolved
sudo rm /etc/resolv.conf
cat <<EOF | sudo tee /etc/resolv.conf
search mgmt.sharpnet.sdac sharpnet.sdac
nameserver 10.42.0.241
nameserver 10.42.0.242
nameserver 10.42.0.1
EOF
```
