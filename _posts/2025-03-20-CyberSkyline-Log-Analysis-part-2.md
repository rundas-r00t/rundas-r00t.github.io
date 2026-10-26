---
layout: post
title: "Cyberskyline Log Analysis (part 2)"
author: "rundas-r00t"
date: 2025-03-20 12:00:00 -0000
categories: "ctf writeups"
---

# NginX Log Analysis

we are given an nginx log file `access.log` and are tasked to answer these questions:


1. How many different IP addresses reached the server?
2. How many requests yielded a 200 status?
3. How many requests yielded a 400 status?
4. What IP address rang at the doorbell?
5. What version of the Googlebot visited the website?
6. Which IP address attempted to exploit the shellshock vulnerability?
7. What was the most popular version of Firefox used for browsing the website?
8. What is the most common HTTP method used?
9. What is the second most common HTTP method used?
10. How many requests were for \x04\x01\x00P\xC6\xCE\x0Eu0\x00? 	

first, let's download the given file and do some prelim investigation:

<details markdown="1">
  <summary>Click to reveal</summary>

**NOTE:** i renamed the file to `nginxaccess.log` so that it's more specific so i don't accidentally use a different file in the `log-analysis` folder.


`head nginxaccess.log`

![image](https://github.com/user-attachments/assets/c8eba1eb-653f-4d14-914f-e4504e6d1ab0)


`wc -l nginxaccess.log`

![image](https://github.com/user-attachments/assets/ffd3d28a-bd34-4834-873d-7a4d0340bb50)



</details>


ok so it's not too big. the first question asks about the number of different IP addresses. let's use `awk` and `sort -u` to see if we can get a unique list of IP's.

<details markdown="1">
  <summary>Click to reveal</summary>


`awk '{print $1}' nginxaccess.log | sort -u > IPs.txt; cat IPs.txt  `

![image](https://github.com/user-attachments/assets/546ce2a4-c855-4f53-b50b-6f94fb766be6)


how may unique's do we have?

![image](https://github.com/user-attachments/assets/41c8d633-92a4-4352-b805-2fdb25496f76)


</details>

Nice.

next we need to look for Status 200's. sounds simple enough. let's try to `grep` those into a new file:



<details markdown="1">
  <summary>Click to reveal</summary>

`grep "200" nginxaccess.log > 200s.txt; wc -l 200s.txt`

![image](https://github.com/user-attachments/assets/a8a94df4-860a-4606-bf73-865db4821549)

</details>

Nice again!

the next question is basically the same; this time looking for Status 400:

<details markdown="1">
  <summary>Click to reveal</summary>

we need to adjust the prior command a bit to ensure we're not looking at time zone information, namely the `-0400` as seen here:

![image](https://github.com/user-attachments/assets/f73701db-1907-421b-8dcb-a57774f62e6f)

in order to avoid this, we can use `awk` instead of `grep`. `grep` alone will include some Status 200s and Status 502s, etc. the status codes exist in the 9th column, so let's try:

`awk '{print $9}' nginxaccess.log| grep "400" > 400s.txt; cat 400s.txt`

![image](https://github.com/user-attachments/assets/b6051dc7-e925-48b7-8577-c4890737f239)

and the total is:

`wc -l 400s.txt`

![image](https://github.com/user-attachments/assets/c742bbd3-80c5-4502-9e8d-3a012516cc36)

unfortunately, because of how `awk` interprets "", we are missing some of the Status 400's. One example being the 5th line of the file:

![image](https://github.com/user-attachments/assets/74e545f5-2e5e-4130-b64e-e7c5c2576afc)

because it has `""`, from `awk`'s point of view, it should return `0` instead of `400`. so let's turn to ChatGPT and see if it can help.

![image](https://github.com/user-attachments/assets/27401406-9593-4efe-891b-1df8a44d7e6e)

this looks like what we need. let's try it.

`awk -F'"' '{split($3, a, " "); if (a[1] == 400) print $0}' nginxaccess.log`

![image](https://github.com/user-attachments/assets/bb1de7a3-c5f6-48e4-9b0b-25412f2998a7)

well, it's not very clean, but upon visual inspection, it did grab the `"" 400`-type lines, so let's drop it in a file and do a line-count on it:

`awk -F'"' '{split($3, a, " "); if (a[1] == 400) print $0}' nginxaccess.log > 400s.txt; wc -l 400s.txt`


![image](https://github.com/user-attachments/assets/451ca5f1-e32b-479c-ab84-0268ce88b61c)


</details>

there we go! we finally got it!

the next question is "What IP address rang at the doorbell?". i have no idea what this means, but thankfully we have the power of `grep`!


<details markdown="1">
  <summary>Click to reveal</summary>

`grep -i "door" nginxaccess.log`

![image](https://github.com/user-attachments/assets/d2b29f66-9db8-40ed-821d-037b6e228fb9)

nothing. ok let's try `bell`.

`grep -i "bell" nginxaccess.log`

![image](https://github.com/user-attachments/assets/e1b31564-c378-4f01-8356-8c6414ee3057)


</details>

ok! we got it! awesome!

let's continue our awesome grepping skills to try to answer the next question:

<details markdown="1">
  <summary>Click to reveal</summary>

`grep -i "googlebot" nginxaccess.logs`

![image](https://github.com/user-attachments/assets/59f2d369-1747-424b-bd82-7431fe0d9caa)

</details>

Nice!

for this next question, i must admit i am not familiar with the "shellshock" vulnerability, but i'm guessing it's a similar payload to `nc -l 8080 -e /bin/sh`, so let's look for `/bin` and see what we can see:


<details markdown="1">
  <summary>Click to reveal</summary>

`cat nginxaccess.log | grep -i "bin"`

![image](https://github.com/user-attachments/assets/202954a7-7ec6-4cf1-a965-16334e02831f)


ok looks like we found something!

let's refine it to `cat nginxaccess.log | grep -i "bin/bash" `

![image](https://github.com/user-attachments/assets/64677689-0f1a-40a5-ba82-0e603400e96c)

and then let's drop this into a file and `awk` the first column to see how many IP addresses we have that are sending this payload.

`cat nginxaccess.log | grep -i "bin/bash" | awk '{print $1}' | sort -u > evilIP.txt; cat evilIP.txt`

![image](https://github.com/user-attachments/assets/a6704ea7-946e-4496-a4a8-74b4290b37db)

</details>

perfect! exactly what we were hoping to see.

the next question asks about firefox, so let's grep again and see what we get

<details markdown="1">
  <summary>Click to reveal</summary>

`grep -i "firefox" nginxaccess.log`

![image](https://github.com/user-attachments/assets/4cf420c7-8da2-4835-b3fd-bcc93e311ddb)

</details>

We have quite a few options to choose from. so let's `awk` again and see if we can isolate that column:

<details markdown="1">
  <summary>Click to reveal</summary>

`grep -i "firefox" nginxaccess.log | awk '{print $18,$19,$20,$21}'`

![image](https://github.com/user-attachments/assets/aed24b1a-614a-4061-95d1-cf14ed460c51)

again, it's messy but now we can easily see that v31.0 appears most often

</details>


the next question asks for the most common HTTP method, aka "HTTP request type". ok so we're looking for `GET`, `POST`, etc.
so now, (you guessed it!) let's `grep`:


<details markdown="1">
  <summary>Click to reveal</summary>


this time, let's use a bash loop and just drop each request type into it's own file so we can easily count each type.

`for type in GET POST HEAD PUT DELETE OPTIONS CONNECT TRACE; do grep "$type" nginxaccess.log > "req${type}.txt"; done`

![image](https://github.com/user-attachments/assets/58c26cdc-fd87-4411-88dd-6910651e4717)

![image](https://github.com/user-attachments/assets/a2f4efb9-1ed4-4b64-aa55-7ad9492ccac7)

now we can `wc -l` each file to see which has the most requests (with a bash loop of course):

`for file in reqCONNECT.txt reqDELETE.txt reqGET.txt reqHEAD.txt reqOPTIONS.txt reqPOST.txt reqPUT.txt reqTRACE.txt; do wc -l $file; done`


![image](https://github.com/user-attachments/assets/bbb5dcd7-a21f-4921-87f4-43f77f319937)

so now we can easily see which type of request occurs the most, and, which is in 2nd place, answering questions 8 and 9.



</details>



and finally, for the last request, we're just going to `grep` the syntax of characters in the question, being sure to double-backslash b/c `\` is a shell meta character that needs to be escaped:

<details markdown="1">
  <summary>Click to reveal</summary>

** NOTE: ** up until this point, we have been using "" with grep to find specific strings. best practice is to use single quotes ' '. Use double quotes (") when you want Bash to evaluate variables. Use single quotes (') when you want exact text, no variable substitution.

keep this in mind if you get no results for a phrase that you know exists; like this:

![image](https://github.com/user-attachments/assets/07a1c5f8-6ccd-4d7d-83a5-adec069378de)

we get no results even though the question tells us that the string exists. so let's try again with single quotes:

`grep '\\x04\\x01\\x00P\\xC6\\xCE\\x0Eu0\\x00' nginxaccess.log`

![image](https://github.com/user-attachments/assets/51a0f5a7-333e-4e87-be20-6b178a88f1e4)

there we go. now since it's asking for a total count again, let's drop it into a file and count the lines:

`grep '\\x04\\x01\\x00P\\xC6\\xCE\\x0Eu0\\x00' nginxaccess.log > oddrequests.txt; wc -l oddrequests.txt`

![image](https://github.com/user-attachments/assets/6890c48b-9e4a-459d-8c1a-d6cd83390046)


</details>

And there we have it. NginX logs? DONE
