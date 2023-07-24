# Brute It

![logo](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/648413ba-68d8-4bce-beb7-be79cce9134e)

## Reconnaissance

Let's nmap scan the target to find open ports :

```bash
nmap -sT -sV 10.10.140.109
```

![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/b9f1c508-92b8-4632-9ea2-65ae4c455893)

Ports `80` and `22` is open

* question #1 : How many ports are open? :  `2`

* question #2 : What version of SSH is running? : `OpenSSH 7.6p1`

* question #3 : What version of Apache is running? : `2.4.29`

* question #4 : Which Linux distribution is running? : `ubuntu`

Open a browser and navigate to matchine IP address, there is a default apache page

then we FUZZ the target with ffuff :

```bash
ffuf -w ~/wordlist/raft-medium-directories.txt  -u http://10.10.140.109/FUZZ -fc 403
```

![image](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/9f0755dd-14aa-407a-ab24-447dee18da11)

yes, there is an admin directory which we found with fuzzing

* question #5 : What is the hidden directory? : `/admin`

Next, browse to `/admin` directory :

![login](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/599e9b71-9b59-43e3-a175-f172f4b15a97)

SQLi does not work here, but when you see the page source you find a comment :

```html
<!-- Hey john, if you do not remember, the username is admin -->
```

we can now brute force the login page with `hydra`, to do this we need to find the exact parameters that is being sent to the server

as page source form element shows two parameters (user, pass) are sent when you submit the form.

so we need to inform hydra with this information :

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.140.109 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid" -v
```

after a minute or two hydra will find the password for you :

![hydra](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/c1f7e047-4d5c-40f9-8c85-f944bd9ea79b)

now login the website with `admin:xavier`

after logging in the page shows a flag and a link to download a private key

![afterlogin](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/0b1827cb-001f-4fbe-b764-9264bc7e5a98)

download the private key which for user "john" and change the permission to `600`
```bash
chmod 600 john.key
```





















