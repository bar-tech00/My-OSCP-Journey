# RootMe CTF Walkthrough

In this walkthrough, I will demonstrate how I completed the RootMe challenge on TryHackMe.

Victim: 10.10.116.177


I began by scanning the target machine using `nmap`. I included the `-sV` switch to identify service versions, which helped me answer the first three questions in one go.

```
nmap -sC -sV 10.10.116.177
```

![1. First nmap scan](/images/TryHackMe/RootMe/1_nmap_first.png)

*Question*: Scan the machine, how many ports are open?

Answer: 2

Question: What version of Apache is running?

Answer: 2.4.29

Question: What service is running on port 22?

Answer: SSH


I discovered two running services on ports `22` which is SSH and `80` which is http server. Since SSH is usually not the first attack vector in CTF challenges, I focused on the web server. Upon visiting the website, I found a simple page with no immediately interesting content.

![2. Website](/images/TryHackMe/RootMe/2_website.png)

To discover hidden resources, I used `ffuf`, adding the `-e` switch to include potential `.php` files.

```
ffuf -w /usr/share/wordlists/dirb/big.txt -u http://10.10.116.177/FUZZ -e php
```

![3. Discovered directories](/images/TryHackMe/RootMe/3_discovered_directories.png)

The scan uncovered nine resources, though some returned `403 Forbidden` errors. The most interesting ones were:

*`/panel`
*`/uploads`


Question: What is the hidden directory?

Answer: /panel/


Navigating to `/panel/`, I found an upload page.

![4. Panel directory](/images/TryHackMe/RootMe/4_panel_directory.png)

Additionally, the `/uploads/` directory listed files uploaded to the web server.

![5. Uploads directory](/images/TryHackMe/RootMe/5_uploads_directory.png)


To gain a foothold, I used a PHP reverse shell from [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), saving it as `shell.php`. After modifying it to connect back to my machine on port `4444`, I attempted an upload. However, a restriction prevented `.php` files from being uploaded.

![6. Extension php not allowed](/images/TryHackMe/RootMe/6_php_extension_not_allowed.png)


To bypass this filter, I tried different extensions for PHP files. The extension `.php5` successfully bypassed the restriction, allowing me to upload `shell.php5`.

![7. Extension php5 allowed](/images/TryHackMe/RootMe/7_php5_extension_allowed.png)


I then set up a netcat listener on my machine:.

```
nc -nlvp 4444
```

Next, I accessed the `/uploads/` directory and executed `shell.php5` by clicking on it. This triggered the reverse shell, giving me access to the target machine as the `www-data` user.

![8. Revere shell](/images/TryHackMe/RootMe/8_reverse_shell.png)


With shell access, I used the `find` command to locate the `user.txt` flag:

```
find / -name “user.txt” 2>/dev/null
```

The flag was located in `/var/www/user.txt`.

![9. User flag](/images/TryHackMe/RootMe/9_user_flag.png)


Question: user.txt

Answer: THM{y0u_g0t_a_sh3ll}


To escalate privileges, I searched for binaries with the SUID bit set:

```
find / -type f -perm -04000 -ls 2>/dev/null
```

Among the results, `/usr/bin/python` stood out as a potential privilege escalation vector. Using[GTFObins](https://gtfobins.github.io/), I leveraged it to escalate to root:

```
/usr/bin/python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

![10. Root privilege](/images/TryHackMe/RootMe/10_root_privilege.png)


Once root access was obtained, I searched for `root.txt`:

```
find / -name “root.txt”
```

The flag was found in `/root/root.txt`, which I read using `cat`.

![11. Root flag](/images/TryHackMe/RootMe/11_root_flag.png)


Question: root.txt

Answer: THM{pr1v1l3g3_3sc4l4t10n}
