# Kenobi
![](https://i.imgur.com/mmjgn6M.png)

We need to run a scan to know what is going on on our targets network
### Scan results
```bash
┌──(kali㉿kali)-[~]
└─$ nmap -A 10.10.226.52 -T5   
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-29 05:25 EST
Warning: 10.10.226.52 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.226.52
Host is up (0.29s latency).
Not shown: 964 closed tcp ports (conn-refused), 29 filtered tcp ports (no-response)
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
| http-robots.txt: 1 disallowed entry 
|_/admin.html
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100005  1,2,3      34475/udp6  mountd
|   100005  1,2,3      53789/udp   mountd
|   100005  1,2,3      53833/tcp   mountd
|   100005  1,2,3      60453/tcp6  mountd
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 2h01m48s, deviation: 3h27m51s, median: 1m47s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2024-01-29T10:28:22
|_  start_date: N/A
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2024-01-29T04:28:22-06:00
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 55.05 seconds
```

- From the above results we see that out target has 7 ports open
- we can get more information about our target from the open SMB port `139` and `445`
```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse <IP>
```
![](https://i.imgur.com/RZM1Od8.png)
- we have 3 shares available on SMB port `455`
- one of the share that stands out is the anonymous share:
```bash
\\10.10.226.52\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
```
it has a user home directory path
- we can use a tool called `smbclient` to enumerate further on that share
```bash
smbclient //<IP>/<SHARE>
```
![](https://i.imgur.com/aXX8We6.png)
- i used the get command to download a file that is available in the share to my own machine.

- reading the log file i see Information generated for Kenobi when generating an SSH key for the user
- and Information about the ProFTPD server
![](https://i.imgur.com/g57aec2.png)

>[!summary] 
our earlier nmap port scan has shown port 111 running the service rpcbind. This is just a server that converts remote procedure call (RPC) program number into universal addresses. When an RPC service is started, it tells rpcbind the address at which it is listening and the RPC program number its prepared to serve.

- In our case, port 111 is access to a network file system. Lets use nmap to enumerate this:
```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount <IP>
```
![](https://i.imgur.com/WUoOYRL.png)

From the above output we can see that there is a mountable NFS share `/var`

Going back to the initial nmap scan the first open port:
```bash
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
```
- ProFTPD version 1.3.5  is running
we can search for possible exploit on searchsploit and we have 4 possible exploits
![](https://i.imgur.com/ctbPdvF.png)
#### About the exploit
The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the file system to a chosen destination.

We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. 

so we can use Netcat to connect to the FTP server and use the vulnerable copy module to copy the private ssh key of the user kenobi `/home/kenobi/.ssh/id_rsa` to the mountable share `/var` 
![](https://i.imgur.com/KpFvRzt.png)

Lets mount the /var/tmp directory to our machine
```bash
mkdir /mnt/kenobiNFS
mount 10.10.226.52:/var /mnt/kenobiNFS
ls -la /mnt/kenobiNFS
```
![](https://i.imgur.com/HZAkMYS.png)
then  i SSH into kenobi's account with the private key (using sudo)
![](https://i.imgur.com/0oaFTVx.png)

## Privilege Escalation with Path Variable manipulation

To search the a system for file with a SUID bit run the following: 
```bash
find / -perm -u=s -type f 2>/dev/null 
```
![](https://i.imgur.com/oIJQoyN.png)

we found some files, but the one that is odd from the results is the menu file.
- the `/usr/bin` directory is a directory that is usually in the PATH variable:
![](https://i.imgur.com/giTWmMv.png)
- this means if i just type `menu` in the CLI it should do something

![](https://i.imgur.com/6jfFj26.png)

See): ?

then i use the `strings` command to looks for human readable strings on a binary:
![](https://i.imgur.com/g5TnHHF.png)
- This shows us the binary is running without a full path (e.g. not using /usr/bin/curl or /usr/bin/uname)
- As this file runs as the root users privileges, we can manipulate our path gain a root shell.
![](https://i.imgur.com/K4es2mq.png)
We copied the /bin/sh shell, called it uname, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "uname" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!

## Thanks For Reading
<button onclick="window.location.href='https://hassans-sec.github.io';">Back To Home</button>
