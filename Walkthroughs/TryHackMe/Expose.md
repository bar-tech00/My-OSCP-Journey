# Expose

In this note I will present how I have completed Expose room on TryHackMe.

Victim: 10.10.195.148


Firstly, I started by scanning the host with nmap. Right away I scanned every port on the victim machine by using -p- option which will give me overall information about services running on the victim machine across all ports. If this option would’t be used then nmap would scan the top 1000 TCP ports.
```
nmap -p- 10.10.195.148
```
`-p-` - will scan all TCP ports.

![1. First nmap scan](/images/TryHackMe/Expose/1_nmap_first.png)

I have discovered 5 opened ports. I will focus on them by specifying them in my next nmap command. Also, I will add two other options `-sV` and `-sC`.
```
nmap -p 21,22,53,1337,1883 -sV -sC 10.10.195.148
```

`-p <port1>,<port2>,<port n>` - will specify which port or ports we want to specify durning the scan.

`-sV` - Probe open ports to determine service/version info.

`-sC` - Performs a script scan using the default set of scripts. It is equivalent to `--script=default`. Some of the scripts in this category are considered intrusive and should not be run against a target network without permission.

![2. Second nmap scan](/images/TryHackMe/Expose/2_nmap_more_details.png)

I got some additional information on previosly discovered services running on a victims machine. I have played around with FTP on port 21 for a while but this led me to nowhere. Instead I have focused on port 1337. As seen in nmap scan this is http server with page title “EXPOSED”. I opened this webpage by typing IP of a victim host followed by port on which webpage is hosted on in address bar. I used Mozzila FireFox browser available on AttackBox.

```http://10.10.195.148:1337```

![3. Main expose website](/images/TryHackMe/Expose/3_expose_main_website.png)

It is empty. I checked source code of a webpage.

![4. Main expose site source code](/images/TryHackMe/Expose/4_expose_main_site_source_code.png)

But no additional info to be found there.

There could be some additional resources like directories and files hosted there. There is no link to them but still they could be there. I need to try to find them. Great tool to search for other directories on a webpage is gobuster. I used below command for this task.

```
gobuster dir -u http://10.10.195.148:1337 -w /usr/share/wordlists/dirb/common.txt
```

`dir` - Uses directory/file enumeration mode, typical option for enumerating other resources on a webpage.

`-u` - specifies what domain should be enumerated. I added port 1337 as this domain is hosted there. If it would be added then attack would not be conducted.

`-w` - specifies a worldlist that will be used during enumeration. I used the one available on AttackBox called common.txt because, as it says in it’s name, there are common names.

![5. gobuster common](/images/TryHackMe/Expose/5_gobuster_common.png)

I have discovered additional resources which can be accessible. I have focused on two of those:

* `/admin`
* `/phpmyadmin`

Firstly, `/phpmyadmin` was a login page to database. I tried some bruteforcing but that led my nowhere.

![6. PhpMyAdmin login page](/images/TryHackMe/Expose/6_phpmyadmin_login_page.png)

I decided to switch my focus onto second one of interest - `/admin`.

![7. Admin login page](/images/TryHackMe/Expose/7_admin_login_page.png)

The “continue” button on a page was not clickable. Additionally, note “Is this the right admin portal?” was a hint to look for something else.
I decided to go back a little and conduct another enumeration on this website. I used previous command but changed it just a little bit. Instead of using wordlist “common.txt” which is, so to speak, medium size, I used “big.txt” form the same location on AttackBox. It is bigger then “common.txt”

```
gobuster dir -u http://10.10.195.148:1337 -w /usr/share/wordlists/dirb/big.txt
```

I won’t explain what each of it means as it was explained above in great detail.

![8. gobuster big](/images/TryHackMe/Expose/8_gobuster_big.png)

As seen above, I have discovered another directory called `/admin_101` which was not found by using smaller dictionary. I entered this location on a webserver and this was a right step towards rooting this machine.

```
http://10.10.195.148:1337/admin_101
```

![9. admin_101 login page](/images/TryHackMe/Expose/9_admin_101_login_page.png)

In the login form an email was provided: hacker@root.thm. This time the button “Continue” was clickable. I tried using some common passwords but this was not it. I decided to peep on the request that is being sent to the server and its response.

![10. admin_101 response query](/images/TryHackMe/Expose/10_admin_101_response_query.png)

In the response a SQL query can be seen. I tired to use some bruteforcing using BurpSuite (can be also achived by. ex. hydra) but no success there. I decided on using sqlmap. I tried using sqlmap in different ways, but the one below worked. Firstly, I saved HTTP request which would be send to the server in a file I called `req`. The request I'm talking about is on the left side of the upper image.


Then I used sqlmap command seen below. It asked me couple times to choose between different options (yes/no) and after attack is completed I have received below output.

```
sqlmap -r req -dump
```

`-r` - the “-r” option tells sqlmap the file containing the HTTP request which contains the injection point.

`-dump` - is used to "dump" (or more soficitaded word: exfiltrate) whole database from victims server.

![11 sqlmap output](/images/TryHackMe/Expose/11_sqlmap_output.png)

By doing this password for hacker@root.thm was discovered which is VeryDifficultPassword!!#@#@!#!@#1231.

Also, there was some information on other URI’s most probably also accessible on our webserver.

| url  | password |
| ------------- | ------------- |
| /file1010111/index.php  | 69c66901194a6486176e81f5945b8929 (easytohack)  |
| /upload-cv00101011/index.php  | // ONLY ACCESSIBLE THROUGH USERNAME STARTING WITH Z  |

Firstly, I opened site `http://10.10.195.148:1337/file1010111/index.php` 

![12 file1010111 login page](/images/TryHackMe/Expose/12_file1010111__login_page.png)

And on login page there I used provided password from sqlmap output for this login page. I wrote them down in table above.

![13 file1010111 successful login](/images/TryHackMe/Expose/13_file1010111__successful_login.png)

Great, I got access. Not much on this website from the first look, but text `Parameter Fuzzing is also important :)  or Can you hide DOM elements?` gave me a little hint. I also checked source code of this website. This gave me another clue: `Hint: Try file or view as GET parameters?`. Firstly, I forgot what were GET parameters mentioned here. By searching the web I recalled that those GET parameters were the ones like on the example below:

```
https://<IP>:<port>/adminloginpage/index.php?username=<someusername>&password=<somepassword>
```
This is an example as no website should send credentials in clear text.

So… Here I was provided with names of this parameters: `file` or `view`. Such parameters could be exploited by path traversal vulnerability. This was also the case here. I just need to create a good attack. I decided to go for `/etc/passwd` file.

```
http://10.10.195.148:1337/file1010111/index.php?file=../../../../../etc/passwd
```
I was asked to provide `easytohack` password again.

Fortunately there was no additional layer of security which would santize input. This got me content of a very important file on each computer. This will come in handy in the next step.

![14 file1010111 path traversal](/images/TryHackMe/Expose/14_file1010111_path_traversal.png)

Previously, I discovered another accessible site under `/upload-cv00101011/index.php`

```
http://10.10.195.148:1337/upload-cv00101011/index.php
```

![15. upload_cv00101011 login page](/images/TryHackMe/Expose/15_upload_cv00101011_login_page.png)

There is information that the password is the name of the machine user starting with the letter “z”. Previously, I got the content of /etc/passwd file where all usernames are. I run through the /etc/passwd file I disovered before and I found the user “zeamkish”. I tried typing it into form and this got us access to the portal.

![16. upload_cv00101011 successful login](/images/TryHackMe/Expose/16_upload_cv00101011_successfull_login.png)


There is a place to upload files. Only .png and .jpg can be uploaded. I tried uploading some .png file. I captured a request and response with Burp. As seen below I received a response with text “File upload successfully! Maybe look in source code to see the path” and “in /upload_thm_1001 folder” so I checked this folder. 

![17. upload_thm_1001 folder burp request capture](/images/TryHackMe/Expose/17_upload_thm_1001_burp_request_capture.png)

As said in response after uploading a file I checked folder `/upload-cv00101011/upload_thm_1001/`. I can see exacly that my file was uploded here.

```http://10.10.195.148:1337/upload-cv00101011/upload_thm_1001/```

![18. upload_thm_1001 folder with my file](/images/TryHackMe/Expose/18_upload_thm_1001_folder_with_my_file.png)


If I could put a file with code which when executed will give me a shell then I would be very happy. I used php reverse shell code form [PentestMonkey](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and saved on my machine in `file.png`. I changed IP and port to which my shell should connect to to my IP and port 4444. I set up Burp to capture request which should be send to the server when uploading a file. Then I uploaded my `shell.png` onto the server on upload portal. In a reqest before it was send I changed a file format of my file from `.png` to `.php`. After change I sent request through.

![19. uploading shell and changing format with burp](/images/TryHackMe/Expose/19_uploading_shell_format_change_with_burp)

Then I entered `/upload-cv00101011/upload_thm_1001/` where I could see my shell. Firstly, I set up listener on my host before I execute my shellcode so the connection could be made.

```
nc -nlvp 4444
```

![20. shell on webserver](/images/TryHackMe/Expose/20_shell_on_webserver.png)

When I clicked on file containing my reverse shell it was executed on the side of the webserver and it made a connection back to my machine which got me a shell. I have looked around it a little bit and found two very important files in `/home/zeamkish/` directory. Account on which I currently was did not have perimission to view `flag.txt` file but I could see `ssh_creds.txt`.

![21. ssh creds on victims machine](/images/TryHackMe/Expose/21_ssh_creds.png)

Username: zeamkish

Password: easytohack@123

I connected to the victim machine by SSH by using the below command then typing password in a proper filed when asked.

```
ssh zeamkish@10.10.195.148
```

![22. ssh login](/images/TryHackMe/Expose/22_ssh_login.png)

And I got ssh connection to a victim host on user account zeamkish. This allowed me to see contents of the file `flag.txt` in his user space `/home/zeamkish`.

User flag:

THM{ROOT_EXPOSED_1001}

![23. user flag](/images/TryHackMe/Expose/23_user_flag.png)

Great. Now we need to escalate priviledeg to root to achieve the root flag. I poked around trying different metchods but the one correct here is a very simple one - SUID. Actually, it was the first method I checked but I did not read carefully enough what binaries had SUID set so I could exploit them. This oversight made me lose some time. Lesson for everybody - ready carefully, don’t just run through things. If I would spend 30 seconds more than I could save around 45 minutes. To check which files have SUID I used the command seen below.

```
find / -type f -perm -04000 -ls 2>/dev/null
```

![24. SUID binaries](/images/TryHackMe/Expose/24_suid_binaries.png)

The one that look out of place is `/usr/bin/find` binary. It can be used to escalate privilege. I used [GTFObins](https://gtfobins.github.io/) to achieve that. There are many instruction how to use different binaries that could be on victim machine which could be used to escalate privileges. Back to `/usr/bin/find`. By using below command:

```
/usr/bin/find . -exec /bin/sh -p \; -quit
```

I achieved root privilege. Then I went straight for the flag in `/root` directory.


Root flag:

THM{ROOT_EXPOSED_1001}

![25. root flag](/images/TryHackMe/Expose/25_root_flag.png)
