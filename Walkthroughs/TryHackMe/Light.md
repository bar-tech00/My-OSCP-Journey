# Light CTF Walkthrough

This walkthrough will guide you through solving the TryHackMe room called Light. The room focuses on exploiting SQL injection vulnerabilities and learning how to extract sensitive information from a database. Below, I provide the steps I took, insights gained, and the queries I used to solve the challenges in this room.

Firstly, I started with scanning the victim host with `nmap`, but this was not needed in this task. The open port 22 for SSH connections was meant to misdirect users during this exercise. Because of that I'm not going to post any screenshots of typical enumeration with `nmap` here. I tried brute forcing SSH but with no success. With this walkthrough I'm just gonna go straight for the database.

I started with user `smokey` As it was written in the task. In the output I received password for this user which was: `vYQ5ngPpw8AdUmL`.

![1. Smokey login](/images/TryHackMe/Light/1_smokey_login.png)


If this is an exercise on SQL injections, it means that the database will be vulnerable to such attacks. I started using some unusual characters that wouldn’t typically appear in user input for normal databases to observe the database's response. The first interesting observation I made was that when I used `'`, I received the following error: `Error: unrecognized token: "''' LIMIT 30"`

![2.Error](/images/TryHackMe/Light/2_error.png)

This output provided me with really helpful information and insight into how the command is created on the database side. The query seems to be something like this:

```
SELECT password FROM table WHERE username='user_input' LIMIT 30
```

Unfortunately, I didn’t check in the browser what this error could mean. If I had, it wouldn’t have taken me as long as it did. In this walkthrough, I’ll jump straight into the error and explain how it was useful for my later penetration testing.
Firstly, I assumed this was a MySQL database and attempted to gather information using MySQL commands. However, after searching for this error, I found the following article:
[Solving 'Unrecognized Token' Error While Using SQLite Insert Command](https://stackoverflow.com/questions/57017469/solving-unrecognized-token-error-while-using-sqlite-insert-command). 
This revealed that the database was not MySQL but SQLite, which will be useful later in the task.

I started creating various SQLi payloads and encountered an interesting output. When I used `--`, a common part of SQLi, I received the following output:

![3. Filter1](/images/TryHackMe/Light/3_filter1.png)

This indicated that these characters were restricted. I continued experimenting with different SQLi techniques. For reference, here is a helpful [SQLi payload list](https://github.com/payloadbox/sql-injection-payload-list). At one point, I began using `UNION SELECT` payloads and observed another interesting behavior.

![4. Filter2](/images/TryHackMe/Light/4_filter2.png)


The filter was blocking `/*`, `--`, `%0b`, and `UNION SELECT` statements. However, it only flagged these operators when written entirely in lowercase or uppercase. If I used a mix of cases (e.g., `unioN selecT`), the filter allowed it.

![5. Filter3](/images/TryHackMe/Light/5_filter3.png)


I looked for SQLite-specific commands to identify the table names. This [helpful article](https://stackoverflow.com/questions/71160273/how-to-find-information-on-all-columns-in-a-sqlite-database) guided me to the following query:
```
SELECT name FROM sqlite_master WHERE type = 'table
```

Upper command is a way to go. I just need to tailor it for my needs. Based on what I learned so far I create below query:

```
' unioN Select name FROM sqlite_master WHERE type='table
```

![6. Table name](/images/TryHackMe/Light/6_table_name.png)

This returned the table name: `admintable`.


Here if I would use ‘UNION SELECT’ or ‘union select’ or mix of those I would not get the desirable response from the server as the filter is filtering for input that I described previously that's why I used one capital letter in each word.

Okay,  right now I have the table name and now I need to figure out what are the names of the columns used in the table. I browsed the internet looking on how I can get this information in SQLITE. At some point I found another [helpful article](https://stackoverflow.com/questions/947215/how-to-get-a-list-of-column-names-on-sqlite3-database) which gave me an idea on my next payload. By using the below syntax I managed to discover columns’ names.

```
' unioN seleCt sql FROM sqlite_master WHERE tbl_name='admintable' AND type='table
```

![7. Table collums](/images/TryHackMe/Light/7_table_collums.png)


Based on this output I know there are three columns named respectively `id`, `username` and `password`. 

Then I focused on discovering how many rows are in this table by searching the Internet and  I came up with this idea.

```
' uNion Select COUNT(*) FROM 'admintable
```

![8.Table rows count](/images/TryHackMe/Light/8_table_rows_count.png)

Now I know that there are two records In the table. One method that could be used right now is to enumerate the whole database with the correct SQL query by checking the username with each letter of the alphabet. If there are only two rows that means that it wouldn't be that much work to do. I decided to blindly use below command to figure out at least one of the usernames in the database and by knowing one of them I can create a query to figure out the second one.

```
' unioN selecT username FROM admintable WHERE username LIKE '%
```

![9. Admin username](/images/TryHackMe/Light/9_admin_username.png)


If you don't understand this query and how `WHERE` and `LIKE` operators work then you need to read on your own - that's how we learn! Also, this is not a place where I am focusing on explaining exactly how SQL queries are created and how they work.

This is an answer to the first question.

```
What is the admin username?
TryHackMeAdmin
```

Now I need to create a query to discover TryHackMeAdmin user password. So far I know everything I need to know to do it.

```
' unioN selecT password FROM admintable WHERE username LIKE 'Try%
```

![10. Smokey login](/images/TryHackMe/Light/10_admin_password.png)


This is an answer to the second question.

```
What is the password to the username mentioned in question 1?
mamZtAuMlrsEy5bp6q17
```

If there are two records in the table and I know one of them I can just create a reverse query by asking for a username that is not a username of  TryHackMeAdmin. To do that I will use `NOT LIKE` operator. 

```
' unioN Select username FROM admintable WHERE username NOT LIKE 'TryHackMeAdmin
```

![11. Flag username](/images/TryHackMe/Light/11_flag_username.png)

Now I know that the second username is `flag`. I will ask the database on what is this users’ password.

`' unioN selecT password FROM admintable WHERE username='flag`

![12. Flag password](/images/TryHackMe/Light/12_flag_password.png)

And this is the answer to the first question and the flag do its exercise

```
What is the flag?
THM{SQLit3_InJ3cTion_is_SimplE_nO?}
```


