![](https://i.imgur.com/kXP4vxS.png)

- choosing my target:
```bash
❯ nmap -sn 10.150.150.10-254
```

![](https://i.imgur.com/mYxUSR8.png)

- network mapping and target scanning:
![](https://i.imgur.com/Px6xxb5.png)
- My target isn't running alot of services but the one of more value is the `FTP` port.
- so i searched for possible exploit for the `FTP` version (vsftpd 2.0.8) and found this:
![](https://i.imgur.com/rpO5MmA.png)

![](https://i.imgur.com/eEhxZDj.png)

```bash
❯ nmap --script ftp-vsftpd-backdoor -p 21 10.150.150.12
```

![](https://i.imgur.com/VcTZiM8.png)


- then use metasploit to exploit the target using the module:
```
exploit/unix/ftp/vsftpd_234_backdoor
```

![](https://i.imgur.com/FhnnUKo.png)

- and here is the flag:
![](https://i.imgur.com/J4YfWR9.png)

![](https://i.imgur.com/6rxqXCp.png)
