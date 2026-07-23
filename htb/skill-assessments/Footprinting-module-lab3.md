# Lab 3 - Hard

## Summary

In this lab I had an MX and Management service for the internal Network. Where we need to again find the password of the user HTB.

## Information Gathering

Using nmap `sudo nmap -sV -sC -A SERVER_IP --top-ports=1000` I find different services in different ports open.

    - Port 22 ssh OpenSSH 8.2
    - PORT 110,143,993,995 for IMAP and POP3 services
    - Port UDP 161 a SNMP service.

Using the flag -sU in nmap scan I found that 161 port was open with SNMP service. But I'm not have been started in this port. 

After looking in this scan result I was thinking where to start first, since the assessment description says that the server have the function of a backup server for the internal accounts I started to enumerate the IMAP/POP3 ports to see if I find something.

## Enumeration process

I tried first if the ports of IMAP and POP3 would accept a connection without credentials, but it's not worked. 

So I was thinking to get some type of credentials in those emails services, but I can't connect in them. I started to look for suggestions online and figure out that try to enumerate the port 161 would help me.

I enumerate this port using the tools: snmpwalk, onesixtyone, and braa. The results showed that a script called `tom_recover.sh` was executed and some logs said that tom is already a valid user of the interanl network. The output of the script was logged as well and leaked the password of tom.

## More enumeration and Exploit

With these credentials I was able to log in the IMAP service and look for message inside the mailboxes. One email was still there and this email was leaking a private ssh key.

```
HELO dev.inlanefreight.htb
MAIL FROM:<tech@dev.inlanefreight.htb>
RCPT TO:<bob@inlanefreight.htb>
DATA
From: [Admin] <tech@inlanefreight.htb>
To: <tom@inlanefreight.htb>
Date: Wed, 10 Nov 2010 14:21:26 +0200
Subject: KEY

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAACFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAgEA9snuYvJaB/QOnkaAs92nyBKypu73HMxyU9XWTS+UBbY3lVFH0t+F...

```

With that private key I thought now I'm able to log into the server via ssh as user tom. It worked and I started to look into the server directories and files to see if I find any clue about the user HTB.

After spend time open files and dirs, I found nothing relevant. So I decide to make a cat command into the path `/etc/passwd` to look if is there any other hidden user.

Doing that I see that a mysql database was running on the server, and I thought that maybe tom is authorized to connect in this database. Again it worked and connected into the database I was able to retrieve the password of user HTB.

## Final Thoughts

The module was very good I learned about some services that is used in a network enviroment. I learned how to enumerate them and look for misconfigurations and vulnerabilities that an attacker can explore. 

The 3 labs was good as well and I think I learn a lot with them, unlocked new skills and feel more prepare to new challenges.