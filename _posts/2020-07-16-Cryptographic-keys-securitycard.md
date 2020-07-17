---
title: 'Managing the Cryptographic Keys with hardware security Card'
date: 2020-07-16 00:00:00
featured_image: '/images/posts/gpg-keys-yubi.jpg'
excerpt: Asymmetric/Public Key Cryptography is a great way for Confidentiality, Integrity and Authentication. It overcomes the key exchange issues that generally happens with Symmetric key encryption.  
---

![](/images/posts/yubikey.png){:height="40%" width="40%"}

Asymmetric/Public Key Cryptography is a great way for Confidentiality, Integrity and Authentication. It overcomes the key exchange issues that generally happens with Symmetric key encryption. On the other side one has to really maintain the keys securely and also use it appropriately.
One of the best way is to store the keys in a hardware tokens such as Yubikey.

The below article describes the procedure on how to store the keys in Hardware token (using, Yubikey) and the to achieve below regular activities:
- Message or File Decryption
- Signing the GitHub commits
- Logging into servers over SSH    

The following setup is based of debian based operating system.

### Installing Required Packages
First lets install the required packages for the Operating System:

```bash
sudo apt update
sudo apt install -y \
    curl gnupg2 gnupg-agent \
    cryptsetup scdaemon pcscd \
    yubikey-personalization \
    dirmngr \
    secure-delete \
    hopenpgp-tools
```

### Generating a Master Key
If you don't have a key pair already, lets generate a new one:
```
gpg --full-gen-key
```

1. Select `Option 1`, which generates the key with RSA algorithm along with encryption subkey.
2. Provide the key size as `4096` bits.
3. Optionally specify the key expiry time, but can select `0` for life long validity.
4. Provide Name, Email and Comment and select `O` to generate the new key.
5. Provide a passphrase for your private key.
6. A new key pair will be generated and looks something like below.

```
pub   rsa4096 2020-07-17 [SC]
      993ECC149B693F4496D3987BC911A16425694327
uid                      Test-Key (This is a demo key) <test@example.com>
sub   rsa4096 2020-07-17 [E]
```
From above you can see, the Master key was generated and has a key id `993ECC149B693F4496D3987BC911A164256943271` and also a subkey for Encryption [E].

### Generating Subkeys
Lets generate separate sub-keys for Signing and Authentication purposes by editing the Master key.
For sake of simplicity, lets create save the key id in a variable

```bash
export KEYID=993ECC149B693F4496D3987BC911A16425694327
```
Editing the Master key:

```bash
gpg --expert --edit-key $KEYID
```
This will open the gpg prompt.

#### Subkey for Signing

Lets generate a subkey for Signing purposes - like code signing etc.

```bash
gpg> addkey
```
1. Select Option 4 for creating the signing key with RSA.
2. Repeat the same steps as in Master Key generation. But its a good practice to define the subkey validity, say 1y for 1 year.
3. Once the key is generated, you will see new entry in the key list with usage as S, as shown below.

```bash
ssb  rsa4096/A5361560938D5F1D
     created: 2020-07-17  expires: 2021-07-17  usage: S   
[ultimate] (1). Test-Key (This is a demo key) <test@example.com>
```

#### Subkey for Authentication
Lets continue within `gpg` prompt to generate a separate sub-key for Authentication purposes. By default gpg dont have option for Authentication in its menu.

```bash
gpg> addkey
```
1. Select Option 8 to generate key with our own capabilities.
2. By default the Current allowed actions: Sign Encrypt; we need to toggle these and get Authentication.
3. Select Option S to disable Sign
4. Select Option E to disable Encrypt
5. Select Option A to Enable Authentication
6. Now the option should looks like Current allowed actions: Authenticate
7. Select Option Q to quit the menu and go back for key generation.
8. Repeat the key generation options like in Signing subkey with a validity, say 1y for 1 year.
9. Once the key is generated, you will see a new entry in the key list as A, as shown below.

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

### Backup the Keys

#### Verify
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
You can modify the validity of any sub-key with `expire` option with gpg edit-key.

#### Backup for offline storage
It is important to export the keys to a file and storage in a safe medium.

```bash
#exporting private key  
$ gpg --armor --output private_key.gpg --export-secret-key $KEYID
#exporting sub keys    
$ gpg --armor --output sub_keys.gpg --export-secret-subkeys $KEYID
#exporting public key    
$ gpg --armor --output public_key.gpg --export $KEYID

```
Move the above generated files (.gpg) to a secure medium and share the `public_key.gpg` to the whole world by posting to key servers.   

### Transfer the keys to Hardware (Yubikey)
It's time to transfer the three sub keys Encryption,  to the Hardware device (Yubikey)

1. Plug your Yubikey device  
2. Verify the device by `gpg --card-status`.
3. Enter into the key edit mode as done earlier, by `gpg --edit-key $KEYID`.
4. You will see a total of four keys. One Sec and three ssb's.
5. Select the first ssb with option `gpg> key 1`.
6. This will select the Encryption key and you will notice a star (ssb*).
7. Use `keytocard` option to transfer the key to the card.
8. The gpg will prompt, under which section the key needs to be stored on the card. Select Encryption.
9. Deselect the key by `gpg> key 1` and select Signing key by `gpg> key 2`.
10. Repeat the steps 7 and 8. Select appropriate storage option.
11. Repeat the same for Authentication key.
12. Save and quit.
13. Verify the keys on the smart card by `gpg --card-status` and the keys information under General Key info.

### Delete the Master Private key
1. Verify the above backup step is completely done.
2. We don't need the Private key on the computer anymore, delete it by `$ gpg --delete-secret-keys $KEYID`.
3. Verify the keys on the card with `gpg --card-status` and you will now notice `sec#`, under General Key info.

At this point, we have all our Private Key and its sub keys on the card and only public key on the computer.

### Generate SSH key

We need to tell gpg-agent to handle the requests from SSH, add the below line to the `gpg-agent.conf` file.

```bash
$ echo enable-ssh-support >>  ~/.gnupg/gpg-agent.conf
```

Notify the agent, which key id need to use:
```bash
$ echo $KEYID >> ~/.gnupg/sshcontrol
```

Last, we need to tell the SSH how to access the gpg-agent.

```bash
$ echo "export SSH_AUTH_SOCK=$(gpgconf --list-dirs agent-ssh-socket)" >> ~/.bashrc
$ echo "gpgconf --launch gpg-agent" >> ~/.bashrc
```

Extract the SSH public key
```
$ ssh-add -L >> id_rsa_card.pub
```
Transfer this ssh public key to a desired remote server and add to authorized_keys file.

## Use Cases
Lets perform Confidentiality, Signing and Authentication use cases.
First, unplug the key card from the computer.

### Use Case 1 - Confidentiality
Lets perform the encryption and decryption of a message.

```bash
#Encryption
$ echo "This is a secret message" | gpg --encrypt --recipient $KEYID > message.gpg

#Decryption
$ gpg --decrypt --recipient $KEYID < message.gpg
#This will prompt to plug in the key card and for user pin.
```

### Use Case 2 - Code Signing
Sign the git commits
Unplug the key card from the computer

```bash
# Make Sure the Signing is enabled for commits for your repo.
$ git config commit.gpgsign true  

# Tell Git which key needs to use for commits
$ git config --global user.signingkey $KEYID

# Perform a commit
$ git commit -S -m "commit message"

#This will prompt to plug in the key card and for user pin.

```

### Use Case 3 - Authentication
Unplug the key card from the computer

```bash
#ssh to the server where the ssh public key is transfered
$ ssh user@server

#This will prompt to plug in the key card and for user pin.
```
