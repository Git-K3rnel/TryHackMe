# Lian_Yu

![cover](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e0b3e246-0819-4181-aaf8-8ead35392b65)


## Enumeration
Let's use nmap to find out which ports are open :

```bash
nmap -sT -sV 10.10.25.101
```
![nmap](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/781bb240-0369-4cbc-9e55-ba3991a93916)

let's check the web page :

![webpage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/61e43eae-f86b-499b-ab84-3645d60a561d)

so, there is nothing here, not in the page source , we try to `FUZZ` the website to find any new directories.

using `raft-medium-directories` i found the `island` direcotry :

![islandPage](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/004d4388-7ff0-4d96-8702-4feb349f720a)

i checked the page source and found `viglante` :

![islandSource](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/a2d3a6d9-ce3b-4cf3-b574-5367490c7c1b)

this seems to be a username or something, so write it down.

according to one of the questions : `What is the Web Directory you found?` and its hint : `In numbers` i though maybe its directory is in numbers so i used this list : 

`raft-large-directories.txt` to FUZZ the `island` directory :

```bash
ffuf -w raft-large-directories.txt -u http://10.10.25.101/island/FUZZ
```

and i found `2100` as the new page :

![2100](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/3cd7f20c-1560-4c16-a038-0115b6130719)

so check the page source :

![2100Source](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/60450c88-7528-41f7-bacc-98dd95c58a14)

according to comment on the page we see `.ticket` can be found on this page , after trying several wordlists i used `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt` :

```bash
ffuf -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.25.101/island/2100/FUZZ.ticket
```

and i found `green_arrow.ticket` file :

![ticket](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/7c03f0b0-f2dc-40fd-abc2-cb78f0465c20)

i first tried to use this password for FTP connection but it didn't work, after try decoding with several encodings i found it is `base58`

so connect to ftp using `vigilante` and the base58 decoded of the passwrod :

![ftp](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/57cc57f4-1340-4ed2-af8d-467d6b6348a5)

## Steganography

I downloaded all three files, and the first file for sure is to check `Leave_me_alone.png` image, if you see it with `exiftool` command. you notice it shows `file format error`

so i checked it with `hexedit` command to see the file signature :

![pnghex](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/5108606b-4262-4afb-b924-420597f87481)

it seems there is no appropriate header, so i tried to edit it , search google with `png magic header` i found one in [Wikipedia](https://en.wikipedia.org/wiki/List_of_file_signatures)

so i edited the hex with `hexedit` program :

![pnghexedit](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/2c681752-7305-4be8-a5a9-f1186a3df0b7)

open the image and you see the password :

![pngFile](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/5f8aced4-05f5-4cf4-a944-86cb4a0642c3)

write it down and check the `aa.jpg` with steghide : 

```bash
steghide extract -sf aa.jpg
```

and provide the password you found in the previous image, by doing that you will get the `ss.zip` file , unzip it and there is 2 files :

`passwd.txt` and `shado` , open shado and there is a password in it

## Gaining Shell

I first tried to ssh into the machine with user vigilante and this password but it didn't work, so it took me a while to recheck my steps.

another connection to ftp and this time i navigated one directory back :

![ftpuser](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/b343dba7-c3c3-4635-95ef-dc84df604ea5)

and yes there is another user `slade` that the password in `shado` file belongs to him/her

ssh to device using slade and the password found in shado file :

![ssh](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/6c5fc68a-1185-464f-bc4d-154107a0ba02)

## Privilege Escalation

Using `sudo -l` and turns out we can use :

```bash
(root) PASSWD: /usr/bin/pkexec
```
if you run this binary you see the help menue :

![bin](https://github.com/Git-K3rnel/TryHackMe/assets/127470407/e0c0dca0-871f-415d-858f-862be13b67ed)

read the man page of this binary and you will find that you can run any command with this binary as root

so simply use :

```bash
/usr/bin/pkexec /bin/bash
```

and you will become root, `cat` the root.txt

```bash
cat /root/root.txt
```

and this is how you can finish this matchine. :)



