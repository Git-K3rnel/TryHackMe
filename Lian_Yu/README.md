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

so, there is nothing here, not in the page source , we try to `FUZZ` the website to find any new directories

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

according to ticket





















