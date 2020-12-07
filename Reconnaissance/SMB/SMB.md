# SAMBA Reconnaissance
## Default port
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

## Workgroup
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

## Samba Version, NetBios computer name (nmap)
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

## Samba Version (Metasploit)
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

## Samba Version, NetBios computer name (nmblookup)
* `#nmblookup -A 192.176.83.3`

## Test Null Session (smbclient)
*`#smbclient -L 10.0.0.1 -N` null session is allowed if shares are displayed without password

## Test Null Session (rpcclient)
Null session is allowed if no errors while connecting without credentials 
```
root@attackdefense:~# rpcclient -U "" -N 10.0.0.1   
rpcclient $> 
```


