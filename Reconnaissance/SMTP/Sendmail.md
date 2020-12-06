# SMTP SendMail Reconnaissance
## Get banner, server name and domain name
* `#nc 10.0.0.1 25`
* `#telnet 10.0.0.1 25`
* `nmap -sV -script banner -p- 10.0.0.1`

## Identify Supported Commands
```
help
214-2.0.0 This is sendmail version 8.15.2
214-2.0.0 Topics:
214-2.0.0       HELO    EHLO    MAIL    RCPT    DATA
214-2.0.0       RSET    NOOP    QUIT    HELP    VRFY
214-2.0.0       EXPN    VERB    ETRN    DSN     AUTH
214-2.0.0       STARTTLS
```

## Manual User Enumeration
The enumeration can be done with EXPN, VRFY or RCPT
```
HELO x
250  pleased to meet you
EXPN user
550 5.1.1 user... User unknown
EXPN admin
250 2.1.5 admin <admin@domain>
```

## User Enumearion ([smtp-user-enum](https://tools.kali.org/information-gathering/smtp-user-enum))
`#smtp-user-enum -M RCPT -U users.txt -t 10.0.0.1`

## User Enumeration ([Metasploit](https://www.rapid7.com/db/modules/auxiliary/scanner/smtp/smtp_enum))
```
msf5 auxiliary(scanner/smtp/smtp_enum) > show info
Basic options:
  Name       Current Setting                                                Required  Description
  ----       ---------------                                                --------  -----------
  RHOSTS     10.0.0.1                                                       yes       The target address range or CIDR identifier
  RPORT      25                                                             yes       The target port (TCP)
  THREADS    1                                                              yes       The number of concurrent threads
  UNIXONLY   true                                                           yes       Skip Microsoft bannered servers when testing unix users
  USER_FILE  /usr/share/metasploit-framework/data/wordlists/unix_users.txt  yes       The file that contains a list of probable users accounts.
```

## Mail Relay (Manual)
```
root@attackdefense:~# nc 192.77.76.3 25
220 domain-x ESMTP Sendmail 8.15.2/8.15.2/Debian-10; Sun, 6 Dec 2020 18:00:25 GMT; 
helo x
250 domain-x Hello, pleased to meet you
mail from: admin@attacker.com
250 2.1.0 admin@attacker.com... Sender ok
rcpt to: root@domain-x
250 2.1.5 root@domain-x... Recipient ok (will queue)
data
354 Enter mail, end with "." on a line by itself
Subject: hi

Hello, test

.
250 2.0.0 0B6I0Pe9000330 Message accepted for delivery
```

## Mail Relay ([sendemail](http://www.postfix.org/sendmail.1.html))
```
sendemail -f admin@attacker.com -t root@domain-x -s 10.0.0.1 -u Fakemail -m "Hi test" -o tls=no
Email was sent successfully!!!
```

