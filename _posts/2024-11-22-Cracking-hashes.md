---
layout: post
title: "Cracking Hashes"
author: "rundas-r00t"
date: 2024-11-22 12:00:00 -0000
categories: ctf writeups
---


11/22/2024 
found a post on /r/HashCracking where a professor challenged a student to crack a password-protected pdf. the hash was already extracted by the student using pdf2john:

$pdf$4\*4\*128\*-1060\*1\*16\*6fd3986b540246489d761922111fe762\*32\*5ca84958ddc136d19b9950b72f4c927a00000000000000000000000000000000\*32\*a93df17be9f3f4478dab6d9ce567a8364dc4dab1e15f0884885f09893439abc7


inside my kali VM, i used john with weakpass.com's top2billion common password lists. after a few hours and 100% complete, the hash was not cracked.
2nd attempt using the all_in_one wordlist outside of my VM to utilize my GPU to crunch the hashes. caching 28,438,933,564 potential passwords took about 90 minutes. currently running through the file to find a match. will update once complete. doubtful we'll get a hit b/c i'm betting the professor didn't use a weak password on the file, but we'll see how it turns out.

11/23/2024

could not crack the password, even with the all_in_one wordlist. the student reported back and said that the professor used a 30-word passphrase, which would take an eternity to crack. so yeah, good exercise using john and hashcat, but ultimately futile.
