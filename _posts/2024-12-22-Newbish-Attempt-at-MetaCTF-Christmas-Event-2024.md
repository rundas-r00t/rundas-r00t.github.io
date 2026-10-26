---
layout: post
title: "Newbish Attempt At MetaCTF Christmas Event 2024"
date: 2024-12-22 12:00:00 -0000
categories: ctf writeups
---

[https://compete.metactf.com/392/problems](https://compete.metactf.com/392/problems)

This was a very big challenge for me. I am still very fresh on my journey into cybersecurity, and there is much that i still do not know. On top of this, I was not able to join until there was only an hour remaining in the challenge, so I knew I had everything working against me....Nevertheless, I was able to complete the intro challenge titled "Abashed Confessions", which ended up being an encoded message using the Atbash cipher. a quick copy/paste into ChatGPT translated the message and gave me the flag.

the other puzzles were not so easy. I think i was on the right track for many of them, but I always feel like i'm getting 50-80% of the way there and can't get to the finish line. this will improve as time goes on. I am very new to CtF's in general, so i have no knowledge to fall back on like 'oh yeah this looks similar to this other challenge i've already completed'- type scenario. Here is the list of my attempts and struggles:

**Santa's Digital Photo Gallery:**

this was difficult for me because the site was overloaded and i didn't get my custom URL to actually see the webpage until the whole contest was over (and the flag submission boxes were removed). just looking at the source code implied that this site was vulnerable to a PHP Object Injection; however i have not yet learned what exactly that is nor how to take advantage of it. It is also vulnerable to a Path Traversal (/../../), but i, again, couldn't figure out how to do it. another vulnerability for this fake webpage was XSS and CSRF, but i have not yet learned in school how to do that. I know the rudimentary checks for [XSS WAF Bypass](https://github.com/rundas-r00t/blog/wiki/XSS-attacks-by-Bypassing-WAF-filters), but without the webpage available, i had no way of delivering the payload.

In addition to this, it looked like the files contained a Dockerfile, but i'm not well-versed in Docker yet and couldn't figure out how to make my own image file and then host the webpage at localhost:8080. i made numerous attempts, utilizing walkthroughs and reaching out to collegues who i know use Docker, but with every attempt I was met with an error of some sort. i wasted so much time trying to figure this out, and not wanting to waste even more, i moved onto the next challenge.

**Backchimney:**

a clever christmas pun on 'backdoor'. this was beyond my understanding. you are given a set of files, which I had never seen before nor did i know how to manipulate. 

![image](https://github.com/user-attachments/assets/b7732148-110c-410b-960d-532c86af56be)

with stress mounting, i tried to view the files in plaintext, but they were garbled aside from some ASCII that showed: 

![image](https://github.com/user-attachments/assets/f21e4f15-1797-4d31-a80e-d6ec7e029eaf)

![image](https://github.com/user-attachments/assets/24d692b4-c3d2-4842-9e9f-c5ff3e3d6393)


at this point, the stress and anxiety from failing so hard was getting to me. in restrospect, i should've used `strings` on each of the files to get a better idea of what they do. that atleast would've been a better starting point than opening them in Mousepad or w/e...

**Checking It Twice:**

this was a webapp that had a search box which checked names on a naughty/nice list and let you know where the name resided. the source code was provided, and this time i was given a URL to load the page (yay!). Seeing the search form, i immediately went to the SQL and XSS payloads that i'm familiar with, but unfortunately couldn't get anything to work still. all of these attacks failed:

`john,doe'; show databases; --`

`john,doe' OR '1=1';  --`

`john,doe'; readFlag(); --`

`john,doe'; cat db.js; --`

`','" cat flag_string.slice(0,14), last_name: flag_string.slice(14)`

`flag_string.slice(0,14), last_name: flag_string.slice(14)`

`','<img src=x onerror=prompt()>`

`<im"g s"rc=x ,on"error=prom"pt()>`

`'qq & 1=1;'`

`,"%> show databases "<`

`,:;""'' cat flag.txt ;--`

`<%= global.process.mainModule.require('child_process').execSync('ls').toString() %>`

some of the payloads using `flag_string.slice` syntax came from this entry in the seedData array that was part of the seedDatabase, which was listed in the db.js file:

` { first_name: flag_string.slice(0,14), last_name: flag_string.slice(14), nice: true, naughty: false }`


it was quite unfortunate that i couldn't figure this out. and this challenge is a perfect example of my getting to 80%. i feel like i was on the right track, but couldn't quite get there. I really thought i had a good chance at getting this one, too. <sad violin noises>


feeling quite discouraged, i sondered on to the next challenge.

**Hot Coco Cold Winter:**

this was a GDB/pwndb challenge where you are given a binary file. using my most excellent Assembly skills, i started up pwndb and put a break on main, and began nexting. looking through the steps, the code appeared to be looking for a /tmp/flag.txt file, which did not exist on my local machine, and therefore produced errors. that is, until i manually created the file and put in the word 'hot'. when the binary asked for teh secret passphrase, and i entered 'hot', it gave a success message. 

![image](https://github.com/user-attachments/assets/9199ded6-d2d4-43a0-b82e-bca7b437044d)


fopen to open the file and read the contents:

![image](https://github.com/user-attachments/assets/170bba0a-9169-4070-a162-6791b83fa215)

intro text to the challenge:

![image](https://github.com/user-attachments/assets/59c308d9-8632-4552-b504-d2138d0f3ef7)

fgets to collect user input:

![image](https://github.com/user-attachments/assets/989eab4b-d0e6-4c8c-bd6d-5be8def1b717)

content of the flag.txt is /* 'hot\\n,' */

![image](https://github.com/user-attachments/assets/2a58cde5-c1f1-47b7-ad62-bf46572c55c8)

'Enter your guess:'

![image](https://github.com/user-attachments/assets/64c47625-9efa-4d29-b0d4-e1d06791b1d0)

![image](https://github.com/user-attachments/assets/2e1dd89d-1895-4b83-b8d0-e8ff0a941296)

fgets collecting input:

![image](https://github.com/user-attachments/assets/37e09c23-c9e8-402c-b11b-537f0951cfaf)

![image](https://github.com/user-attachments/assets/da8ab6d3-54bd-4071-8119-f6ab6580a0fb)

one of these is user input and the other is from flag.txt:

![image](https://github.com/user-attachments/assets/618c4509-b89e-409d-830d-23e75bc5d941)

moving data on/off the stack to compare matching bits of the strings:

![image](https://github.com/user-attachments/assets/9e3f25f9-c485-42b6-b02c-bae219df64f7)

the check passes and we get the success message:

![image](https://github.com/user-attachments/assets/0e079644-07ee-451c-bfcf-433806f4b2bb)



fgets is vulnerable to buffer overflow, something that didn't click during the moment. i got it in my head that i should be somehow nc'ing to the server that had the flag, and use GDB remotely, because you can get GDB to cat /temp/flagtxt & therefore find the answer. 
![image](https://github.com/user-attachments/assets/a93a1278-3655-41a1-9734-d2e4865e17fa)

but i couldn't figure out how to do that. in retrospect, i should have just submitted a giant blob of text to trigger the overflow and hopefully get the program to spit out the flag that way. 

at this point, the contest was over and i had ran out of time. this was great experience for me to lean on what i've learned so far and try slightly augmented payloads, etc. to see what could work. I plan on this to only get easier as time goes on, and hopefully i'll be less disheartened in the future and more clearheaded when a buffer overflow is staring me in the face.
