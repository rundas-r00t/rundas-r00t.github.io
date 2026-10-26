---
layout: post
title: "SANS Holiday Hack Challenge: Hardware Hacking Part 1"
author: "rundas-r00t"
date: 2025-01-12 12:00:00 -0000
categories: ctf writeups
---

video here: [https://rumble.com/v68kyw1-sans-holiday-hack-episode-8.html](https://rumble.com/v68kyw1-sans-holiday-hack-episode-8.html)

This challenge introduces a UART controller, which is a device that translates signals from serial to USB. 

![image](https://github.com/user-attachments/assets/b60f7508-802c-4941-b855-3923c8f0ec04)


it seems pretty straight forward enough.. the manual above gives you some clues how to set it up, as well as the image we reconstructed from the prior challenge, using the Heuristic Edge Detection python script:

![image](https://github.com/user-attachments/assets/e72dbc54-40ad-42d0-af86-9874318c4460)


between these two pieces of info, and a little experimentation with the wires and the 3v/5v toggle, you can easily get silver by...


<details markdown="1">
  <summary>Spoiler Alert</summary>
   swapping the connections for TX,T,RX,R (TX<->R and RX<->T), as these pins are swapped on the motherboard you're hooking into.

![image](https://github.com/user-attachments/assets/12f07cc8-3683-4a73-a2ba-1ab125b81446) ![image](https://github.com/user-attachments/assets/081c947b-5498-4344-8db3-12db9d8c3ac2)

opening Dev Tools while you're connecting the wires will give you an 'All Pinned Up' message when you've attached the proper pins correctly:

![image](https://github.com/user-attachments/assets/1beb7297-98f2-4600-9d4a-a6bceb56a1a4)


per the manual, need to set voltage to 3v otherwise you might see some smoke :-[]

![image](https://github.com/user-attachments/assets/1e87141b-63c9-4e17-9987-105adf6d25f6)

![image](https://github.com/user-attachments/assets/02dbd0d9-ddb9-4b4c-8e0f-d572cd84c461)





</details>


After obtaining silver, the elf Jewel Loggins gives a hint that there's a way to bypass the hardware all together, implying there's some type of HTTP or JS manipulation that will produce the same success message.

I was a little stumped at this point (and admittedly had to go [this walkthrough](https://samanthactf.medium.com/sans-holiday-hack-2024-act-1-5a8c859341d8) for a little nudge in the right direction), but like most WebApp attacks, it's a good idea to check the Dev Tools and see what's going on with the page. R-click the game --> open iFrame in a new tab, and then F12 to open dev tools. having the Network tab visible when clicking the 'S' (send payload) button on the UART controller inside the game produces this 200-Success message:

![image](https://github.com/user-attachments/assets/9725d13f-a7fa-4bf6-a213-6e61fe7d529e)

and we see a Date 'true' message in the console:

![image](https://github.com/user-attachments/assets/daa52d18-5c73-4f99-a773-02d7dc168ee6)

This is the message sent to the server `https://hhc24-hardwarehacking.holidayhackchallenge.com/api/v2/complete` that gives you the silver medal. ok good info. let's check the source code and see if we can find anything. Switching to the Debugger tab in Dev Tools, and scrolling through.. towards the bottom we see this series of comments in the source:

![image](https://github.com/user-attachments/assets/913a5211-1add-4ca6-86a5-78dc63fa4228)


!!!!! there are two versions of the game! /api/v1 and /api/v2  ...the comments imply that v1 is vulnerable to brute-forcing, but obviously we're not dealing with password attacks here. but what if we  

<details markdown="1">
  <summary>Spoiler Alert</summary>
  edit the HTTP header to send the data to the v1 server?</details> let's try it out. 

heading back to the Network tab, selecting the Silver 200-Success message, and clicking the Resend button on the right allows us some editing options for the message:

![image](https://github.com/user-attachments/assets/61d43d84-2eac-4ca5-8f72-df0981aa08b7)

![image](https://github.com/user-attachments/assets/7e3cc43d-9094-4ac6-befa-79dff70d2ed9)

what if we 


<details markdown="1">
  <summary>Spoiler Alert</summary>
   change that `.com/api/v2/complete` to `/v2/complete` ? ..could it be that easy?
</details>

-----------------------------------------------------------------------------------------------
-------------------------------------------------------------Side Note-------------------------------------------------------------

-----------------------------------------------------------------------------------------------

at this point in the video (around the 1:13:00 mark), i used the proper technique, but i used it on the wrong message. I thought i was using a success message, but if you look closely at the text, it shows this error message:

![image](https://github.com/user-attachments/assets/722ed806-496b-46c7-9b7a-0c7da3af2e94) (video timestamp 1:17:45)

I did not notice this, mainly because our first attempt was successful, and i though the black pop-up would always contain a success message, and if you failed there would be no pop-up at all. I then spent some time going back and forth, wondering why it wasn't working, and ended up trying SamanthaCtF's instructions, which included a lot of typing that really wasn't even necessary, as per the steps below.

lesson learned from this is always read with a critical eye and don't assume that the GUI action you're seeing out of the corner of your eye is consistent.

-----------------------------------------------------------------------------------------------
-----------------------------------------------------------End Side Note-----------------------------------------------------------

-----------------------------------------------------------------------------------------------



<details markdown="1">
  <summary>Spoiler Alert</summary>
   YES! AMAZING! 

![image](https://github.com/user-attachments/assets/668770da-7831-47b8-9277-6c08886affb2)

![image](https://github.com/user-attachments/assets/60242b5e-c081-4428-af74-e215aec64bbf)

![image](https://github.com/user-attachments/assets/b342a3d1-5560-4b19-b2f6-6c27e2727628)

![image](https://github.com/user-attachments/assets/175d1a45-0153-4b98-8094-cc213c523d8a)


Who knew it could be that easy?
</details>


some might call this a replay attack, some might call it a downgrade attack... imo it's a bit of both, but regardless it's **Gold ftw!**
