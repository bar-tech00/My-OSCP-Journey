# W1seGuy CTF Walkthrough

Target: 10.10.178.10

At the beginning of the challenge, as suggested in the task, I downloaded the file from the “Task Files” section. This file contained a Python script, which I opened in a text editor and analyzed. I recommend reviewing the code yourself to understand the logic better. Here are some crucial takeaways from the script:
* The code is executed on the target machine and sets up a server, which we can connect to - this becomes evident later when we connect using `netcat`.
* The script contains an example of how a flag should look. It uses: `THM{thisisafakeflag}`. This suggests the real flag will likely follow the same format: starting with `THM{` and ending with `}`. This will become very useful later. In fact, most TryHackMe flags follow this format.
* We can also observe how the encryption key is generated - it consists of uppercase letters, lowercase letters, and digits, and is 5 characters long:
  ```
  res = ''.join(random.choices(string.ascii_letters + string.digits, k=5))
  key = str(res)
  ```
* The encryption algorithm used to encrypt the message looks like this:
  ```
  xored = ""    
  for i in range(0,len(flag)):
    xored += chr(ord(flag[i]) ^ ord(key[i%len(key)]))
  ```
* The encrypted message is first encoded and then translated to hex format before being sent over the network:
  ```
  hex_encoded = xored.encode().hex()
  ```

Let’s now connect to the target using the hint provided in the task - port 1337:
```
nc 10.10.178.10 1337
```
Once connected, a banner is displayed showing the encrypted message and explaining the operations applied to it. This is the message we’ll need to decrypt.

![1. Banner](/images/TryHackMe/W1seGuy/1_banner.png)

I believe we now have everything we need to solve this challenge. My plan was to write a Python script that would reverse the encryption to recover the original message.

To do that, we need to find the key used in the encryption process. All of the information gathered so far will help us achieve this. Below is my code, which I’ve broken into logical sections that represent each step of the decryption process.

Part 1: Initialization
```
msg_encoded_hexed = "1504220a04702d031f0004341b300035780c1a1700221d42152d0016192133381641013334200309"
possible_chars_list = "qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM1234567890"
flag = "THM{"
last_char_of_the_flag = "}"
realkey = ""
all_possible_4_char_keys_list = []
```
In the first section, I declare all necessary variables:
* `msg_encoded_hexed` is the encrypted message we got from the banner (on port 1337).
* `possible_chars_list` includes all possible characters (letters + digits) used in the key.
* Since we know the flag starts with `THM{`, we assign that to the flag variable.
* The last character of the flag is a curly brace `}`, so we store it in a separate variable.
* `realkey` will store the final key once it's discovered.
* `all_possible_4_char_keys_list` is a list that will store all 4-character key combinations (we’ll explain why just 4 characters are enough in the next section).


Part 2: Generating 4-character key candidates
```
# Finding all possible 4-characters keys
for i in possible_chars_list:
    for j in possible_chars_list:
        for a in possible_chars_list:
            for b in possible_chars_list:
                    new_key = i+j+a+b
                    all_possible_4_char_keys_list.append(new_key)
```
In this step, I generate all possible 4-character keys. Even though the original key is 5 characters long, I only generate 4-character combinations. Why? Because we only know the first 4 characters of the flag -> `THM{)`, and looking at the encryption algorithm, it performs a modular operation `(i % len(key)`. This means the key wraps around during encryption. Therefore, the fifth character of the key doesn't influence the encryption of the first 4 characters of the flag. This optimization significantly reduces the total number of key combinations - skipping the fifth character cuts down our search space by a factor of 62.

Part 3: Finding the first 4 characters of the key
```
# Finding first 4 characters of the key
for key in all_possible_4_char_keys_list:
    xored = ""
    for i in range(0, len(flag)):
        xored += chr(ord(flag[i]) ^ ord(key[i % 5]))
    hex_encoded = xored.encode().hex()
    if hex_encoded.startswith(msg_encoded_hexed[:8]):
        print("First 4 char of the key: " + key)
        realkey += key
```

In this section, I replicate the encryption process from the original script using the known prefix of the flag `THM{` and each candidate key. I encrypt the flag snippet with each 4-character key encode and perform hex function on the output. Then I compare the result with the beginning of the real encrypted message from the banner. If the first 8 hex characters match, we’ve found the correct 4-character prefix of the key.

Part 4: Finding the last character of the key
```
# Finding last character of the key
for i in possible_chars_list:
    lastXored = ""
    lastXored += chr(ord(last_char_of_the_flag) ^ ord(i))
    lastXored_encoded = lastXored.encode().hex()
    if lastXored_encoded == msg_encoded_hexed[-2:]:
        print("Last char of the key: " + i)
        realkey += i
```
In this section, I focus on discovering the fifth (last) character of the key. We already know the last character of the flag is `}`, and due to the nature of the encryption (with modulo wrapping), the last character of the ciphertext corresponds to this flag character encrypted with the last key character. To find the correct one, I XOR the `}` with every possible character from `possible_chars_list`, encode it, convert it to hex, and compare it with the last two characters of the encrypted message. If it matches, we’ve found the missing character and can now reconstruct the full 5-character key.

Part 5: Decrypting the full message
```
# Deciphering the message
print("The real key is: " + realkey)
msg_in_ciphertext = str(bytes.fromhex(msg_encoded_hexed).decode("utf-8"))
print("Ciphertext is: " + msg_in_ciphertext)
real_message = ""
for i in range(0, len(msg_in_ciphertext)):
    real_message += chr(ord(msg_in_ciphertext[i]) ^ ord(realkey[i % len(realkey)]))
print(real_message)
```

Now that we’ve recovered the full 5-character key, we can proceed to decrypt the entire message. Steps:
1. Convert the hex string back into bytes,
2. Decode the bytes to get the XOR-ed ciphertext,
3. Perform XOR between each character of the ciphertext and the corresponding character from the key,
4. Print the final message.

Upon execution above code I received the first flag.

![2. First flag](/images/TryHackMe/W1seGuy/2_first_flag.png)

Flag: THM{p1alntExtAtt4ckcAnr3alLyhUrty0urxOr}

Now that we know the key which was used for encryption we can answer the question asked in the banner. After submiting the found key we recive the second flag.

![3. Second flag](/images/TryHackMe/W1seGuy/3_second_flag.png)

Flag: THM{BrUt3_ForC1nG_XOR_cAn_B3_FuN_nO?}
