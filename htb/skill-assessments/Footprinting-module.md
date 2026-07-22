## Skill assessments

## Lab 1 - Easy

# Summary

The lab consist in gather information about the provided server to footprinting the services available. Use this information to look for vulnerabilities and exploits to capture the flag requested.

# The information obtained

I scanned the server initially with nmap. I saw that the server had some open tcp ports like, 22, 21, 2121, and a DNS 53.

Since the description of the lab said that the server had an DNS service, and provided credentials of an user. I started to look into this 52 port, got some subdomains through dig commands but nothing that I can explore more. 

So I decide to test the provided credentials in the task description on the 21 open port. The connection was successful, but the server at this path was restricted, there is nothing in there.

Then I tested the port 2121, again with success connection and now the user ceil got permissions and actually some content inside the server.

# Capturing the flag

Inside the port ftp server 2121 I was able to get the id_rsa file from .ssh dir.

Downloaded the file to my local machine, and I used the command `ssh -i id_rsa ceil@<SERVER_IP>` that way I was able to connect via ssh into the server.

The user ceil as home dir which had a flag dir containing the flag.txt file, just get the content of the file and submited it successfuly.