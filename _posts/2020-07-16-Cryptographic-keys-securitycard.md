---
title: 'Managing the Crypto Keys with a hardware security Card for Confidentiality, Integrity and Authentication'
date: 2020-07-16 00:00:00
featured_image: '/images/posts/gpg-keys-yubi.jpg'
excerpt: Asymmetric/Public Key Cryptography is a great way for Confidentiality, Integrity and Authentication. It overcomes the key exchange issues that generally happens with Symmetric key encryption.  
---

![](/images/posts/yubikey.png){:height="40%" width="40%"}

*[Image](https://www.yubico.com/products/): Yubikeys from Yubico*


Asymmetric(Public Key) Cryptography is a great way to maintain Confidentiality, Non-repudiation and Authentication. This method overcomes the key exchange issues that generally happens with Symmetric key encryption but on the other side one has to really keep the other half of the key (private key) secure and also use it appropriately.

One of the convenient way is to store the keys in a hardware security card such as Yubikey and Google Titan.

This page describes the procedure on how to store the keys in a hardware security card and how to use it for regular activities like:
- Message or file decryption
- Signing the GitHub commits
- Logging into servers over SSH    

**This procedure is based on Debian based Operating System and for Yubikey.**  
Before proceed please make sure you have your security card admin and user password. The default admin password is `12345678` and the user password is `123456` for Yubikey.  

### 1. Installing required Packages
Let's first install the required packages for the Debian based operating System:

```bash
$ sudo apt update
$ sudo apt install -y \
    curl gnupg2 gnupg-agent \
    cryptsetup scdaemon pcscd \
    yubikey-personalization \
    dirmngr \
    secure-delete \
    hopenpgp-tools
```

### 2. Generating a Master Key
If you don't have a key pair already, lets generate a new one or if you already have one, you can skip this step.
```
gpg --full-gen-key
```

1. Select `option 1`, which generates the keys with RSA algorithm along with encryption subkey.
2. Provide the key size as `4096` bits for better security.
3. Optionally specify the key expiry time, but you can select `0` for life long validity.
4. Provide Name, Email and Comment and select `O` (as in Okay) to generate the new key.
5. Provide a passphrase for your private key in the prompt.
6. A new key pair will be generated and looks something like below.

```
pub   rsa4096 2020-07-17 [SC]
      993ECC149B693F4496D3987BC911A16425694327
uid                      Test-Key (This is a demo key) <test@example.com>
sub   rsa4096 2020-07-17 [E]
```
From above you can notice, the Master key was generated and with a key id `993ECC149B693F4496D3987BC911A164256943271` and also a subkey for Encryption, denoted by [E].

### 3. Generating Subkeys
Lets generate separate subkeys for Signing and Authentication purposes by editing the Master key.
For sake of simplicity, lets create save the key id in a variable and can use the variable for rest of the process.

```bash
#Assign your key id to KEYID variable
$ export KEYID=993ECC149B693F4496D3987BC911A16425694327
```
Editing the Master key:

```bash
$ gpg --expert --edit-key $KEYID
```
This will open the gpg prompt.

#### 3a. Subkey for Signing

Generate a subkey for Signing purposes - like code signing etc.

```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 16 Jul 2021 10:21:57 PM EDT
Is this correct? (y/N) y
Really create? (y/N) y
```

1. Select `option 4` for creating the signing key with RSA.
2. Mention the key size as 4096 bits.
3. Define the expiry of the subkey, say `1y` for 1 year.
3. Once the key is generated, you will see new entry in the key list with usage as *S*, as shown below.

```bash
ssb  rsa4096/A5361560938D5F1D
     created: 2020-07-17  expires: 2021-07-17  usage: S   
[ultimate] (1). Test-Key (This is a demo key) <test@example.com>
```

#### 3b. Subkey for Authentication
Lets continue within `gpg` prompt to generate a separate sub-key for Authentication purposes. By default gpg didn't have an option for Authentication in its menu. So we will generate with own capabilities option.

```
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
  (14) Existing key from card
Your selection? 8

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Sign Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Encrypt

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? E

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions:

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? A

Possible actions for a RSA key: Sign Encrypt Authenticate
Current allowed actions: Authenticate

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 1y
Key expires at Fri 16 Jul 2021 10:37:49 PM EDT
Is this correct? (y/N) y
Really create? (y/N) y

```
1. Select option 8 to generate key with our own capabilities.
2. By default - Current allowed actions: Sign Encrypt; we need to toggle these and have only Authentication.
3. Select option S to disable the Sign
4. Select option E to disable the Encrypt
5. Select option A to enable the Authentication
6. Now the option should looks like Current allowed actions: Authenticate
7. Select option Q to quit the menu and go back for key generation.
8. Repeat the key generation options like in Signing subkey with a validity of 1 year.
9. Once the key is generated, you will see a new entry in the key list with usage as A.

```bash
ssb  rsa4096/ECCA4FC9281139C6
     created: 2020-07-17  expires: 2021-07-17  usage: A   
[ultimate] (1). Test-Key (This is a demo key) <test@example.com>
```

Quit the prompt and save the changes:
```bash
gpg> quit
Save changes? (y/N) y
```

### 4. Backup the Keys

#### 4a. Verify
Verify the keys that are generated:

```bash
$ gpg --list-secret-keys

------------------------------
sec   rsa4096 2020-07-17 [SC]
      993ECC149B693F4496D3987BC911A16425694327
uid           [ultimate] Test-Key (This is a demo key) <test@example.com>
ssb   rsa4096 2020-07-17 [E]
ssb   rsa4096 2020-07-17 [S] [expires: 2021-07-17]
ssb   rsa4096 2020-07-17 [A] [expires: 2021-07-17]
```
You can modify the validity of any sub-key with `expire` option and also with `adduid` you can add additonal identities with gpg edit-key. Please refer `help` option within gpg.

#### 4b. Backup for offline storage
It is important to export the keys to a file and storage in a safe medium.

```bash
#exporting the private key  
$ gpg --armor --output private_key.gpg --export-secret-key $KEYID
#exporting sub keys    
$ gpg --armor --output sub_keys.gpg --export-secret-subkeys $KEYID
#exporting the public key    
$ gpg --armor --output public_key.gpg --export $KEYID

```
Move the above generated files (.gpg) to a secure medium and share the `public_key.gpg` to rest of the world by posting to key servers or to your website or to my favorite keybase profile .   

### 5. Transfer the keys to Security Card (Yubikey)  
It's time to transfer the keys to the hardware security device, Yubikey.   

1. Plug-in your Yubikey device to the computer.
2. Verify the device status by `gpg --card-status`.
3. Enter into the key edit mode as did earlier, by `gpg --edit-key $KEYID`.
4. You will see a total of four keys. One sec and three ssb's.
5. Select the first ssb with option `gpg> key 1`.
6. This will select the Encryption key and you will notice a star (ssb*).  
7. Type `keytocard` to transfer the key to the card.
8. This will ask under which section the key needs to be stored on the card. Select Encryption.
9. Deselect the key by `gpg> key 1` and select Signing key by `gpg> key 2`.
10. Repeat the steps 7 and 8. Select appropriate storage option.
11. Repeat the same for Authentication key.
12. Save and quit.
13. Verify the keys on the smart card by `gpg --card-status` and the keys information will be under General Key info.

```
#selecting sub key

$gpg --edit-key $KEYID

sec  rsa4096/C911A16425694327
     created: 2020-07-17  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/36986E709A44F19E
     created: 2020-07-17  expires: never       usage: E   
ssb  rsa4096/A5361560938D5F1D
     created: 2020-07-17  expires: 2021-07-17  usage: S   
ssb  rsa4096/ECCA4FC9281139C6
     created: 2020-07-17  expires: 2021-07-17  usage: A   
[ultimate] (1). Test-Key (This is a demo key) <test@example.com>

gpg> key 1

sec  rsa4096/C911A16425694327
     created: 2020-07-17  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb* rsa4096/36986E709A44F19E
     created: 2020-07-17  expires: never       usage: E   
ssb  rsa4096/A5361560938D5F1D
     created: 2020-07-17  expires: 2021-07-17  usage: S   
ssb  rsa4096/ECCA4FC9281139C6
     created: 2020-07-17  expires: 2021-07-17  usage: A   
[ultimate] (1). Test-Key (This is a demo key) <test@example.com>  

```

### 6. Delete the Private key
1. Verify and make sure the backup step (4b) is successfully completed.
2. We don't need the private key on the computer anymore, delete it by `$ gpg --delete-secret-keys $KEYID`.
3. Verify the keys on the card with `gpg --card-status` and you will now notice `sec#`, under General Key info.

At this point, we have our master keys and its sub keys on the card safely and only the public keys on the computer.

### 7. Setup SSH and key

We need to let *gpg-agent* to handle the requests from SSH, add the below line to the `gpg-agent.conf` file:

```bash
$ echo "enable-ssh-support" >>  ~/.gnupg/gpg-agent.conf
```

Notify the agent, which key id it should use during the ssh session establishment:
```bash
$ echo $KEYID >> ~/.gnupg/sshcontrol
```

Let the *SSH* know how to access the gpg-agent:

```bash
$ echo "export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)" >> ~/.bashrc
$ echo "gpgconf --launch gpg-agent" >> ~/.bashrc
```

Extract the SSH public key to a file:
```bash
$ ssh-add -L >> id_rsa_card.pub
```
Transfer this ssh public key to the desired remote server and add it to authorized_keys file.

## 8. Use Cases

We have completed all the required setup.   
Lets validate the setup by performing Confidentiality, Signing and Authentication use cases.  
Before we proceed further, please unplug the security key card from the computer to notice the prompts for the device during the operations.  

### Use Case 1 - Confidentiality
Validating the encryption and decryption process:

```bash
#Encryption
$ echo "This is a secret message" | gpg --encrypt --recipient $KEYID > message.gpg

#Decryption
$ gpg --decrypt --recipient $KEYID < message.gpg
#This will prompt to plug in the security card and for user pin.
```

### Use Case 2 - Code Signing
Validating the signing process with GitHub:  
Add the public_key.gpg to your GitHub account.  
*Tip:* Make sure key uid email address is same as Git commits email address.

```bash
# Please make sure the Signing is enabled for commits to your repo.
$ git config commit.gpgsign true  

# Tell Git which key needs to use for commits
$ git config --global user.signingkey $KEYID

# Perform a commit
$ git commit -S -m "commit message"

#This will prompt to plug in the security card and for user pin.  

```

### Use Case 3 - Authentication
Validating the ssh login with a server

```bash
#ssh to the server where the ssh public key is transfered
$ ssh user@server

#This will prompt to plug in the security card and for user pin.  
```

Though the overall setup process looks lengthy,  it is a one-time process and makes easy to maintain and manage the keys.
You can carry the hardware security card along with you everyday and use the appropriate keys where needed by just plugging the card. This will also enable the Multi Factor Authentication (MFA) with "something you have (Card) and something you know (Card PIN)".
