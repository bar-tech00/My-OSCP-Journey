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






<details>
  <summary>User flag (CLICK TO SEE)</summary>
THM{USER_FLAG_1231_EXPOSE}
</details>



<details>
  <summary>Root flag (CLICK TO SEE)</summary>
THM{ROOT_EXPOSED_1001}
</details>
