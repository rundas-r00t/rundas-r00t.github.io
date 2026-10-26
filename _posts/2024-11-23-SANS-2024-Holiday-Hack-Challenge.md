---
layout: post
title: "SANS 2024 Holiday Hack Challenge"
author: "rundas-r00t"
date: 2024-11-23 12:00:00 -0000
categories: "ctf-writeups"
---

SANS Holiday Hack Challenge 2024 - Prologue

took two attempts at hacking the Elf Connect challenge. first attempt i just played through the front-end and did not beat the high score.

second attempt, i used the Developer Console in Firefox to look at the js code. i found this statement, which implies that i can inject a value into the 'score' variable:

let score = parseInt(sessionStorage.getItem('score') || '0'); //Initialize score


this line basically says that the value for the 'score' variable is stored in the session on the client's side, not the server's side. so we boot up the game, which will initialize the score to 0, and then going to the console and typing:

score = 1000000

should set the 'score' variable to 1000000.

unfortunately for me, this did not work. i double-checked my therory against [this writeup](https://samanthactf.medium.com/sans-holiday-hack-2024-prologue-dfa78d984b5c) and it looks like the author did the same thing i did, but it did not work and i'm not sure why. 

put a message out on the discord server to see if i'm missing something like 'use chrome instead of firefox'. i doubt it's that simple but you never know...


edit:
11/25/2024

i think i have it figured out. there is one crucial step i was missing with the Dev Tools window. i was trying to set the variable at the 'main' tree instead of at the 'elfconnect' tree. changing trees and redoing the steps above granted a successful hack!
