Firstly I started with scanning The Host with nmap, but this was not needed in this task. The open port number 22 for a SSH connections Was meant to misdirect users during this exercise. Because of that I'm not going to post any screenshots of typical enumeration with nmap here. I tried brute forcing SSH but with no success. With this walkthrough I'm just going to go straight for the database.

Firstly I started with user ‘smokey’ As it was written in the task. in the output I received password for this user which was: `vYQ5ngPpw8AdUmL`

If this is an exercise on SQL injections, it means that the database will be vulnerable to such attacks. I started using some unusual characters that wouldn’t typically appear in user input for normal databases to observe the database's response. The first interesting observation I made was that when I used `’`, I received the following error: `Error: unrecognized token: "''' LIMIT 30"`
