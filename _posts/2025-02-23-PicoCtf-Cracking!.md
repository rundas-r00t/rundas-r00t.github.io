---
layout: post
title: "PicoCTF Cracking!"
date: 2025-02-23 12:00:00 -0000
categories: ctf writeups
---

# Password Cracking

https://rumble.com/v6oe7ai-picoctf-episode-2.html
https://rumble.com/v6oen2u-picoctf-episode-2-part2.html

after my computer blue-screened and i got the stream back up and running, I went to https://play.picoctf.org/ and looked for some fun Medium-difficulty challenges, and found password cracking challenges PW Crack 3, PW Crack 4, and PW Crack 5. all 3 of these challenges give you a binary file for the hash and an encrypted txt file for the flag, as well as a python script to iterate the password against the flag file. 

PWC3 gives you a wordlist of 7 possibilities. since this was so small, i just manually checked each one until the correct password was entered and the flag was obtained. 

PWC4 is the same set-up but with a wordlist of 100 different possibilities, listed as an array inside the python script source. using ChatGPT, i created a one-liner loop to check for each of the given passwords in the list:

<details>
  <summary>Click to reveal</summary>

`for pw in 8c86 7692 a519 3e61 7dd6 8919 aaea f34b d9a2 39f7 626b dc78 2a98 7a85 cd15 80fa 8571 2f8a 2ca6 7e6b 9c52 7423 a42c 7da0 95ab 7de8 6537 ba1e 4fd4 20a0 8a28 2801 2c9a 4eb1 22a5 c07b 1f39 72bd 97e9 affc 4e41 d039 5d30 d13f c264 c8be 2221 37ea ca5f fa6b 5ada 607a e469 5681 e0a4 60aa d8f8 8f35 9474 be73 ef80 ea43 9f9e 77d7 d766 55a0 dc2d a970 df5d e747 dc69 cc89 e59a 4f68 14ff 7928 36b9 eac6 5c87 da48 5c1d 9f63 8b30 5534 2434 4a82 d72c 9b6b 73c5 1bcf c739 6c31 e138 9e77 ace1 2ede 32e0 3694 fc92 a7e2; do echo $pw | python3 level4.py; done`

</details>

For PWC5, it gives you a dictionary wordlist to loop through, but i didn't want to continue solving the challenge using the python script and `for` loops. i much prefer using hashcat. so i ditched the py-script, and converted the binary hash file back in to ASCII using this one-liner:

<details>
  <summary>Click to reveal</summary>

`xxd -p level5.hash.bin > hash.txt`

</details>

then i used hashcat to crack the hash. inside the python script source code, it tells you what type of hash to use:

<details>
  <summary>Click to reveal</summary>

`def hash_pw(pw_str):`

    `pw_bytes = bytearray()`

    `pw_bytes.extend(pw_str.encode())`

    `m = hashlib.md5()`

    `m.update(pw_bytes)`

    `return m.digest()`

</details>

so now the easy one-liner using hashcat:

<details>
  <summary>Click to reveal</summary>

`hashcat -a0 -m0 hash.txt dictionary.txt`

</details>

and 
<details>
  <summary>Click to reveal</summary>

  `hashcat -a0 -m0 hash.txt dictionary.txt --show`

![image](https://github.com/user-attachments/assets/8cff906c-3890-4ba5-8ac8-8b55fb1a342f)

</details>

BOOM! we have the proper password, now we just load up the python script and enter the password to get the flag:
<details>
  <summary>Click to reveal</summary>

 `python3 level5.py`

![image](https://github.com/user-attachments/assets/9ff001ec-5bc9-4182-9ee4-aff833bc552a)


</details>

PWC5? DONE!


...now on to WebApp hacking with Pico Challenge named "No SQL Injection". 

admittedly, prior to this challenge, i though it was just a clever name, like "there's no sql injection here so don't try that, try something else". so immediately i'm thinking, ok it's command injection. I had no idea until just now that there is a language called "NoSQL". the existence of the space between "No" and "SQL" in the title of the challenge completely threw me off. Needless to say, i was unable to complete this challenge, namely because i had never heard of NoSQL before. the syntax is different from MySQL/SQLite, and all of my payloads were wrong. I even tried scanning with sqlmap, which was unsuccessful at finding any vulnerabilities. I will have to do some [light reading](https://portswigger.net/web-security/nosql-injection) to find out how NoSQL works and what payloads might be successful. Time to hit the books!! :-D

