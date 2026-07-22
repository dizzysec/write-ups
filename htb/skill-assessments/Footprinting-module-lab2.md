# Lab 2 - Medium

## Summary

The objective of this lab was enumerate the Windows server target, escalate priviligies, and extract the password of the user 'HTB' using SQL Server Management Studio (SSMS).

## Information gathering 

As usual I started the TCP port scan on the provided server IP to indentify running services and potential entry points.

```
sudo nmap SERVER_IP -sV -sC -A --top-ports=1000
```

Key Findings:

    - Port 445 (SMB): Standard Windows file sharing.

    - Port 2049 (NFS): Network File System. Finding NFS on a Windows machine is highly unusual and a strong indicator of a potential misconfiguration or intended file-sharing mechanism.

    - Port 3389 (RDP): Remote Desktop Protocol open, allowing graphical interface access if credentials can be obtained.

    - Port 1433 (SQL): Notably absent from the external scan, indicating the database is only accessible internally (localhost).

## Vulnerability Assessment

I Tried first to check for exposed directories using showmount in the NFS.

`showmount -e 10.129.202.41`

This revelead to me the directory `/TechSupport` exported to everyone.

To explore the content of this dir I used. 

```
# Create local mount point
mkdir /tmp/techsupport

# Mount the remote share
sudo mount -t nfs SERVER_IP:/TechSupport /tmp/techsupport
```

With access to the content of TechSupport dir I found a file named ticket.txt that was a conversation of two employees that was leaking some credentials.

    - Username: alex
    - Password: lol123!mD

## Lateral Movement & Enumeration via SMB

With a valid set of credentials, the focus shifted to verifying password reuse across other services. The credentials were authenticated against the SMB service using smbclient to map available network shares.

`smbclient -L //10.129.202.41 -U 'alex'`

Key Findings:
In addition to default Windows shares (ADMIN$, C$, IPC$, Users), a non-standard custom share was discovered: devshare.

By connecting directly to //SERVER_IP/devshare and authenticating as alex, further enumeration of the development files was conducted, ultimately leading to the discovery of the Administrator password for the machine.

## Privilege Escalation & Objective Execution

With the local Administrator credentials secured, a Remote Desktop session was established to access the machine's GUI, which was required to interact with the internal SQL database.

`xfreerdp /v:SERVER_IP /u:Administrator /p:'<recovered_admin_password>' /cert:ignore`

## Once inside the Windows desktop environment:

1. Launched Microsoft SQL Server Management Studio (SSMS).

2. Bypassed standard SQL authentication by connecting to the database engine using Windows Authentication. Because the RDP session was running under the Administrator context, SSMS automatically granted top-level sysadmin rights to the database.

3. Navigated the Object Explorer: Databases -> [Target Database] -> Tables.

4. Located the target user table containing account information.

5. Used the GUI feature "Edit Top 200 Rows" to view the table's contents in cleartext.

**Result:** Successfully located the HTB user record within the database table and extracted the final cleartext password, completing the objective.

## Vulnerability Summary & Lessons Learned

This attack chain demonstrates how minor misconfigurations compound into total system compromise:

1. Insecure File Sharing (NFS): Exposing internal support directories to the public network without IP whitelisting.

2. Sensitive Data Exposure: Pasting cleartext passwords and configuration files into unencrypted, permanently stored chat logs.

3. Insecure Permissions (SMB): Allowing standard users (alex) read access to sensitive developer shares (devshare), facilitating privilege escalation.

4. Implicit Database Trust: Relying solely on Windows Authentication for database access, meaning a compromise of the OS immediately results in a complete database breach.