# Chocolate Factory

![Charlie](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/8b5d62d5-05e3-4c30-a535-8acbab677d09)

## Enumeration

We begin by enumerating the target :

```bash
nmap -sT -sV 10.10.198.134
```
![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/005c2016-e177-4bdd-99fd-482577aae1cf)

as it shows there are pleny of ports open like `21` , `22`, `80` and other ports.

on port 80 you can see there is a login page :

![loginpage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/7cf3893c-0f7a-44bf-a7ab-374b23889d0d)

but we don't have any credentials, and **SQLi** attack does not work here.

let's fuzz the website with `ffuf` :
```bash
ffuf -w ~/wordlist/raft-medium-files.txt  -u http://10.10.198.134/FUZZ -fc 403
```

![ffuf](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/f894ce42-01c1-4dde-a574-2eb132bdbece)

yes , we have found 2 new directory : `home.php` and `validate.php`

navigate to `home.php` and there is a input box which you can type a command :

![home](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/46db25de-4d09-4703-8c44-3e267a9fc820)

use `ls` command to see the contents of the current directory :
```
 home.jpg home.php image.png index.html index.php.bak key_rev_key validate.php 
```
let's download the `index.php.bak` file



we first try to login to ftp with `anonymous` user :

```bash
ftp 10.10.198.134
```

![ftp](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/01c16df5-2a33-4e7b-b872-22228a3746e1)

download the image with `get gum_room.jpg` and look to see if any data is embedded inside it :
```bash
steghide info gum_room.jpg
```
