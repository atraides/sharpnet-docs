# smilpdev8001.sharpnet.sdac

## Pre-requisites

- [SharpNET Controller](https://github.com/atraides/sharpnet-controller)
- [Raspberry PI Imager](https://www.raspberrypi.com/software/)

## Installation

### Install the system using PXE Boot

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
smilpdev8001_become_password: "Password choosen during install"
...
```

#### Run the **initial** SharpNET deployment role

???+ info
    If you see an error message about the Host key make sure to delete the old host key with ssh-keygen, then add the new Host key by logging to the system via SSH.
    ```shell
    ssh-keygen -f ~/.ssh/known_hosts -R "smilpdev8001.sharpnet.sdac"
    ssh-keyscan -H "smilpdev8001.sharpnet.sdac" >> ~/.ssh/known_hosts
    ```

!!! warning
    This playbook usually runs for a long time. On average the execution takes between **20-25 minutes**.

```shell
ansible-playbook -i inventory.yml sharpnet.deployment.fresh_system \
-e shncmd_target_host=smilpdev8001.sharpnet.sdac -e shncfg_reboot=true
```
