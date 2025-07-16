# Evil-GPT CTF Walkthrough

In this walkthrough, I will demonstrate how I completed the Evil-GPT challenge on TryHackMe.

Victim: 10.10.79.85

Firstly, as it was written, I had to connect from my host to a victim host by using `nc 10.10.79.85 1337`. After typing this command in the terminal the following screen appeared.

![1. First view.](/images/TryHackMe/Evil-GPT/1_first_view.png)

I am connected to the system I have to compromise.
I started by typing ‘ls -la` command to list the contents of a directory I am currently in.

![2. ls -la.](/images/TryHackMe/Evil-GPT/2_ls_la.png)

You can see that the command was understood by the system. It listed the files for me. I was interested in the ‘evilai.py` file, so I will try to display its contents. I will use command ‘cat evilai.py` to do so.

![3. cat evilai.py.](/images/TryHackMe/Evil-GPT/3_cat_evilai.png)

Strange... The system understood my command as ‘python evilai.py` and after I accepted the command it threw me an error that python was missing on the machine. I need to try a different way. Maybe the ‘tail` command will work?

![4. tail evilai.py.](/images/TryHackMe/Evil-GPT/4_tail_evilai.png)

The system understood my command differently and displayed the entire Python file. From what I can tell after quickly analyzing the script, it`s the one responsible for listening for incoming connections on port 1337 on the vulnerable machine. However, in the import section, something caught my attention: ‘import ollama`. Does this mean there`s actually a language model running on the machine, converting user queries into commands executed on the system? Maybe it`s not necessary to write exact commands - perhaps giving instructions in plain language is enough? Let`s check. As a test, I`ll try to find out who I am using both of the mentioned methods.

![5. whoami.](/images/TryHackMe/Evil-GPT/5_whoami.png)

I used the command ‘whoami`, which the system interpreted as showing the ‘$USER` variable. Then I asked the system in natural language, ‘“What is my role as a user?”`, which it understood as the ‘whoami command`. Interestingly, the system replied that my role is ‘root`!

Spróbujmy poszukać flagi za pomocą komendy ‘find`.
Let`s try finding the flag with the ‘find` command.

![6. find.](/images/TryHackMe/Evil-GPT/6_find.png)

Unfortunately, the ‘find; command takes too long to execute, and the system returns a timeout. I decided to read ‘root` folder as shown below.

![7. Highest folder.](/images/TryHackMe/Evil-GPT/7_highest_folder.png)

The system understood that it should display the top-level directory, meaning ‘/`. So, I tried again, this time adding the ‘/` before ‘root` to read the contents of the ‘/root` folder.

![8.Root folder.](/images/TryHackMe/Evil-GPT/8_root_folder.png)

Thanks to that, I found a file named ‘flag.txt`. So, I politely asked the system to display the contents of the ‘/root/flag.txt` file.

![9. FLag.](/images/TryHackMe/Evil-GPT/9_flag.png)

Flag: THM{AI_HACK_THE_FUTURE}
