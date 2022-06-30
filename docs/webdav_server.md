# WebDAV server configuration

``` sh
sudo mkdir /srv/webdav
sudo groupadd
sudo groupadd -g 9001 webdav
sudo useradd 
sudo useradd -M -s /bin/false -g 9001 -u 9001 -d /srv/webdav webdav
sudo setfacl -Rm d:u:atraides:rwx /srv/webdav
sudo setfacl -Rm u:atraides:rwx /srv/webdav
sudo chmod -R g+sw /srv/webdav
mkdir /srv/webdav/data
cd /srv/docker
git pull https://github.com/dgraziotin/docker-nginx-webdav-nononsense.git webdav
vi webdav/docker-compose.yml
docker-compose build
```
