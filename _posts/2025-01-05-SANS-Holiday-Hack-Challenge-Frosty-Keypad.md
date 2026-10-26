---
layout: post
title: "SANS Holiday Hack Challenge: Frosty Keypad"
author: "rundas-r00t"
date: 2025-01-05 12:00:00 -0000
categories: ctf writeups
---

[https://rumble.com/v667ir7-sans-holiday-hack-episode-7.html](https://rumble.com/v667ir7-sans-holiday-hack-episode-7.html)

on the last attempt at this challenge, I obtained Silver by punching in the proper digits that cooresponded to the fingerprints on the keys (visible after collecting the UV flashlight). Today, we're going for gold.

After getting silver, you get an item "One Thousand Little Teeny Tiny Shredded Pieces of Paper", which includes a link to a zip file: https://holidayhackchallenge.com/2024/shreds.zip ..This is to be used in tandem with [Arnydo's Heuristic Edge Detection script](https://gist.github.com/arnydo/5dc85343eca9b8eb98a0f157b9d4d719) to reassemble the shredded image back into it's proper form. unfortunately, the image is still distorted, sliced into a third and mirrored horizontally:

![image](https://github.com/user-attachments/assets/f0cca03d-85ee-4cb9-ac11-f0201ff3883c)


fixing this in paint (b/c i'm not a photoshop/gimp guru) results in this image:

![image](https://github.com/user-attachments/assets/316d765d-ea26-466e-9a5c-71845f31cedb)

i did a little bit of research with ChatGPT, who said that the writing refers to a serial communication setup, likely for an RS-232 configuration:
* A high-speed connection with 115200 bps.
* Error-checking with EVEN parity.
* 7 bits of actual data per frame.
* 1 stop bit for frame synchronization.
* Hardware flow control using RTS.

this apparently is a dead-end, atleast for me with my current knowledge. i could not understand how this information was to lead me to a secret 5-digit code containing digits 2, 6, 7, & 8. but using Arnydo's tool was a very cool experience and i'm glad i git-cloned the page to my Kali VM, in case i need to use it in the future.

so! dead-end. back to the drawing board, which was a 20sqft blank canvas. I was stuck. Without any ideas, I consulted the walkthrough written by SamanthaCtF at medium.com. She utilised the Intruder\Sniper Attack\Brute Force payload option with Burp Suite to inject all 5-digit iterations of 2678. So naturally, i fired up Burp, enabled the redirect in Firefox, and sent the proxy info to Intruder. Setting up the Brute Force was easy, but i did stumble a bit, as this was my first attempt using this feature of Burp. In the Intruder window, i double-clicked the section of the HTML request where the 5-digit number belonged, and clicked the 'Add§' button to tell Burp that this is where the payloads should go. 

![image](https://github.com/user-attachments/assets/cb0125d4-ba9e-4a03-b42d-c448f37567b8)

I then set-up the Brute Force Payload, choosing 'brute forcer' in the dropdown, character set of 2678, and max/min length of 5:

![image](https://github.com/user-attachments/assets/176bf878-fd50-4ca1-b2d6-67a0b5c95f91)

then i clicked the 'Start Attack' button, which produces a pop-up of all the attempts:

![image](https://github.com/user-attachments/assets/72365ab1-8fa2-4658-a89f-6b24ffa10dd5)

Notice this shows 1024 total iterations. this will take FOREVER if you're using the free/community version of Burp, because as the completion percentage increases, the iterations become throttled. i did not want to wait this long, and i noticed that Burp was not respecting the 'repeat one digit twice' that i would have liked to stipulate, but couldn't figure out how. i did not see any hashcat-like masking that i could take advantage of. this is why i had so many possibilities, because Burp will happily repeat a digit 3 or 4 times as shown here:

![image](https://github.com/user-attachments/assets/f6cec455-dc1b-48ab-9b61-fe3baea51a87)
![image](https://github.com/user-attachments/assets/c3ad8826-a0e4-4aa6-993a-ae115f5381ad)

i also found dark-mode in Burp :-D:

![image](https://github.com/user-attachments/assets/d9741e9f-4b74-452a-980a-b2debe487d03)

sooo sloooowwww....this is U-N-A-C-C-E-P-T-A-B-L-E !!! why? b/c AIN'T NOBODY GOT TIME FOR THAT!! 

![image](https://github.com/user-attachments/assets/030b5428-a635-47b3-a8f9-7216eb04ba53)


so, i canceled this attack. i played around with different payload types until i saw the 'simple list' option. ahHA! i can give it a list of payloads to try! perfect!

so, i went outside my VM, and told ChatGPT to make me a list of the possible combinations of 2678, repeating only one digit, producing a 5-digit number:

![image](https://github.com/user-attachments/assets/2d70b967-3301-49d1-8afa-200bce1a4855)





copy/pasting this into a txt file inside my VM:

![image](https://github.com/user-attachments/assets/f74e5fdd-e8b2-47fb-8ab8-1cb01b4cf68b)


replacing spaces with `\n` character:

![image](https://github.com/user-attachments/assets/51bb7bb9-3a59-4b66-aa88-ebba7da5207f)




i used this list with Burp\Intruder\Brute Force and re-set-up the attack:

![image](https://github.com/user-attachments/assets/0ac15dde-04ce-47c0-bffe-a155411c7ed9)

now we only have 212 iterations to process! much success!! AND because ChatGPT semi-ordered the list from smalled to largest, we actually got lucky and found the secret code in less than 10 iterations!!

![image](https://github.com/user-attachments/assets/207f7d41-8b30-495c-8abc-4e3cf85a4e3f)


status code of 200=GREAT SUCCESS!! 

![image](https://github.com/user-attachments/assets/f6060850-d9da-4973-98b9-59736c4982ad)

Entering this code into the keypad solves the challenge and awards GOOOOOOOOOOOOOOOOOOLLLLLLLLLLLLD :-D

![image](https://github.com/user-attachments/assets/45a80d79-0e57-40b5-baeb-d4e06829959a)


![image](https://github.com/user-attachments/assets/2c12236f-cd52-48dc-9320-6bb0cef65bc0)


![image](https://github.com/user-attachments/assets/3adbd27b-82c8-4171-8d92-44522cbac109)







