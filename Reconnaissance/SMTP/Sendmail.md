# SMTP SendMail Reconnaissance
## Get banner, server name and domain name
* `#nc 10.0.0.1 25`
* `#telnet 10.0.0.1 25`

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

## User Enumeration (Metasploit)

## Mail Relay (Manual)

## Mail Relay (sendemail)

