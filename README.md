<h1 aligner="center"> eJPT guide 2022 (eLearnSecurity Junior Penetration Tester </h1>

---

<p> <strong> Author: Rodolfo "m4ll0ck3r" Tavares </strong>  <p>

<h3> To be honest, everything you need to pass the eJPT is in that free course at https://ine.com, but if you want more content and <strong> FREE </strong> enjoy !</h3>

* [Wireshak essentials] https://www.hackingarticles.in/wireshark-for-pentesters-a-beginners-guide/
* [Burp suite proxy basics] https://github.com/Ignitetechnologies/BurpSuite-For-Pentester
* [Linux Routing guide] - https://opensource.com/business/16/8/introduction-linux-network-routing
* [Session Cookies] https://www.hackingarticles.in/beginner-guide-understand-cookies-session-management/
* [XSS Guide] https://www.hackingarticles.in/comprehensive-guide-on-cross-site-scripting-xss/ and
https://www.hackingarticles.in/cross-site-scripting-exploitation/
* [SQL injection guide] - https://www.hackingarticles.in/beginner-guide-sql-injection-part-1/
https://www.hackingarticles.in/beginner-guide-sql-injection-boolean-based-part-2/
https://www.hackingarticles.in/manual-sql-injection-exploitation-step-step/
https://www.hackingarticles.in/form-based-sql-injection-manually/
https://www.hackingarticles.in/exploiting-form-based-sql-injection-using-sqlmap/
https://www.hackingarticles.in/bypass-filter-sql-injection-manually/
https://www.hackingarticles.in/exploiting-sql-injection-nmap-sqlmap/
* [Automation SQL Attacks] - https://www.hackingarticles.in/exploiting-sql-injection-nmap-sqlmap/ and
https://www.hackingarticles.in/comprehensive-guide-to-sqlmap-target-options15249-2
* [John the Ripper] - https://www.hackingarticles.in/beginner-guide-john-the-ripper-part-1/ and
https://www.hackingarticles.in/beginners-guide-for-john-the-ripper-part-2/
* [Metasploit] - https://www.youtube.com/watch?v=8lR27r8Y_ik&list=PLBf0hzazHTGN31ZPTzBbk70bohTYT7HSm

---

<h1> Talking about the test itself </h1>

First, pay attention to the file that will be sent with the .pcap and wordlists to use in the environment. Then, when connecting to the exam VPN, test if it is working correctly, and then check the routes used, more information can be seen in the links above. But the ip route add command is more than enough

* ip route
```bash
Syntax
ip route add <Network-range> via <router-IP> dev <interface>
eg.
ip route add 10.10.10.0/24 via 10.10.11.1 dev tap0

```

At least for me. "It's worth remembering here, that this is my journey, so there are probably other ways to do it". After that, focus and footprint the network with:

* NMAP
```bash
nmap -sn 10.10.10.0/24
nmap -sV -p- -iL targets -oN nmap.initial -v
nmap -A -p- -iL targets -oN nmap.aggressive -v
nmap -p --script=vuln -v
```

* fping
```bash
fping -a -g 10.10.10.0/24 2>/dev/null > targets

```

After enumerating the network assets, check the service versions for possible exploits. This can even be done with nmap via the --script argument. Examples:

```
-> SMB
nmap --script=smb-* -iL hosts.nmap
-> FTP
nmap --script=ftp-* -iL hosts.nmap
-> SSH
nmap --script=ssh-* -iL hosts.nmap
```

Probably from here you have already enumerated everything, and you can already exploit, the services with metasploit:

* Metasploit
```
metasploit -q

search ftp
```

At this point, you can answer at least 6 questions. When you find the web application, use burp to navigate through each application's functionality, and look for SQL injection and XSS, in addition to being aware of questions regarding the site itself. You can explore almost all services manually, but if you want to automate it, you can use SQLMAP.


```

sqlmap -r Post.req
sqlmap -u "http://10.10.10.10/file.php?id=1" -p id
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin"

Get database if injection Exists
sqlmap -r login.req --dbs
sqlmap -u "http://10.10.10.10/file.php?id=1" -p id --dbs
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin" --dbs\

Get Tables in a Database
sqlmap -r login.req -D dbname --tables
sqlmap -u "http://10.10.10.10/file.php?id=1" -p id -D dbname --tables
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin" -D dbname --tables

Get data in a Database tables
sqlmap -r login.req -D dbname -T table_name --dump
sqlmap -u "http://10.10.10.10/file.php?id=1" -p id -D dbname -T table_name --dump
sqlmap -u "http://10.10.10.10/login.php" --data="user=admin&password=admin" -D dbname -T table_name --dump

```

If you already know about crackmapexec, you can use it to avoid having to use smblclient.

* SMB 
```
smbclient -L //10.10.10.10/
enum4linux -U -M -S -P -G 10.10.10.10
nmap --script=smb-enum-users,smb-os-discovery,smb-enum-shares,smb-enum-groups,smb-enum-domains 10.10.10.10 -p 135,139,445 -v
nmap -p445 --script=smb-vuln-* 10.10.10.10 -v

Access Share
smbclient //10.10.10.10/share_name
```

* Crackmapexec
```
crackmapexec smb ms.evilcorp.org
crackmapexec smb 192.168.1.0 192.168.0.2
crackmapexec smb 192.168.1.0-28 10.0.0.1-67
crackmapexec smb 192.168.1.0/24
crackmapexec smb targets.txt



# Enumerate users
crackmapexec smb 192.168.215.104 -u 'user' -p 'PASS' --users

# Perform RID Bruteforce to get users
crackmapexec smb 192.168.215.104 -u 'user' -p 'PASS' --rid-brute

# Enumerate domain groups
crackmapexec smb 192.168.215.104 -u 'user' -p 'PASS' --groups

# Enumerate local users
crackmapexec smb 192.168.215.104 -u 'user' -p 'PASS' --local-users

# Enumerate shares
crackmapexec smb 192.168.215.104 -u 'user' -p 'PASS' --shares


```


In case you need to use john the ripper for you to crack hash or something like that, follow the useful tutorial.

* https://www.cyberciti.biz/faq/unix-linux-password-cracking-john-the-ripper/



If you can get a shell in a windows environment it might help:


* Windows cmd helper

```

To search for a file starting from current directory
dir /b/s "*.conf*"
dir /b/s "*.txt*"
dir /b/s "*filename*"

Check routing table
route print
netstat -r

Check Users
net users

List drives on the machine
wmic logicaldisk get Caption,Description,providername

```
