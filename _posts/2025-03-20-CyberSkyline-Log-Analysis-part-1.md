---
layout: post
title: "Cyberskyline Log Analysis (part 1)"
author: "rundas-r00t"
date: 2025-03-20 12:00:00 -0000
categories: "ctf-writeups"
---

# SSH Log Analysis

The first challenge is to parse a file `auth.log` and answer these questions:

1. `what is the name of the compromised ssh server?`

1. `what was the first IP address to attack the server?`

1. `what is the second IP address to attack the server?`

1. `what is the third IP address to attack the server?`

1. `what user was targeted during the attack?`

1. `which attacker IP address was used to successfully log-in?`

to start, let's make a new directory and download the log-file to it:

`mkdir log-analysis; cd log-analysis`

![image](https://github.com/user-attachments/assets/227a5fed-8058-462c-8e23-5f54609e1044)


(to download the log-file, `r-click--> save as ` and select the proper folder.)

let's check to see exactly how big this file is by using `wc -l auth.log`:

![image](https://github.com/user-attachments/assets/701bd8de-072f-4f9f-9a41-ba06f2fd421a)



so it's not insanely gigantic, which is nice. my current knowledge for parsing logs would be to take a pcap file, create a zeek log from it, and import that into RITA and have RITA crunch it away. unfortunately it doesn't look like we were given a zeek log, since zeek is tab-delimited and this log-file seems to be inconsistently space-delimited.

luckily, it looks like we can atleast answer the first question by just `cat`'ing the file:


<details markdown="1">
  <summary>Click to reveal</summary>


`cat auth.log`

![image](https://github.com/user-attachments/assets/ecb851e5-446b-4887-9055-deddae9912f7)


</details>

in these first few lines, we see the answer to the first couple of questions, actually. question 1 is column 4, the name of the server. 
question 2 is the IP address associated with the first failed password attempt we see in the log. question 3 is the 2nd run of failed password attempts, but with a new IP address. likewise for question 4. 

we can easily see these iterations by first `awk`'ing columns 6 through 14 into it's own file (stripping the timestamps) and then grepping for `"Failed password"` and piping to `sort -u` to get a unique list

<details markdown="1">
  <summary>Click to reveal</summary>

`awk '{print $6,$7,$8,$9,$10,$11,$12,$13,$14}' auth.log | grep -i "failed" > failures.txt `

`cat failures.txt`

![image](https://github.com/user-attachments/assets/602eb2d3-d45e-4351-995c-7686bbb0860f)


</details>

now that we've stripped the timestamps, we can `sort -u` to get a list of unique IP addresses that are attempting to connect:

<details markdown="1">
  <summary>Click to reveal</summary>

`cat failures.txt | sort -u`

![image](https://github.com/user-attachments/assets/917ac73a-c524-4b10-bf3c-0179f8b7e4b6)



</details>

now we can easily answer questions 2, 3, and 4. 

**NOTE**: when i `sort -u`'ed this list, the output was reordered, and i'm not sure why. my output ended up being IP1, IP3, and IP2. so be sure to compare this list to the original file so that you answer the questions in chronological order.


for Question 5, this should be obvious as it's the username associated with the failed password attempts:

<details markdown="1">
  <summary>Click to reveal</summary>

`cat auth.log | grep "Failed password"`

![image](https://github.com/user-attachments/assets/56125ea2-e457-45ca-8bd3-a1e61fbdca3a)



</details>


The final question asks for the successful log-in attempt. so let's do a reverse grep by using `grep -v` to exclude `"Failed password"` any any other extraneous lines we don't need:



<details markdown="1">
  <summary>Click to reveal</summary>

`cat auth.log | grep -v "Failed password"| grep -v "Received disconnect"|grep -v "Connection"` 

**NOTE:** i ran a few iterations of this to narrow it down to just a handful of lines, each iteration adding a new `| grep -v ""` to remove more data from the results.

![image](https://github.com/user-attachments/assets/13eae033-a783-4d2b-8a1e-120eb0c49f66)


and there we go! that one line near the bottom is exactly what we're looking for.

just to cover our bases, let's `grep "Accepted"` and make sure it's the only successful log-in:

![image](https://github.com/user-attachments/assets/c0a8eb00-06cb-4b52-aa6a-e69d9af32311)

</details>

Perfect. SSH Log Analysis? DONE!
