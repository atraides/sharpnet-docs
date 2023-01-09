# Docker Controller Hub

## Install docker

``` sh
sudo apt install docker docker-compose
sudo mkdir /srv/docker
sudo chown root:docker /srv/docker
```

``` sh title="Add the current admin user to the docker group"
sudo usermod -a -G docker sharpadm
sudo usermod -a -G docker atraides
```

``` sh title="Set the correct permission for the docker folder"
sudo chown -R root:docker /srv/docker
sudo chmod g+sw /srv/docker
sudo setfacl -m d:g:docker:rwx /srv/docker
```

## Create ans iSCSI target and target-port-group

``` sh
targetcli
```
