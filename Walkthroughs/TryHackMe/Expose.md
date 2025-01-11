# Expose

In this note I will present how I have completed Expose room on TryHackMe.

Firstly, I started by scanning the host with nmap. Right away I scanned every port on the victim machine by using -p- option which will give me overall information about services running on the victim machine across all ports. If this option would’t be used then nmap would scan the top 1000 TCP ports.
```
nmap -p- 10.10.113.128
```
`-p-` - will scan all TCP ports

I have discovered 5 opened ports. I will focus on them by specifying them in my next nmap command. Also, I will add two other options -sV and -sC.
```
nmap -p 21,22,53,1337,1883 -sV -sC 10.10.113.128
```

`-p` - \<port1\>,\<port2\>,\<port n\> - will specify which port or ports we want to specify durning the scan

`-sV` - Probe open ports to determine service/version info

`-sC` - Performs a script scan using the default set of scripts. It is equivalent to --script=default. Some of the scripts in this category are considered intrusive and should not be run against a target network without permission.



<details>
  <summary>User flag (CLICK TO SEE)</summary>
THM{USER_FLAG_1231_EXPOSE}
</details>



<details>
  <summary>Root flag (CLICK TO SEE)</summary>
THM{ROOT_EXPOSED_1001}
</details>
