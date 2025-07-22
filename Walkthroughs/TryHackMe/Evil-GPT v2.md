# Evil-GPT_v2

Target: 10.10.42.156

Unlike the previous challenge, no initial hints were provided this time. In the first part of this series (Evil-GPT), the task required establishing a connection to another host using the command `nc <IP> 1337`. This challenge, however, doesn't follow that pattern. I started by scanning the target host using the following command:

`nmap -p- 10.10.42.156`

`-p-` - enables full port scanning (scans all 65535 ports).

![1. Nmap scan](/images/TryHackMe/Evil-GPT_v2/1_first_nmap.png)

The scan revealed several exposed services on the victim host. First, I decided to check the web service running on port `80`.

The target exposed a web application that allows interaction with a language model capable of responding to custom prompts. In the example below, I initiated the conversation with a polite greeting.

![2. AI hello](/images/TryHackMe/Evil-GPT_v2/2_AI_hello.png)

I then directly asked the AI to provide the flag, but the model refused:

![3. Flag request rejected](/images/TryHackMe/Evil-GPT_v2/3_flag_rejected.png)

It responded with a lengthy message, which I won’t include in full here. But flag was not provided.

Next, I attempted to extract `/root/flag.txt` using a "comforting grandma" prompt injection strategy - essentially trying to manipulate the assistant into role-playing in a way that might bypass implemented security policies.

![4. Grandmother tatics](/images/TryHackMe/Evil-GPT_v2/4_grandma.png)

Unfortunately, this tactic didn't succeed.

I also asked explicitly for any SSH credentials that might be stored in a file like ssh.txt, but again, the AI assistant refused to cooperate.

![5. SSH creds](/images/TryHackMe/Evil-GPT_v2/5_ssh_creds.png)

It consistently referenced internal security policies as the reason for withholding information. Therefore, I tried a different angle: I asked the AI assistant to share its policies themselves.

![6. Policy and flag](/images/TryHackMe/Evil-GPT_v2/6_policy_and_flag.png)

This led to the assistant disclosing Policy Rule #3, which conveniently contained the flag:

THM{AI_NOT_AI}
