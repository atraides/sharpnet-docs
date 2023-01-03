# smilprpi0010.sharpnet.sdac

## Pre-requisites

- [SharpNET Controller](https://github.com/atraides/sharpnet-controller)
- [Raspberry PI Imager](https://www.raspberrypi.com/software/)

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

``` yaml title="vault.yml" hl_lines="2"
...
smilprpi0010_become_password: "Password choosen during install"
...
```

#### Run the **initial** SharpNET deployment role

???+ info
    If you see an error message about the Host key make sure to delete the old host key with ssh-keygen, then add the new Host key by logging to the system via SSH.
    ```shell
    ssh-keygen -f ~/.ssh/known_hosts -R "smilprpi0010.sharpnet.sdac"
    ssh-keyscan "smilprpi0010.sharpnet.sdac" >> ~/.ssh/known_hosts
    ```

!!! warning
    This playbook usually runs for a long time. On average the execution takes between **20-25 minutes**.

```shell
ansible-playbook -i inventory.yml -e target=smilprpi0010.sharpnet.sdac \
  server_first_boot.yml
```

<div id="rpi-success-asciinema" style="z-index: 1; position: relative; max-width: 100%;"></div>

### Check the Step CA status

```shell
sudo systemctl status step-ca
```

``` text title="sudo systemctl status step-ca" hl_lines="3"
● step-ca.service - step-ca
     Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-01-03 16:11:38 CET; 1s ago
   Main PID: 20168 (sh)
      Tasks: 10 (limit: 4414)
     Memory: 15.1M
        CPU: 312ms
     CGroup: /system.slice/step-ca.service
             ├─20168 /bin/sh -c "/usr/local/bin/step-ca /etc/step-ca/config/ca.json"
             └─20169 /usr/local/bin/step-ca /etc/step-ca/config/ca.json
```

???+ warning
    If the service status show as `activating` for a long time we may have a problem with our step-ca installation.
    ```shell hl_lines="2 3"
    ● step-ca.service - step-ca
        Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
        Active: activating (auto-restart) (Result: exit-code) since Sun 2023-01-01 20:10:41 CET; 7s ago
        Process: 2364 ExecStart=/bin/sh -c /usr/local/bin/step-ca /etc/step-ca/config/ca.json (code=exited, status=2)
    Main PID: 2364 (code=exited, status=2)
            CPU: 106ms
    ```
    Follow [these](Troubleshooting/step-ca.md) steps for troubleshooting.

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
dig +noall +answer stepca.sharpnet.sdac @smilprpi0010.sharpnet.sdac
```

``` text
❯ dig +noall +answer stepca.sharpnet.sdac @smilprpi0010.sharpnet.sdac
stepca.sharpnet.sdac.   3600    IN      A       10.42.0.10
```

### (Optional) Install the Argon One driver

!!! info
    If the Raspberry case being used for the server is an [Argon One](https://www.argon40.com/products/argon-one-m-2-case-for-raspberry-pi-4) only then we should install this driver.

```shell
curl https://download.argon40.com/argon1.sh | bash
```

#### Configure the Argon One fans

``` shell
sudo argonone-config
```

``` text
--------------------------------------
Argon One Fan Speed Configuration Tool
--------------------------------------
WARNING: This will remove existing configuration.
Press Y to continue:y
Thank you.

Select fan mode:
  1. Always on
  2. Adjust to temperatures (55C, 60C, and 65C)
  3. Customize behavior
  4. Cancel
NOTE: You can also edit /etc/argononed.conf directly
Enter Number (1-4):2

Please provide fan speeds for the following temperatures:
55C (0-100 only):40
60C (0-100 only):60
65C (0-100 only):80
Configuration updated.
```

<script>
  window.onload = function(){
    AsciinemaPlayer.create('/static/recordings/asciinema/smilprpi0010-config.cast', document.getElementById('rpi-success-asciinema'), {
        poster: 'npt:1:23',
        rows: 20
    });
}
</script>