# Install / Configuration guide for `smilprpi0011.sharpnet.sdac`

## Pre-requisites

- SharpNET Controller
- Raspberry PI Imager - [download](https://www.raspberrypi.com/software/)

## Install

First step is to create a new Ubuntu "installation" for our Raspberry System using the RPi Imager and an Ubuntu 22.04 Server Image.

1. Click "**CHOOSE OS**" and select "**Other general-purpose OS**"
   ![Choose OS](images/RPI-Imager/RPi-Image-011.png)
   ![General-purpose OS](images/RPI-Imager/RPi-Image-014.png)
2. Select "**Ubuntu**"
   ![Ubuntu](images/RPI-Imager/RPi-Image-013.png)
3. Select "**Ubuntu Server 22.04.X LTS (64-bit)**"
   ![Ubuntu Server](images/RPI-Imager/RPi-Image-012.png)
4. Click "**CHOOSE STORAGE**" and select your SD Card/Hard Disk
   ![Choose Storage](images/RPI-Imager/RPi-Image-010.png)
   ![Choose MMC/HDD](images/RPI-Imager/RPi-Image-009.png)
5. Click on the :fontawesome-solid-gear: icon
6. ![Click options](images/RPI-Imager/RPi-Image-001.png)
7. Change the system's hostname
   ![Change hostname](images/RPI-Imager/RPi-Image-008.png)
8. Enable SSH and add your SSH public key
   ![Enable SSH](images/RPI-Imager/RPi-Image-005.png)
9. Set the install user and it's password
   ![Set the install user](images/RPI-Imager/RPi-Image-007.png)
10. Set the locales and "**Save**"
   ![Set locales](images/RPI-Imager/RPi-Image-006.png)
11. Click on "**WRITE**"
   ![Write to disk](images/RPI-Imager/RPi-Image-002.png)
12. Confirm the overwrite of **all data on the drive**
   ![Confirm Overwrite](images/RPI-Imager/RPi-Image-004.png)
13. Wait for the Write process to finish
   ![Finish](images/RPI-Imager/RPi-Image-000.png)

## Docker

### Portainer

```yaml title="