# How to setup GPG and SSH agent

## Linux

Original documentation [here](https://wiki.archlinux.org/title/GnuPG#gpg-agent)

```sh
gpg --full-gen-key --expert
```

## Windows

### Requirements

- A compatible Yubikey
- A current version of GnuPG software
  - Windows: [GPG4Win](https://www.gpg4win.org/download.html)
  - macOS: [GPG Suite](https://gpgtools.org/gpgsuite.html)
  - Linux: Pre-installed on most distributions

### Generating Keys externally

!!! notice
It is strongly recommended that you to generate keys on an offline system, such as a live Linux distribution like Ubuntu. Note that with live Linux, certain packages (like scdaemon) may need to be installed manually.

- Insert the Yubikey into the USB port if it is not already plugged in.
- Enter the GPG command: ```gpg --expert --full-gen-key```
- When prompted to specify the key type, enter ```(9) ECC (sign and encrypt)``` and press Enter.
- Specify the "size" of key you want to generate. Recommended is ```(1) Curve 25519```
- Specify the expiration date of the key, and press Enter. Verify the expiration date when prompted.
- Now enter your user information. 
  - Enter your Real Name and press Enter. **Be sure to enter both your first and last name.**
  - Enter your Email Address and press Enter.
  - If desired, enter a Comment about this key, and press Enter.
- Review the information you entered, make any changes if necessary.
- If all information is correct, enter O (for Okay) and press Enter.
- Enter your passphrase for the new key __(You will need this to move the key to your yubikey)__

!!! info
While the key is being generated, move your mouse around or type on the keyboard to gain enough entropy.

When the key has been generated, you will see several messages displayed. Make a note of the key ID, that is displayed in the message such as "```gpg: key 1234ABC marked as ultimately trusted```". The key ID in this case is ```1234ABC``` and you will need this key ID to perform other operations.

#### Example

```shell
PS C:\Users\dhagy> gpg --expert --full-gen-key
gpg (GnuPG) 2.4.0; Copyright (C) 2021 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (13) Existing key
  (14) Existing key from card
Your selection? 9
Please select which elliptic curve you want:
   (1) Curve 25519 *default*
   (2) Curve 448
   (3) NIST P-256
   (4) NIST P-384
   (5) NIST P-521
   (6) Brainpool P-256
   (7) Brainpool P-384
   (8) Brainpool P-512
   (9) secp256k1
Your selection? 1
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
Key expires at 28-Dec-24 19:07:23 Central Europe Standard Time
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Daniel Hagyarossy
Email address: daniel@hagyarossy.hu
Comment:
You selected this USER-ID:
    "Daniel Hagyarossy <daniel@hagyarossy.hu>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: revocation certificate stored as 'C:\\Users\\dhagy\\AppData\\Roaming\\gnupg\\openpgp-revocs.d\\E353B22F4B32095EBA750326EC79A3F0D7542233.rev'
public and secret key created and signed.

pub   ed25519 2022-12-29 [SC] [expires: 2024-12-28]
      E353B22F4B32095EBA750326EC79A3F0D7542233
uid                      Daniel Hagyarossy <daniel@hagyarossy.hu>
sub   cv25519 2022-12-29 [E] [expires: 2024-12-28]
```
