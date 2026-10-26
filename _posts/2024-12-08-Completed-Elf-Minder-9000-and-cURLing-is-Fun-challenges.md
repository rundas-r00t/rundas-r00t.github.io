---
layout: post
title: "Completed 'Elf Minder 9000' and 'Curling Is Fun' Challenges"
date: 2024-12-08 12:00:00 -0000
categories: ctf writeups
---

video walkthrough here: [https://rumble.com/v5xchyt-rundas-gets-r00t.html](https://rumble.com/v5xchyt-rundas-gets-r00t.html)

Finally got the Elf Minder 9000 challenge, using SamanthaCTF's advice from her article on [medium.com](https://samanthactf.medium.com/sans-holiday-hack-2024-prologue-dfa78d984b5c). i had mis-clicked a square and that's why the elf was catapulting back to start instead of hopping to the finish.


with that challenge out of the way, i successfully attempted the 'cURLing Is Fun' challenge, the first challenge of Chapter 1. to begin, i followed the directions to completion, with the help of the help menus and a little help from ChatGPT, due to my mousewheel being broken and unable to scroll through the entire extended help menu. there was little beyond following the directlions, mainly looking up different flags/switches/parameters for using with the 'curl' command. silver obtained. 

the gold-level mystery for this challenge is completing it in 3 commands. i experimented with a variety of attempts by stringing multiple commands together, such as:

`curl https://curlingfun:9090 -k -d "skip=alabaster"; curl https://curlingfun:9090 -k -b "end=3"; curl https://curlingfun:9090 -k -I; curl https://curlingfun:9090 -k --header "stone:granite"; curl https://curlingfun:9090/../../etc/hacks -k --path-as-is`

but the challenge could not be short-cutted via this method, because the typical way locks you into following the prompts. and we don't want to follow the prompts. therefore, the hunt begins.

i opened up DevTools for the page, and began skimming and searching through all the js files that the page was using, trying to build off my knowledge from prior challenges. i was hoping to find some source-code that showed me the '....successfully..'' prompt for passing each step. my thought process here was that in prior challenges, you could edit the client-side stored data to circumvent steps or take advantage of bugs, etc. so that's where i began, thinking i could shortcut some steps by utilizing a certain syntax or editing stored data to make a buffer overflow or something of that nature. but nothing i was doing panned out.

then a eureka moment: the challenge begins in a linux terminal! this allows us to use whatever CLI command we want. 

pwd, whoami, ls -alh

AH! this exposes a file named 'HARD-MODE.txt'

catting the file produces text: 

![image](https://github.com/user-attachments/assets/4fd5aaa5-67a7-47f9-9fe7-495df72e374a)

from here, i followed the instructions, using combined statements, such as:

<details markdown="1">
  <summary>Spoiler Alert</summary>
  curl https://curlingfun:9090 -k -d "skip=bow" -b "end=10" --header "Hack:12ft"
</details>

then, to answer the 2nd question, repeat the same syntax from step 7 of the regular challenge

<details markdown="1">
  <summary>Spoiler Alert</summary>
   curl https://curlingfun:9090/../../etc/button -k --path-as-is
</details>

once succesful, the directions provide you with a secret URL

finally, the third challenge utilizes the location switch to update the secret URL's landing page, granting you gold:


<details markdown="1">
  <summary>Spoiler Alert</summary>
   curl https://curlingfun:9090/GoodSportsmanship -k -L
</details>
