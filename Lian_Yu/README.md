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

