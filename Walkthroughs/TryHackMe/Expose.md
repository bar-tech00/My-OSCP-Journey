# Expose

In this note I will present how I have completed Expose room on TryHackMe.

AttackBox: 10.10.23.110

Victim: 10.10.78.204


Firstly, I started by scanning the host with nmap. Right away I scanned every port on the victim machine by using -p- option which will give me overall information about services running on the victim machine across all ports. If this option would’t be used then nmap would scan the top 1000 TCP ports.
```
nmap -p- 10.10.78.204
```
`-p-` - will scan all TCP ports

![First nmap scan](/images/TryHackMe/Expose/nmap_first.png)

I have discovered 5 opened ports. I will focus on them by specifying them in my next nmap command. Also, I will add two other options -sV and -sC.
```
nmap -p 21,22,53,1337,1883 -sV -sC 10.10.113.128
```

`-p <port1>,<port2>,<port n>` - will specify which port or ports we want to specify durning the scan

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

Firstly, I opened site `http://<IP>:1337//file1010111/index.php` 

### SSC11 file1010111/index.php login page

And on login page there I used provided password from sqlmap output. I wrote them down in table above.

### SSC12 file1010111/index.php after login 

Great, I got access. Not much on this website from the first look, but text `Parameter Fuzzing is also important :)  or Can you hide DOM elements?` gave me a little hint. I also checked source code of this website. This gave me another clue: `Hint: Try file or view as GET parameters?`. Firstly, I forgot what were GET parameters mentioned here. By searching the web I recalled that those GET parameters were the ones like on the example below:

```
https://<IP>:<port>/adminloginpage/index.php?username=<someusername>&password=<somepassword>

```
This is an example as no website should send credentials in clear text.

So… Here I was provided with names of this parameters: `file` or `view`. Such parameters could be exploited by path traversal vulnerability. This was also the case here. I just need to create a good attack. I decided to go for /etc/passwd file.

```
http://<IP>:1337//file1010111/index.php?file=../../../../../etc/passwd
```

Fortunately there was no additional layer of security which would santize input. This got me content of a very important file on each computer. This will come in handy in the next step.

### SSC13 path travelsal 

Previously, I discovered another accessible site under `/upload-cv00101011/index.php`

```
http://<IP>:1337//upload-cv00101011/index.php
```

### SSC14 upload-cv00101011/index.php logn page

There is information that the password is the name of the machine user starting with the letter “z”. Previously, I got the content of /etc/passwd file where all usernames are. I run through the /etc/passwd file and I found the user “zeamkish”. I tried typing it into form and this got us access to the portal.

### SSC15 upload-cv00101011/index.php succesful login 


There is a place to upload files. Only .png and .jpg can be uploaded. I tried uploading some .png file and received a response with text “File upload successfully! Maybe look in source code to see the path” and “in /upload_thm_1001 folder” so I checked this folder. 

### SSC16 upload-cv00101011/index.php successful upload 


If I could put a file with code which when executed will give me a shell then I would be very happy. I used php reverse shell code form PentestMonkey and saved on my machine. I changed IP and port to which my shell should connect to my IP and port 4444. Then I uploaded it onto the server on upload portal, then entered /upload_thm_1001 where I could execute my shell code by clicking on it. But firstly I set my listener on my host.

```
nc -nlvp 4444
```

When I clicked on my shell file it was executed on the side of webserver and it made a connection to my machine which got me a shell. I have looked around it a little bit and found two very important files. Account on which I currently was did not have perimission to view `flag.txt` file but I could see `ssh_creds.txt`.

### SSC17 ssh creds

Username: zeamkish

Password: easytohack@123

I connected to the victim machine by SSH by using the below command then typing password in a proper filed when asked.

```
ssh zeamkish@<IP>
```

### SSC18 SSH connection

And I got ssh connection to a victim host on user account zeamkish. This allowed me to see contents of the file `flag.txt` in his user space `/home/zeamkish`.

<details>
  <summary>User flag (CLICK TO SEE)</summary>
  ### SSC19 user flag
  
  THM{USER_FLAG_1231_EXPOSE}
</details>

Great. Now we need to escalate priviledeg to root to achieve the root flag. I poked around trying different metchods but the one correct here is a very simple one - SUID. Actually, it was the first method I checked but I did not read carefully enough what binaries had SUID set so I could exploit them. This oversight made me lose some time. Lesson for everybody - ready carefully, don’t just run through things. If I would spend 30 seconds more than I could save around 45 minutes. To check which files have SUID I used the command seen below.

```
find command for perm 4000
```

### SSC20 SUID binaries

The one that look out of place is /usr/bin/find binary. It can be used to escalate privilege. I used GTFObins to achieve that. There are many instruction how to use different binaries that could be on victim machine which could be used to escalate privileges. Back to /usr/bin/find. By using below command:

```
./find . -exec /bin/sh -p \; -quit
```

I achieved root privilege. Then I went straight for the flag in /root directory.


<details>
  <summary>Root flag (CLICK TO SEE)</summary>
  ### SSC21 root flag
  THM{ROOT_EXPOSED_1001}
</details>
