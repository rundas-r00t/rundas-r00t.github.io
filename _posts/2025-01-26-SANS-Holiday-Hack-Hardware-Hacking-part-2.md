---
layout: post
title: "SANS Holiday Hack Challenge: Hardware Hacking Part 2"
author: "rundas-r00t"
date: 2025-01-26 12:00:00 -0000
categories: "ctf writeups"
---

[https://rumble.com/v6dsals-sans-holiday-hack-2024-episode-9.html](https://rumble.com/v6dsals-sans-holiday-hack-2024-episode-9.html) and Episode 10.

the challenge presents you with a linux terminal and a method to create new ID cards using the slh tool:

![image](https://github.com/user-attachments/assets/2ab3da77-4b97-4c69-8654-15764a2d369d)

the elf wants you to create a new ID card using ID=42, but we also need to find the missing password. after poking around the directories and trying to cat and grep with no success (partially b/c the terminal uses `egrep`, not `grep`), i had to look at the Elf Hints from the information menu:

![image](https://github.com/user-attachments/assets/504cf9bc-cc8a-4bc7-b738-87f562b31aff)

dang, it really is that easy.. typing `history` into the console revieals a history list with the missing passcode

![image](https://github.com/user-attachments/assets/4729fec9-525e-4cd1-b8c9-7fcda7b6945a)

<details markdown="1">
  <summary>Spoiler Alert</summary>
11           slh --passcode CandyCaneCrunch77 --set-access 1 --id 143
</details>

Now that we've found the missing password, we can create the new ID card with a quick one-liner. lets do it:

<details markdown="1">
  <summary>Spoiler Alert</summary>
  slh --id 42 --passcode CandyCaneCrunch77 --set-access 1
</details>

![image](https://github.com/user-attachments/assets/2c0153f0-15a6-4a67-8c97-887a91a7c001)


BOOM! Silver is done.

The elf then says there's a way to create a custom HMAC signature and modify the database directly to earn the Gold medal.

the hint points us to CyberChef:

![image](https://github.com/user-attachments/assets/14028bd2-fd5c-49ef-926e-c185891327eb)

https://gchq.github.io/CyberChef/#recipe=HMAC(%7B'option':'UTF8','string':''%7D,'SHA256')

ll'ing gives us the standard hidden files, plus one file named `access_cards`. running the `file` command reveals that this file is an SQLITE3 database. running `which sqlite3` indeed confirms that we can use sqlite3 to manupulate the database:

![image](https://github.com/user-attachments/assets/35efba45-a3f5-4bb3-8747-a626be0698e8)


![image](https://github.com/user-attachments/assets/2384d4f9-ca7f-4c52-847a-033973d1d619)

let's check out the database by `sqlite3 access_cards` and then `.tables`:

![image](https://github.com/user-attachments/assets/77d710cb-8dfd-439a-8f5a-3be7575a0c78)


the access_cards table seems fairly obvious, it's going to house all the cards. this is the table we will need to manipulate later to `INSERT INTO` once we have the HMAC info. let's check out the config table by typing `select * from config;`"



![image](https://github.com/user-attachments/assets/a2ee4ec1-63df-45b2-8301-e16833431bdb)

<details markdown="1">
<summary>Spoiler Alert</summary>

1|hmac_secret|9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e

2|hmac_message_format|{access}{uuid}

3|admin_password|3a40ae3f3fd57b2a4513cca783609589dbe51ce5e69739a33141c5717c20c9c1

</details>

AH! we have the information we need! ...the only problem is that i don't know the process of how to use this information with Cyberchef to generate the custome HMAC. at this point, if you know how to use it, getting gold should be trivial. For me, however, I will have to do some research and try again on Episode 10 to see if i can win the GOOOOOOOOLLLLLLLLLD!

--edit/update--

the recipe i was using was correct, but the output was not correct, and i have no idea why. on the [this stream](https://rumble.com/v6lk3cy-sans-2024-holiday-hack-is-over-time-for-some-picoctf.html), even though the audio didn't come through on the recording, i point out that on one tab, i have an 'output' field with a magic wand next to it, and on my original recipe, the magic wand thing isn't there. all other aspects of the page are the same, yet the output is different. I still have no idea why this is the case. but this is what was tripping me up..the methodology was correct, but the output from CyberChef was different. 

SANS 2024 Holiday Hack is officially over, so we're having to move on to PicoCtf!

