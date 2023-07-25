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

we can see this is a message to user `chris`
