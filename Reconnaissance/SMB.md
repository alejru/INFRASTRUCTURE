# SMB Reconnaissance
## NMAP
#### Default Ports
```
#nmap 10.0.0.1
Starting Nmap 7.70 ( https://nmap.org ) at 2020-12-07 12:45 UTC
Nmap scan report for target-1 (192.176.83.3)
Host is up (0.0000080s latency).
Not shown: 998 closed ports
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```
```
# nmap -sU --top-ports 25 10.0.0.1
Starting Nmap 7.70 ( https://nmap.org ) at 2020-12-07 12:47 UTC
Nmap scan report for target-1 (10.0.0.1)
Host is up (0.000069s latency).
PORT      STATE         SERVICE
137/udp   open          netbios-ns
138/udp   open|filtered netbios-dgm
```

#### Workgroup
```
# nmap -sV -p 445 10.0.0.1
Starting Nmap 7.70 ( https://nmap.org ) at 2020-12-07 12:50 UTC
Nmap scan report for target-1 (10.0.0.1)
Host is up (0.000043s latency).

PORT    STATE SERVICE     VERSION
445/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: MYWORKGROUP)
MAC Address: 02:42:C0:B0:53:03 (Unknown)
Service Info: Host: HOST
```

#### SMB Version, NetBios computer name 
```
# nmap --script smb-os-discovery.nse -p445 10.0.0.1
Starting Nmap 7.70 ( https://nmap.org ) at 2020-12-07 12:53 UTC
Nmap scan report for target-1 (10.0.0.1)
Host is up (0.000025s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds
MAC Address: 02:42:C0:B0:53:03 (Unknown)

Host script results:
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: host
|   NetBIOS computer name: HOST\x00
|   Domain name: \x00
|   FQDN: host
|_  System time: 2020-12-07T12:53:42+00:00
```
#### Test NT LM 0.12 (SMBv1) dialects
```
# nmap -p445 --script smb-protocols 10.0.0.1
PORT    STATE SERVICE
445/tcp open  microsoft-ds
Host script results:
| smb-protocols: 
|   dialects: 
|     NT LM 0.12 (SMBv1) [dangerous, but default]
|     2.02
|     2.10
|     3.00
|     3.02
|_    3.11
```
#### Find SMB Users
```
 # nmap -p 445 --script smb-enum-users 10.0.0.1
PORT    STATE SERVICE
445/tcp open  microsoft-ds
Host script results:
| smb-enum-users: 
|   SAMBA-RECON\admin (RID: 1005)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\carlos (RID: 1004)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\idiot (RID: 1002)
|     Full name:   
|     Description: 
|     Flags:       Normal user account
|   SAMBA-RECON\marlejo (RID: 1003)
|     Full name:   
|     Description: 
|_    Flags:       Normal user account
```
#### Find Shares
The most interesting is to find shares with READ/WRITE access
```
# nmap --script smb-enum-shares.nse -p445 10.0.0.1
PORT    STATE SERVICE
445/tcp open  microsoft-ds
Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.0.0.1\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (samba.recon.lab)
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.0.0.1\carlos: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\samba\carlos
|     Anonymous access: <none>
|     Current user access: <none>
```




## METASPLOIT
#### SMB Version 
```
msf5 auxiliary(scanner/smb/smb_version) > show info
Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  RHOSTS     10.0.0.1         yes       The target address range or CIDR identifier
  SMBDomain  .                no        The Windows domain to use for authentication
  SMBPass                     no        The password for the specified username
  SMBUser                     no        The username to authenticate as
  THREADS    1                yes       The number of concurrent threads
msf5 auxiliary(scanner/smb/smb_version) > run

[*] 10.0.0.1:445      - Host could not be identified: Windows 6.1 (Samba 4.3.11-Ubuntu)
[*] 10.0.0.1:445      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
#### Test support of SMBv2
```
msf5 auxiliary(scanner/smb/smb2) > show info
Basic options:
  Name     Current Setting  Required  Description
  ----     ---------------  --------  -----------
  RHOSTS   10.0.0.1         yes       The target address range or CIDR identifier
  RPORT    445              yes       The target port (TCP)
  THREADS  1                yes       The number of concurrent threads
msf5 auxiliary(scanner/smb/smb2) > exploit
[+] 10.0.0.1:445     - 10.0.0.1  supports SMB 2 [dialect 255.2] and has been online for 3681089 hours
[*] 10.0.0.1 :445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
#### Find SMB Users
SAM Users
```
msf5 auxiliary(scanner/smb/smb_enumusers) > show info 
Basic options:
  Name       Current Setting  Required  Description
  ----       ---------------  --------  -----------
  RHOSTS     10.0.0.1    yes       The target address range or CIDR identifier
  SMBDomain  .                no        The Windows domain to use for authentication
  SMBPass                     no        The password for the specified username
  SMBUser                     no        The username to authenticate as
  THREADS    1                yes       The number of concurrent threads
msf5 auxiliary(scanner/smb/smb_enumusers) > exploit

[+] 10.0.0.1:139     - SAMBA-RECON [ carlos, admin, jujui ] ( LockoutTries=0 PasswordMin=5 )
[*] 10.0.0.1:        - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
#### Find Shares
`msf5 auxiliary(scanner/smb/smb_enumshares)`
#### Share Brute Force
```
use auxiliary/scanner/smb/smb_login
set PASS_FILE /usr/share/wordlists/metasploit/unix_passwords.txt
set SMBUser carlos
set RHOSTS 10.0.0.1
exploit
```
#### List the Pipes Available (Authenticated)
`msf5> use auxiliary/scanner/smb/pipe_auditor`






## SMBCLIENT
#### Test Null Session Find shares
`#smbclient -L 10.0.0.1 -N` null session is allowed if shares are displayed without password. (Server Description)
#### Explore a share file
```
# smbclient //10.0.0.1/public -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Dec  9 17:56:32 2020
  ..                                  D        0  Tue Nov 27 13:36:13 2018
  file1                               D        0  Tue Nov 27 13:36:13 2018
  file2                               D        0  Tue Nov 27 13:36:13 2018
```
#### Access a Share (Authenticated)
```
# smbclient //10.0.0.1/admin -U admin
Enter WORKGROUP\admin's password: 
Try "help" to get a list of possible commands.
smb: \> ls
```








## RPCCLIENT
#### Test Null Session 
Null session is allowed if no errors while connecting without credentials 
```
# rpcclient -U "" -N 10.0.0.1   
rpcclient $> 
```
#### Server Information
```
# rpcclient -U "" -N 10.0.0.1   
rpcclient $> srvinfo
        HOST    Wk Sv PrQ Unx NT SNT host.local
        platform_id     :       500
        os version      :       6.1
        server type     :       0x809a03
```
#### Test SMB Users
```
#  rpcclient -U "" -N 10.0.0.1
rpcclient $> enumdomusers
user:[carlos] rid:[0x3e8]
user:[julio] rid:[0x3ea]
user:[admin] rid:[0x3ed]
```
#### Find SID of a user
```
#  rpcclient -U "" -N 10.0.0.1
rpcclient $> lookupnames admin
admin S-1-5-21-4056189605-2085045094-1961111545-1005 (User: 1)
```
#### List Domain Groups
```
root@attackdefense:~# rpcclient -U "" -N 10.0.0.1
rpcclient $> enumdomgroups
```





## ENUM4LINUX
#### OS Version
`# enum4linux -o 10.0.0.1`
#### SMB USers
```
# enum4linux -U 10.0.0.1
 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.0.0.1
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, none
```
#### Find Shares
`# enum4linux -S 10.0.0.1`
#### Domain Groups
`# enum4linux -G 10.0.0.1`
#### Printer Info
`# enum4linux -i 10.0.0.1`
#### Enumerate Linux Users by  RID Cycling (Authenticated)
``







## NMBLOOKUP
#### SMB Version, NetBios computer name
`# nmblookup -A 10.0.0.1`





## HYDRA
#### Share Brute Force
`# hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.0.0.1 smb`




## SMBMAP
#### Explore Shares (Authenticated)
```
# smbmap -H 10.0.0.1 -u admin -p password1
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.0.0.1...
[+] IP: 10.0.0.1:445       Name: target-1                                          
        Disk                                                    Permissions
        ----                                                    -----------
        carlos                                                  READ, WRITE
        public                                                  READ ONLY
        admin                                                   READ, WRITE
        IPC$                                                    NO ACCESS
```
