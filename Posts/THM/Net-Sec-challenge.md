# Net Sec challenge

![](https://i.imgur.com/2eH4ylG.png)

### Question 1
- What is the highest port number being open less than 10,000?

running an Nmap scan for `All` Ports on the IP Address i have this result:
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- 10.10.174.60                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-01 07:50 EST
Stats: 0:31:41 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 87.14% done; ETC: 08:27 (0:04:40 remaining)
Nmap scan report for 10.10.174.60
Host is up (0.39s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
8080/tcp  open  http-proxy <---- 
10021/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 2278.31 seconds
```
### Question 2
- There is an open port outside the common 1000 ports; it is above 10,000. What is it?

From the last Scan result we will get the answer to this question
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p- 10.10.174.60                                            
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-01 07:50 EST
Stats: 0:31:41 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 87.14% done; ETC: 08:27 (0:04:40 remaining)
Nmap scan report for 10.10.174.60
Host is up (0.39s latency).
Not shown: 65529 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
8080/tcp  open  http-proxy
10021/tcp open  unknown <----

Nmap done: 1 IP address (1 host up) scanned in 2278.31 seconds
```

### Question 3 
- How many TCP ports are open?

Still from the last Scan result we will get the answer to this question

### Question 4
- What is the flag hidden in the HTTP server header?
There are two ways to answer this question `telnet` or `nmap` will work well
#### Using Telnet
the `HTTP` port is 80 so we will use this command to connect to it through telnet
```bash
telnet <IP> 80
```
Then we will need to specify the type of request we want to send `GET` or `POST` , in this scenario we will be using `GET` and this will be the telnet command:
```bash
GET / HTTP/1.1
```
After typing the above command we will need to click `ENTER` twice on our keyboard for the request to be sent to the `http web server`
![](https://i.imgur.com/dJ5QEPo.png)
#### Using nmap
to get the flag with Nmap all we nee to use is the Nmap scripting Engine, we can specify a script with the `--script "<name>"` flag with nmap or just use the default `-sC` which will also work to get the Http Hearder flag:
![](https://i.imgur.com/GCaUQ3Q.png)

### Question 5
- What is the flag hidden in the SSH server header?
There are also two ways to solve this task, just like the last one we can use `nmap` or `telnet`
#### Using Telnet 
we only need to specify the ip and port and telnet will grab the banner which contains the SSH header flag:
![](https://i.imgur.com/ZkHU5gw.png)

#### Using nmap
to use nmap we will use the exact nmap command from the last task task but change  the port `-p` from 80 to 22 :
![](https://i.imgur.com/aq8V2nw.png)


### Question 6
- We have an FTP server listening on a nonstandard port. What is the version of the FTP server?

From the first Nmap scan i didn't get any FTP port open but there is a port that namp tag as unknown.
the port number is `10021` and default FTP port is `21` , so this has to be another Network Admin trying to make it hard for unauthenticated individuals to figure out the FTP port 😏

Enumerating more on that port i confirm that it is indeed an FTP port and i ot the service version
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -p10021 10.10.174.60 -sV                                    
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-02-01 08:48 EST
Nmap scan report for 10.10.174.60
Host is up (0.33s latency).

PORT      STATE SERVICE VERSION
10021/tcp open  ftp     vsftpd 3.0.3 <----
Service Info: OS: Unix

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.13 seconds
```

### Question 7
- We learned two usernames using social engineering: `eddie` and `quinn`. What is the flag hidden in one of these two account files and accessible via FTP?

we will need to brute force for the passwords of both users with `Hydra`
- i crated a file with available usernames
![](https://i.imgur.com/VvSnI3H.png)

- Then i used hydra with the rockyou.txt as the password wordlist`-P` and the user name list i created as the username wordlist `-L`
```bash
┌──(kali㉿kali)-[~]
└─$ hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ftp://<IP>:10021
```

![](https://i.imgur.com/QGjsbsV.png)

I logged in to the FTP server with `Eddie` credentials but i didn't find the flag then i logged in with `quinn` credentials and found it:
![](https://i.imgur.com/D8a0grf.png)

### Question 8
- Browsing to `http://<IP>:8080` displays a small challenge that will give you a flag once you solve it. What is the flag?
![](https://i.imgur.com/uqly5Ir.png)
- Now we need to scan as slow as we can to finish this challenge
- we can use either the FIN scan or NULL scan here
- Using timing template to slow down the scan

```bash
FIN SCAN
sudo nmap -sF <IP>
```

```bash
NULL SCAN
sudo nmap -sN <IP>
```

![](https://i.imgur.com/qae5wxH.png)

## Thanks For Reading
