
![](https://i.imgur.com/CBJU8BB.png)

```bash
hashid 48bb6e862e54f2a795ffc4e541caed4d

md5
```

```bash
❯ john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

![](https://i.imgur.com/NlVafHN.png)



```bash
hashid CBFDAC6008F9CAB4083784CBD1874F76618D2A97

sha-1
```

```bash
❯ john --format=raw-sha1 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

![](https://i.imgur.com/Qg6wgsY.png)



```bash
❯ hashid 1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032

sha-256
```

```bash
❯ john --format=raw-sha256 --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

![](https://i.imgur.com/Nb1feD6.png)



```bash
$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom
```

The hash `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom` is a bcrypt hash.

### Breakdown of the Bcrypt Hash

- `$2y$`: The hash type identifier for bcrypt.
- `12`: The cost factor, which determines the computational complexity. Here, it is 12.
- The remainder of the string is the salt and the hashed password.

- making a copy of the wordlist but with only 4 characters long words

```bash
❯ grep -E '^.{4}$' /usr/share/wordlists/rockyou.txt > four_char_words.txt 
```

### Explanation
- `grep`: The command-line utility for searching plain-text data.
- `-E`: Enables extended regular expressions.
- `'^.{4}$'`: The regular expression to match lines with exactly 4 characters.
    - `^`: Anchors the match at the start of the line.
    - `.{4}`: Matches exactly 4 characters of any kind.
    - `$`: Anchors the match at the end of the line.
- `original_wordlist.txt`: The input wordlist file.
- `> four_char_words.txt`: Redirects the output to a new file named `four_char_words.txt`.

```bash
❯ hashcat -m 3200 -a 0 hashes.txt four_char_words.txt 
```

![](https://i.imgur.com/X5UCiSO.png)

```bash
❯ hashid 279412f945939ba78ce0758d3fd83daa

md4
```

[online-cracker](https://hashes.com/en/decrypt/hash )

![](https://i.imgur.com/TRjwcEo.png)

## Level 2

```
F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85
```

![](https://i.imgur.com/6pjgyMK.png)


```
1DFECA0C002AE40B8619ECF94819CC1B
```

![](https://i.imgur.com/DflJ4It.png)


```
Hash: $6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.

Salt: aReallyHardSalt
```

the hash you provided is indeed salted. The salt is `aReallyHardSalt`. Both John the Ripper and Hashcat are capable of handling salted hashes.

![](https://i.imgur.com/chJOPAx.png)

- making a copy of the wordlist but with only `six` characters words

```bash
❯ grep -E '^.{6}$' /usr/share/wordlists/rockyou.txt > six_char_words.txt 
```

```bash
❯ hashcat -m 1800 -a 0 hashes.txt six_char_words.txt
```

### Explanation of Hashcat Command

- `hashcat`: The command to run Hashcat.
- `-m 1800`: Specifies the hash type as SHA-512 crypt.
- `-a 0`: Specifies the attack mode (0 for a dictionary attack).
- this attack may take some time

![](https://i.imgur.com/rJVqagr.png)


```
Hash: e5d8870e5bdd26602cab8dbe07a942c8669e56d6

Salt: tryhackme
```

![](https://i.imgur.com/Sc4ndqv.png)

To crack a salted HMAC-SHA1  hash using Hashcat, you need to specify the correct hash mode for salted HMAC-SHA1. The mode for salted HMAC-SHA1 in Hashcat is  `160`. Below are the steps to crack the hash `e5d8870e5bdd26602cab8dbe07a942c8669e56d6` with the salt `tryhackme`.

```bash
❯ hashcat -m 160 e5d8870e5bdd26602cab8dbe07a942c8669e56d6:tryhackme /usr/share/wordlists/rockyou.txt
```

![](https://i.imgur.com/VFGst4j.png)
