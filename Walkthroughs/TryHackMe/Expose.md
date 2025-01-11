# Expose

In this note I will present how I have completed Expose room on TryHackMe.

AttackBox: <IP>

Victim: <IP>

Firstly, I started by scanning the host with nmap. Right away I scanned every port on the victim machine by using -p- option which will give me overall information about services running on the victim machine across all ports. If this option would’t be used then nmap would scan the top 1000 TCP ports.
```
nmap -p- 10.10.113.128
```
`-p-` - will scan all TCP ports

### SSC1 first nmap

I have discovered 5 opened ports. I will focus on them by specifying them in my next nmap command. Also, I will add two other options -sV and -sC.
```
nmap -p 21,22,53,1337,1883 -sV -sC 10.10.113.128
```

`-p` - \<port1\>,\<port2\>,\<port n\> - will specify which port or ports we want to specify durning the scan

`-sV` - Probe open ports to determine service/version info

`-sC` - Performs a script scan using the default set of scripts. It is equivalent to --script=default. Some of the scripts in this category are considered intrusive and should not be run against a target network without permission.

### SSC2 nmap better

I got some additional information on previosly discovered services running on a victims machine. I have played around with FTP on port 21 for a while but this led me to nowhere. Instead I have focused on port 1337. As seen in nmap scan this is http server with page title “EXPOSED”. I opened this webpage by typing IP of a victim host followed by port on which webpage is hosted on in address bar. I used Mozzila FireFox browser available on AttackBox.

```http://<IP>:1337```

### SSC3 "exposed webpage"

It is empty. I checked source code of a webpage.

### SSC4 source code 

But no additional info to be found there.

There could be some additional resources like directories and files hosted there. There is no link to them but still they could be there. I need to try to find them. Great tool to search for other directories on a webpage is gobuster. I used below command for this task.

```
gobuster dir -u http://10.10.113.126:1337 -w /usr/share/wordlists/dirb/common.txt
```

`dir` - Uses directory/file enumeration mode, typical option for enumerating other resources on a webpage

`-u` - specifies what domain should be enumerated. I added port 1337 as this domain is hosted there. If it would be added then attack would not be conducted.

`-w` - specifies a worldlist that will be used during enumeration. I used the one available on AttackBox called common.txt because, as it says in it’s name, there are common names.

### SSC5 gobuster common.txt

I have discovered additional resources which can be accessible. I have focused on two of those:

`/admin`

`/phpmyadmin`

Firstly, /phpmyadmin was a login page to database. I tried some bruteforcing but that led my nowhere.

### SSC6 phpmyadmin login page

I decided to switch my focus onto second one of interest - `/admin`

### SSC6 /admin login page

The “continue” button on a page was not clickable. Additionally, note “Is this the right admin portal?” was a hint to look for something else.
I decided to go back a little and conduct another enumeration on this website. I used previous command but changed it just a little bit. Instead of using wordlist “common.txt” which is, so to speak, medium size, I used “big.txt” form the same location on AttackBox. It is bigger then “common.txt”

```
gobuster dir -u http://10.10.113.126:1337 -w /usr/share/wordlists/dirb/big.txt
```

I won’t explain what each of it means as it was explained above in great detail.

### SSC7 gobuster big.txt

As seen above, I have discovered another directory called `/admin_101` which was not found by using smaller dictionary. I entered this location on a webserver and this was a right step towards rooting this machine.

```
http://<IP>:1337/admin_101
```

### SSC8 /admin_101 login portal

In the login form an email was provided: hacker@root.thm. This time the button “Continue” was clickable. I tried using some common passwords but this was not it. I decided to peep on the request that is being sent to the server and its response.

In the response a SQL query can be seen. I tired to use some bruteforcing using BurpSuite (can be also achived by. ex. hydra) but no success there. I decided on using sqlmap. I tried using sqlmap in different ways, but the one below worked. Firstly, I saved HTTP request which would be send to the server in a file I called `req`.

### SSC9 req file HTTP request

Then I used sqlmap command seen below.

```
sqlmap -r req -dump
```

`-r` -
`-dump` - 

### SSC10 sqlmap output

By doing this password for hacker@root.thm was discovered which is VeryDifficultPassword!!#@#@!#!@#1231.

Also, there was some information on other URI’s most probably also accessible on our webserver.
/file1010111/index.php       | 1    | 69c66901194a6486176e81f5945b8929 (easytohack)

/upload-cv00101011/index.php | 3    | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z

| url  | password |
| ------------- | ------------- |
| /file1010111/index.php  | 69c66901194a6486176e81f5945b8929 (easytohack)  |
| /upload-cv00101011/index.php  | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z
  |


<details>
  <summary>User flag (CLICK TO SEE)</summary>
THM{USER_FLAG_1231_EXPOSE}
</details>



<details>
  <summary>Root flag (CLICK TO SEE)</summary>
THM{ROOT_EXPOSED_1001}
</details>
