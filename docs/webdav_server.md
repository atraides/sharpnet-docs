# WebDAV server configuration

``` sh
cd /srv/docker
git clone git@github.com:atraides/docker-nginx-webdav-nononsense.git webdav
vi webdav/docker-compose.yml
docker-compose build
```

```yaml title="docker-compose.yml"
version: '3.9'
services:
    webdav:
        container_name: webdav
        build:
            context: .
        volumes:
            - /srv/webdav/data:/data
              # - /srv/webdav:/etc/nginx/htpasswd
        environment:
            - PUID=9001
            - PGID=9001
            - TZ=Europe/Budapest
            - SERVER_NAMES=smilpdch4000.sharpnet.sdac
            - TIMEOUTS_S=1200 # these are seconds
            - CLIENT_MAX_BODY_SIZE=120M # must end with M(egabytes) or G(igabytes)
        ports:
          - 32080:80
```
