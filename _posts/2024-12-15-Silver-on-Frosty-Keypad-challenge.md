---
layout: post
title: "Silver On 'Frosty Keypad' Challenge"
date: 2024-12-15 12:00:00 -0000
categories: ctf writeups
---

https://rumble.com/v5zoa0q-sans-holiday-hack-2024-episode-6.html

the challenge presents you with a keypad, a post-it note, and a hint to walk around and find a lost book and a UV flashlight. 

after canvasing the map for the book and the flashlight, I used the flashlight on the keypad, which illuminated 4 digits:
2,6,7,8

after playing around for a bit, i noticed that the keypad could take 5 digits, opening the possibility of repetition for a single digit.
being the stubborn person that i am, however, i utilized ChatGPT to provide me a list of all combinations of 4 digit codes using 2,6,7,8, and then proceeded to brute force all the combinations. this was a complete waste of time, because the gut instinct i had about needing 5 digits was correct.

i spent a great deal of time checking the debugger for similar possibilities of injection, like we saw with the Elf Minder 9000. this too, unfortunately, was waste of time because i didn't see any way to manipulate the data.

on to the post-it note, which had groups of numbers: 2:6:1, 4:19:3, 6:1:1, 3:10:4, 14:8:3.
this screamed 'page/word/letter' to me, so off we hunt, using DevTools this time to more easily deal with the page changes, since the code for the book (and the static text for each page) was listed in the main.js for frost-y-book.com

the major hang-up i had at this stage was for 4:19:3, because on page 4, there is a hyphenated word "new-fallen", which i considered to be two words. this is incorrect. hyphenated words are considered single words. thus, i decoded the 19th word/3rd letter incorrectly as a 3, instead of 2. this led me to believe that possible the 3 was a hidden number, and i should be trying 2,3,6,7,8. the drawback to this, is that there is 120 different sets of unique digits (5! permutations), and as we all know... ain't nobody got time for that.

eventually, i had to reference SamanthaCtF's guide on [medium.com](https://samanthactf.medium.com/sans-holiday-hack-2024-act-1-5a8c859341d8) which made me realize my mistake about the 19th word. redoing my counts, i realized that the 2 was repeated, and going in order of the post-it note, reveals:


<details>
  <summary>Spoiler Alert</summary>
 72682 (SANTA using phone keypad substitution)
</details>




This grants you silver.

Please see next entry for my attempt at gold, which includes using a python script and a zip file full of pieces of an image, to reconstruct the image and produce the hidden code.

