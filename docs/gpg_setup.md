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

!!! warning
    It is strongly recommended that you to generate keys on an offline system, such as a live Linux distribution like Ubuntu. Note that with live Linux, certain packages (like scdaemon) may need to be installed manually.

#### Generate a new GPG key

```shell
gpg --expert --full-gen-key
```

When prompted select to specify the key type choose **9** or **1** for RSA if you have a YubiKey NEO.

```shell
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
```

In the next step we are going to specify the "length" of the key. For a newer elliptic curve the recommended value is **1 (Curve 25519)**. For the RSA keys the length depends on the key type.

- For a YubiKey NEO, select **2048**
- For a YubiKey 4 or 5, enter **4096**

```shell
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
```

The next question is about the expiration of our new key. The default value is **unlimited** however, for production use it is recommended to specify this value.

```shell
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 2y
```

Next we have to enter our personal information

- Real Name **Be sure to enter both your first and last name.**
- Email Address
- If desired, a Comment about this key

```shell
GnuPG needs to construct a user ID to identify your key.

Real name: Daniel H******
Email address: daniel@h******.hu
Comment:
You selected this USER-ID:
    "Daniel H****** <daniel@h******.hu>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
```

!!! warning
    Make sure to use a strong passphrase for the key and store it somewhere safe ( It is strongly recommended to use a password manager )

!!! info
    While the key is being generated, move your mouse around or type on the keyboard to gain enough entropy.

When the key is been generated, you will see several messages displayed. Make a note of the key ID, that is displayed in the message such as "```gpg: key E353B******** marked as ultimately trusted```". The key ID in this case is ```E353B********``` and you will need this key ID to perform other operations.

```shell
gpg: revocation certificate stored as 'C:\\Users\\dh***\\AppData\\Roaming\\gnupg\\openpgp-revocs.d\\E353B********.rev'
public and secret key created and signed.

pub   ed25519 2022-12-29 [SC] [expires: 2024-12-28]
      E353B***********************************
uid                      Daniel H****** <daniel@h******.hu>
sub   cv25519 2022-12-29 [E] [expires: 2024-12-28]
```
