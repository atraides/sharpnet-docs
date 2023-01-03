# smilprpi0011.sharpnet.sdac

## Pre-requisites

- SharpNET Controller
- Raspberry PI Imager - [download](https://www.raspberrypi.com/software/)

## Installation

First step is to create a new Ubuntu "installation" for our Raspberry System using the RPi Imager and the Ubuntu 22.04 Server Image.

### "Burn" the Ubuntu Server image to the MMC/SSD

- Start **Raspberry Pi Imager**

![Start the Imager](/images/RPI-Imager/Start%20RPI%20Imager.png){ align=center loading=lazy }

- Click "**CHOOSE OS**" and select "**Other general-purpose OS**"

![Choose OS](/images/RPI-Imager/RPi-Image-011.png){ align=center loading=lazy }
![General-purpose OS](/images/RPI-Imager/RPi-Image-014.png){ align=center loading=lazy }

- Select "**Ubuntu**"

![Ubuntu](/images/RPI-Imager/RPi-Image-013.png){ align=center loading=lazy }

- Select "**Ubuntu Server 22.04.X LTS (64-bit)**"

![Ubuntu Server](/images/RPI-Imager/RPi-Image-012.png){ align=center loading=lazy }

- Click "**CHOOSE STORAGE**" and select your SD Card/Hard Disk

![Choose Storage](/images/RPI-Imager/RPi-Image-010.png){ align=center loading=lazy }
![Choose MMC/HDD](/images/RPI-Imager/RPi-Image-009.png){ align=center loading=lazy }

- Click on the :fontawesome-solid-gear: icon

![Click options](/images/RPI-Imager/RPi-Image-001.png){ align=center loading=lazy }

- Change the system's hostname

![Change hostname](/images/RPI-Imager/RPi-Image-008.png){ align=center loading=lazy }

- Enable SSH and add your SSH public key

![Enable SSH](/images/RPI-Imager/RPi-Image-005.png){ align=center loading=lazy }

- Set the install user and it's password

![Set the install user](/images/RPI-Imager/RPi-Image-007.png){ align=center loading=lazy }

- Set the locales and "**Save**"

![Set locales](/images/RPI-Imager/RPi-Image-006.png){ align=center loading=lazy }

- Click on "**WRITE**"

![Write to disk](/images/RPI-Imager/RPi-Image-002.png){ align=center loading=lazy }

- Confirm the overwrite of **all data on the drive**

![Confirm Overwrite](/images/RPI-Imager/RPi-Image-004.png){ align=center loading=lazy }

- Wait for the Write process to finish

![Finish](/images/RPI-Imager/RPi-Image-000.png){ align=center loading=lazy }

## Configuration

### Configure the system with Ansible

???+ info
    During the first boot it take some time for the system to apply all the settings we requested. Before we do anything else make sure the cloud-init finished the setup.

    ![Cloud Init Done](/images/Cloud-Init-Done.png){ align=center loading=lazy }

#### Preparation

Before we can run the playbook make sure that the password we choose during the install matches the one in the password vault.

``` shell
ansible-vault edit vault.yml
```

``` yaml hl_lines="9"
---
sharpnet_cfg_admin_password: "<some random password>"
sharpnet_cfg_become_password: "<some random password>"

sharpnet_cfg_mrsharp_admin_password: "<some random password>"
sharpnet_cfg_atraides_admin_password: "<some random password>"

smilprpi0010_become_password: "<some random password>"
smilprpi0011_become_password: "Password choosen during install"
```

#### Run the **initial** SharpNET deployment role

???+ info
    If you see an error message about the Host key make sure to delete the old host key with ssh-keygen, then add the new Host key by logging to the system via SSH.
    ```shell
    ssh-keygen -f ~/.ssh/known_hosts -R "smilprpi0011.sharpnet.sdac"
    ssh-keyscan "smilprpi0011.sharpnet.sdac" | ssh-keygen -lf -
    ```

!!! warning
    This playbook usually runs for a long time. On average the execution takes between **20-25 minutes**.

```shell
ansible-playbook -i inventory.yml -e target=smilprpi0011.sharpnet.sdac server_first_boot.yml
```

<div id="rpi-fail-asciinema" style="z-index: 1; position: relative; max-width: 100%;"></div>

???+ info
    If the playbook fails with the bellow error message about reloading the `step-ca` service, make sure the `yubikey` is connected to the raspberry.
```shell
Unable to start service step-ca: A dependency job for step-ca.service failed.
```

<div id="rpi-success-asciinema" style="z-index: 1; position: relative; max-width: 100%;"></div>

### Verify the system's Yubikey access

```shell
❯ ykman info
Device type: YubiKey 5 Nano
Serial number: 13******
Firmware version: 5.4.3
Form factor: Nano (USB-A)
Enabled USB interfaces: OTP, FIDO, CCID

Applications
FIDO2           Enabled
OTP             Enabled
FIDO U2F        Enabled
OATH            Enabled
YubiHSM Auth    Enabled
OpenPGP         Enabled
PIV             Enabled
❯ ykman list
YubiKey 5 Nano (5.4.3) [OTP+FIDO+CCID] Serial: 13******
```

### Check the Step CA status

```shell
sudo systemctl status step-ca
```

If the status show as `activating` for a long time it's very likely that the permissions on the $STEP_CA directory is incorrect. Make sure the files are owned by the step user and group.

```shell hl_lines="2"
● step-ca.service - step-ca
     Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
     Active: activating (auto-restart) (Result: exit-code) since Sun 2023-01-01 20:10:41 CET; 7s ago
    Process: 2364 ExecStart=/bin/sh -c /usr/local/bin/step-ca /etc/step-ca/config/ca.json (code=exited, status=2)
   Main PID: 2364 (code=exited, status=2)
        CPU: 106ms
```

```shell
sudo chown -R step.step /etc/step-ca
sudo systemctl restart step-ca
```

After this the Step CA should work

```shell
sudo systemctl status step-ca
● step-ca.service - step-ca
     Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2023-01-01 20:21:02 CET; 1s ago
   Main PID: 3305 (sh)
      Tasks: 10 (limit: 2080)
     Memory: 15.6M
        CPU: 231ms
     CGroup: /system.slice/step-ca.service
             ├─3305 /bin/sh -c "/usr/local/bin/step-ca /etc/step-ca/config/ca.json"
             └─3306 /usr/local/bin/step-ca /etc/step-ca/config/ca.json

Jan 01 20:21:02 smilprpi0011 sh[3306]: badger 2023/01/01 20:21:02 INFO: Replay took: 7.516656ms
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Starting Smallstep CA/ (linux/arm64)
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Documentation: https://u.step.sm/docs/ca
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Community Discord: https://u.step.sm/discord
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Config file: /etc/step-ca/config/ca.json
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 The primary server URL is https://smilprpi0011.sharpnet.sdac>
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Root certificates are available at https://smilprpi0011.shar>
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Additional configured hostnames: 10.42.0.241
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 X.509 Root Fingerprint: 37d6d******************************6>
Jan 01 20:21:02 smilprpi0011 sh[3306]: 2023/01/01 20:21:02 Serving HTTPS on :443 ...
```

### Install the SharpNET Root and Intermediate CA

```shell
curl -LO http://smilpdch4000.sharpnet.sdac:32080/static/certs/sharpnet-root.crt
curl -LO http://smilpdch4000.sharpnet.sdac:32080/static/certs/sharpnet-intermediate.crt
sudo mv *.crt /usr/local/share/ca-certificates
sudo update-ca-certificates
```

### Configure CoreDNS

#### Clone the CoreDNS

```shell
cd /srv/docker
git clone https://git.sharpnet.sdac/homelab/sharpnet-dns.git coredns
```

#### Start the CoreDNS container

```shell
cd /srv/docker/coredns
docker-compose up -d
```

#### Test that the DNS server is working

```shell
❯ dig +noall +answer stepca.sharpnet.sdac @smilprpi0011.sharpnet.sdac
stepca.sharpnet.sdac.   3600    IN      A       10.42.0.10
```

### (Optional) Install the Argon One "driver"

!!! info
    If the Raspberry case being used for the server is an [Argon One](https://www.argon40.com/products/argon-one-m-2-case-for-raspberry-pi-4) only then we should install this driver.

```shell
curl https://download.argon40.com/argon1.sh | bash
```

<script>
  window.onload = function(){
    AsciinemaPlayer.create('/images/asciinema/rpi0011-fail.cast', document.getElementById('rpi-fail-asciinema'), {
        poster: 'npt:1:23',
        rows: 13
    });
    AsciinemaPlayer.create('/images/asciinema/rpi0011-success.cast', document.getElementById('rpi-success-asciinema'), {
        poster: 'npt:1:23',
        rows: 13
    });
}
</script>