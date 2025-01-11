# Expose

In this note I will present how I have completed Expose room on TryHackMe.

Firstly, I started by scanning the host with nmap. Right away I scanned every port on the victim machine by using -p- option which will give me overall information about services running on the victim machine across all ports. If this option would’t be used then nmap would scan the top 1000 TCP ports.
```
nmap -p- 10.10.113.128
```


<details>
  <summary>User flag (CLICK TO SEE)</summary>
THM{USER_FLAG_1231_EXPOSE}
</details>



<details>
  <summary>Root flag (CLICK TO SEE)</summary>
THM{ROOT_EXPOSED_1001}
</details>
