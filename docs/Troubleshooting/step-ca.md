# Step CA

## YubiKey access

### Verify keys on the YubiKey

``` shell
ykman piv info
PIV version: 5.4.3
PIN tries remaining: 5/5
Management key algorithm: TDES
CHUID:	3019d4e739**********************************************************************************************************
CCC: 	No data available.
Slot 9a:
	Algorithm:	ECCP256
	Subject DN:	CN=SharpNET Root CA,O=SharpNET
	Issuer DN:	CN=SharpNET Root CA,O=SharpNET
	Serial:		5799446925****************************
	Fingerprint:	37d6d9c639******************************************************
	Not before:	2022-12-28 22:20:20
	Not after:	2032-12-25 22:20:20
Slot 9c:
	Algorithm:	ECCP256
	Subject DN:	CN=SharpNET Intermediate CA,O=SharpNET
	Issuer DN:	CN=SharpNET Root CA,O=SharpNET
	Serial:		1775652118****************************
	Fingerprint:	770245a330******************************************************
	Not before:	2022-12-28 22:20:21
	Not after:	2032-12-25 22:20:21
```

## Step CA configuration

### Verify the `step` user and group

The `step` user and `step` group should already exist in the system with the UID of **9501** and GID of **9501**

``` shell
❯ id step
uid=9501(step) gid=9501(step)
```

### Verify the `$STEPPATH` directory

!!! info
    All files and directories should be owned by the user **step** and the group **step**

``` shell
❯ export STEPPATH=/etc/step-ca
ls -ld ${STEPPATH}
ls -ltr ${STEPPATH}
drwxr-xr-x 6 step step 4096 Jan  3 16:07 /etc/step-ca
total 16
drwx------ 2 step step 4096 Jan  3 16:06 templates
drwx------ 2 step step 4096 Jan  3 16:07 certs
drwx------ 2 step step 4096 Jan  3 16:08 config
drwx------ 2 step step 4096 Jan  3 16:47 db

```

## Rebuild Step CA

!!! danger
    Rebuilding Step CA should be a last resort. Following these steps will delete all previous configuration, database entries, users etc. After the rebuild we will end up with an empty Certificate Authority, with all provision users destroyed.

### Remove the YubiKey from the server

To prevent the step-ca service restarting we have to remove the YubiKey with the SSL keys from the server.

### Verify that the step-ca service is stopped

``` shell
sudo systemctl status step-ca
```

``` shell title="sudo systemctl status step-ca" hl_lines="3"
○ step-ca.service - step-ca
     Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
     Active: inactive (dead)
```

### Set the `$STEPPATH` envirnoment variable

``` shell
export STEPPATH=/etc/step-ca
```

### Remove the old configuration folder

``` shell
sudo rm -rf ${STEPPATH}
```

### Recreate the configuration folder

``` shell
sudo mkdir -p ${STEPPATH}
sudo chown step.step ${STEPPATH}
```

### Re-Initialize the Step CA

``` shell
sudo --preserve-env step ca init --name="SharpNET Root CA" \
     --dns="stepca.sharpnet.sdac,10.42.0.10" --address=":443" \
     --provisioner="daniel@hagyarossy.hu"
```

``` text hl_lines="3"
Use the arrow keys to navigate: ↓ ↑ → ← 
? What deployment type would you like to configure?: 
  ▸ Standalone - step-ca instance you run yourself
    Linked - standalone, plus cloud configuration, reporting & alerting
    Hosted - fully-managed step-ca cloud instance run for you by smallstep
✔ Deployment Type: Standalone
Choose a password for your CA keys and first provisioner.
✔ [leave empty and we'll generate one]: 

Generating root certificate... done!
Generating intermediate certificate... done!

✔ Root certificate: /etc/step-ca/certs/root_ca.crt
✔ Root private key: /etc/step-ca/secrets/root_ca_key
✔ Root fingerprint: 541c4796245acd99b66b11b2ef9bf239c22a07278450a1e985f84a3e5abb995c
✔ Intermediate certificate: /etc/step-ca/certs/intermediate_ca.crt
✔ Intermediate private key: /etc/step-ca/secrets/intermediate_ca_key
✔ Database folder: /etc/step-ca/db
✔ Default configuration: /etc/step-ca/config/defaults.json
✔ Certificate Authority configuration: /etc/step-ca/config/ca.json

Your PKI is ready to go. To generate certificates for individual services see 'step help ca'.

FEEDBACK 😍 🍻
  The step utility is not instrumented for usage statistics. It does not phone
  home. But your feedback is extremely valuable. Any information you can provide
  regarding how you’re using `step` helps. Please send us a sentence or two,
  good or bad at feedback@smallstep.com or join GitHub Discussions
  https://github.com/smallstep/certificates/discussions and our Discord 
  https://u.step.sm/discord.
```

### Add an ACME provisioner

``` shell
sudo step ca provisioner add acme --type acme --ca-config ${STEPPATH}/config/ca.json
```

### Remove the generated certificates

``` shell
sudo sh -c "rm ${STEPPATH}/certs/*.crt"
```

### Download our root and intermediate certs

``` shell
curl -fsSL http://smilpdch4000.sharpnet.sdac:32080/static/certs/sharpnet-root.crt -o - | sudo tee ${STEPPATH}/certs/root_ca.crt 2>&1 >/dev/null
curl -fsSL http://smilpdch4000.sharpnet.sdac:32080/static/certs/sharpnet-intermediate.crt -o -| sudo tee ${STEPPATH}/certs/intermediate_ca.crt 2>&1 >/dev/null
```

### Remove the generated keys

``` shell
sudo rm -rf ${STEPPATH}/secrets
```

### Edit the Step CA config

The top of the `#!shell ${STEPPATH}/config/ca.json` should look like this:

``` json title="${STEPPATH}/config/ca.json" hl_lines="5-9"
{
        "root": "/etc/step-ca/certs/root_ca.crt",
        "federatedRoots": null,
        "crt": "/etc/step-ca/certs/intermediate_ca.crt",
        "key": "yubikey:slot-id=9c",
        "kms": {
            "type": "yubikey",
            "pin": "******"
        },
        "address": ":443",
...
```

### Make sure all `step-ca` files are owned by `step`

``` shell
sudo chown -R step:step ${STEPPATH}
```

### Reconnect the YubiKey and verify step-ca is running

At this stage we can reconnect the YubiKey to the Raspberry and step-ca should start automatically. We can verify the step-ca status using `systemctl`

``` shell
sudo systemctl status step-ca
```

``` text title="sudo systemctl status step-ca" hl_lines="3"
● step-ca.service - step-ca
     Loaded: loaded (/etc/systemd/system/step-ca.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2023-01-03 18:02:55 CET; 2s ago
   Main PID: 21826 (sh)
      Tasks: 11 (limit: 4414)
     Memory: 14.8M
        CPU: 247ms
     CGroup: /system.slice/step-ca.service
             ├─21826 /bin/sh -c "/usr/local/bin/step-ca /etc/step-ca/config/ca.json"
             └─21833 /usr/local/bin/step-ca /etc/step-ca/config/ca.json
```

### Sign a test certificate

#### Initialize the Step CA environment

!!! warning
    The following command will create an environment in your user `#!shell ${HOME}` directory. However, the `#!shell ${STEPPATH}` variable interfere with this. Make sure you unset `#!shell ${STEPPATH}` before running the below commands.
    ``` shell
    unset STEPPATH
    ```

``` shell
step ca bootstrap --ca-url="https://stepca.sharpnet.sdac" \
--fingerprint 37d6d9c639503c7178aeadddd201754ad3265822e562f73ac241b9af5a925360
```

#### Generate a test certificate for **localhost**

``` shell
step ca certificate "localhost" localhost.crt localhost.key
```

``` text title='step ca certificate "localhost" localhost.crt localhost.key'
✔ Provisioner: daniel@hagyarossy.hu (JWK) [kid: vx-rj1A3k59nDzdFmxBd2oehUFB6NFE8ThEY4Z5xayE]
Please enter the password to decrypt the provisioner key: 
✔ CA: https://stepca.sharpnet.sdac
✔ Certificate: localhost.crt
✔ Private Key: localhost.key
```

#### Verify the new certificate is signed

``` shell
step certificate inspect localhost.crt --short
```

``` text title="step certificate inspect localhost.crt --short" hl_lines="3"
X.509v3 TLS Certificate (ECDSA P-256) [Serial: 2057...1521]
  Subject:     localhost
  Issuer:      SharpNET Intermediate CA
  Provisioner: daniel@hagyarossy.hu [ID: vx-r...xayE]
  Valid from:  2023-01-03T17:13:52Z
          to:  2023-01-04T17:14:52Z
```

#### Remove the test certificate

``` shell
rm localhost.{key,crt}
```