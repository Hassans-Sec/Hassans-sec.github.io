
![](https://i.imgur.com/NjhFdlh.png)

# Step 1:
run a scan against the ip address of your target:

![](https://i.imgur.com/V6m2SvM.png)

From the above result we have 2 open port and the only one with more value is the port 80 (web server):

navigating to the web page we can only find a default apache web-page which isn't helpful in this situation so let fuzz for directories

![](https://i.imgur.com/4vZWk32.png)

# Directory Busting

trying to fuzz for directories i tested different tools but the output/discovered directories are not useful but `feroxbuster` worked and found some great looking `hidden` directory

```bash
❯ feroxbuster -u http://10.10.145.43 -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

![](https://i.imgur.com/RLpnl9o.png)

upon navigating to the page we will find an `rsa identity file` 

![](https://i.imgur.com/UYAREjN.png)

i downloaded the file and now i have the rsa file but no username so i no i'm not done yet and i went back to the default apache web page and view sourcecode and found  a comment in the code

![](https://i.imgur.com/OWf9VzN.png)

now let log-in:

![](https://i.imgur.com/eYfqWUZ.png)

and found the user flag:

![](https://i.imgur.com/MXD29dO.png)

# Escalating privilege

![](https://i.imgur.com/ivqDKNm.png)

running `sudo -l` shows that they can run `wget` as sudo without a password :

Assuming that we already know the path of the target file, `wget` offers a number of options for file read and file upload. With `sudo` access, we can therefore read and extract any file on the system. [GTFObins gives the following two examples](https://gtfobins.github.io/gtfobins/wget/) of file upload and file read actions with `wget` :

![](https://i.imgur.com/o1eO9jF.png)

This is good enough to get the root flag (assuming we know that the root flag is at /root/root_flag.txt - which it is). But we can also use `wget`'s file write feature to provide a root shell by overwriting `/etc/crontab`. For this to work, we first need to check that `cron` is running on the target machine :

![](https://i.imgur.com/ls9tcrn.png)

Perfect. The plan is to create a copy of the existing `/etc/crontab` file that adds a new job that is run as root. We can then host this copy on the attack machine, download it to the target machine with `wget` as sudo, and overwrite the existing `/etc/crontab` file (the new job will be applied automatically without requiring a restart of the `cron` service).

we create a reverse shell payload and also copy `/etc/crontab` from the target machine and add a new root job that creates a file in `/home/jessie`

![](https://i.imgur.com/wk6VOyn.png)

![](https://i.imgur.com/p8nkchO.png)

transfer the payload to the example directory to the directory in the edited cronjob file :

![](https://i.imgur.com/j6zmSki.png)

then also transfer the edited cronjob file using `sudo` and using the `-o` flag to save it in the real cronjob directory to overwrite the real one:
![](https://i.imgur.com/igxq27b.png)

then we need to quickly start up a listener on our machine with the port we set in the edited cronjob script, and we wait to get a root shell:

![](https://i.imgur.com/c2YuA5P.png)
