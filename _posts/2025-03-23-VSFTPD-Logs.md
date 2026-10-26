---
layout: post
title: "VSFTPD Logs"
author: "rundas-r00t"
date: 2025-03-23 12:00:00 -0000
categories: "log analysis"
---

# VSFTPD Logs


video walkthrough here: [https://rumble.com/v6r2zb2-rundas-gets-r00t-sunday-ctfs.html](https://rumble.com/v6r2zb2-rundas-gets-r00t-sunday-ctfs.html)


first let's make a new directory, cd to it, and download the log file:

`mkdir vsftpd; cd vsftpd` then `r-click-->save as` on the link and choose the directory.

let's start with some quick prelim investigation

`file vsftpd.log`

`exiftool vsftpd.log`


![image](https://github.com/user-attachments/assets/45e3a37a-bc4d-4105-880b-25cb29a1151f)


nothing super surprising here. let's continue with `head`

`head vsftpd.log`

![image](https://github.com/user-attachments/assets/94f69876-e2d5-43b9-a776-c11e9b8e7bdb)

Question 1: what is the IP address that 'ftpuser' use to first log-in from?

let's head and grep and see if we can easily answer this question:


<details markdown="1">
  <summary>SPOILER</summary>

`head vsftpd.log | grep 'ftpuser'`

![image](https://github.com/user-attachments/assets/54b2d246-5f76-49f0-9574-ab10806506ab)


</details>

so there we go! nice. 

Question 2 is "what is the first directory that ftpuser created? well, we can clearly see that in the above screenshot on line 5, so Question 2 is trivial.

Question 3 is What is the Last directory that ftpuser created? let's try `tail` and see what the bottom entry is:


<details markdown="1">
  <summary>SPOILER</summary>

`tail -n50 | grep -i 'mkdir' vsftpd.log`

![image](https://github.com/user-attachments/assets/d1bfb66c-0b5c-47c3-a95e-556070eb22ce)

</details>

easy peasy. 

Question 4: what file extension was the most used by ftpuser? so basically we're needing to isolate file extensions from the log, and then count the entries to see which extension appears most often. we'll need to use some `awk` and drop these into a new file to parse further. let's do that now.

<details markdown="1">
  <summary>SPOILER</summary>

counting the space delimitizations, let's try to isolate columns 12 and 13 to start

`awk '{print $12,$13}' vsftpd.log`

![image](https://github.com/user-attachments/assets/b73943bd-2741-411c-8a28-8236d7ce042a)

ok looks like 13 is the column we need, so let's do this again and drop it into a file:

`awk '{print $13}' vsftpd.log > files.txt; head files.txt`

![image](https://github.com/user-attachments/assets/727177da-9aff-4cd2-826e-800cacba0d92)

</details>

we see alot of blanks due to the formatting of the log, since some lines don't have a 13th column. i'm sure there's a handy way to clean this up quickly from the command line, but i just ended up going through the file via `gedit` and manually removed the empty lines, until i realized i could use Excel and filter on blanks and easily remove the blank rows & then copy/paste back into a new txt file. since i have LibreOffice, i don't have a 'filter on blanks' feature, so i ended up sorting the lines (which is not best practice since we're changing the order) and copy/pasting back into the txt file. I got lucky on this that we're just using the file for a count and not needing it for anything else. this is partially why i copy/pasted into a new txt file `cleanfiles.txt`, in case i need to reference the original order later. (and then 5 minutes later i changed my mind about this and overwrote the original `files.txt` file anyway lol oh boy...)

now that we have column 13 cleaned up, let's double-check our work: 

`head cleanfiles.txt`

![image](https://github.com/user-attachments/assets/e16b409b-5c69-4592-b5d8-c2e780fab4b0)


looks good. now let's work on getting some totals on these file extensions.

<details markdown="1">
  <summary>SPOILER</summary>

`grep '\.' cleanfiles.txt > files.txt; rm cleanfiles.txt; head files.txt` (overwriting `files.txt` since it doesn't appear that we need the original anymore)

![image](https://github.com/user-attachments/assets/5e701f79-e3ec-4ae6-bddd-3298de68303e)

now we need to `awk` this with a delimiter of `/` so we can remove the folder path and isolate the actual file name and extension:

`awk -F'/' '{print $5}' files.txt | head`

![image](https://github.com/user-attachments/assets/160ad7ae-af7c-4f3f-ab2f-f1d3e6c20f8f)


</details>

ok now that we're happy with the results, let's drop this into a file to delimit further to isolate the file extensions.


<details markdown="1">
  <summary>SPOILER</summary>

`awk -F'/' '{print $5}' files.txt > cleanfiles.txt | head cleanfiles.txt`


![image](https://github.com/user-attachments/assets/a8cf30bc-3f4a-4115-9437-927a510272d1)

now we can delimit again on '.' and drop all of these into an `extensions.txt` file:

`awk -F'.' '{print $2}' cleanfiles.txt > extensions.txt; head extensions.txt`

![image](https://github.com/user-attachments/assets/ea27256f-e379-4c1b-b529-2d2f82522600)

</details>

now we just need to `cat` extensions and `sort -u` to get a list of the totals. but we don't want just the numbers, we want the output to be something like:

`JPG: 16`

`xml: 100`

etc..

so let's use some pipes, `echo` and and a bash loop to see if we can make the output look nice:

<details markdown="1">
  <summary>SPOILER</summary>

first, let's get a unique list of extensions for which we can iterate on with the bash loop:

`cat extensions.txt | sort -u > uniques.txt` and open up this file in `gedit` or `mousepad` and manually remove the new-line characters and make them space delimited.

![image](https://github.com/user-attachments/assets/4bba24b1-a0c8-402f-8312-bf1e77bc6bbc)

![image](https://github.com/user-attachments/assets/f4b9d65b-7578-4163-9a77-0b3527c446a8)

final result:

![image](https://github.com/user-attachments/assets/cafd40d1-3756-40fe-881b-b7ab2503c0b9)

</details>


i'm sure there's a way to do all this in a single command, but for now, we have the end result we need, so let's continue with creating our loop. copying the space-delimited file extensions to our clip board, let's make the bash loop:

<details markdown="1">
  <summary>SPOILER</summary>

`for ext in bak0 chm css DE exe JPG pdf php png txt xml; do echo -n "$ext"; grep "$ext" extensions.txt | wc -l; done'

![image](https://github.com/user-attachments/assets/b5084c78-efa3-41cf-b31d-edfcce76878a)

now let's drop it into a file:

`for ext in bak0 chm css DE exe JPG pdf php png txt xml; do echo -n "$ext"; grep "$ext" extensions.txt | wc -l; done > extcounts.txt; cat extcounts.txt`

![image](https://github.com/user-attachments/assets/58b67720-e87b-46ae-ab80-18886ca8345f)


</details>

so now we can easily answer Question 4. 

Question 5 asks to find another user in the log. in the video, i originally just tried grepping the file to see if we could see any other users in the `/home/user/` file directory list on column 13. this is very messy and you'll be scrolling for days.. so let's go back to the original file and see if we can `awk` and `sort -u` on column 8, which has syntax of `[username]` :


<details markdown="1">
  <summary>SPOILER</summary>

`awk '{print $8}' vsftpd.log | sort -u`

in the video, i used a more complex command, but the above will work and is shorter, so it's better than what i originally used.

![image](https://github.com/user-attachments/assets/b2a8707a-92f8-4258-b57a-8ea2f84b38ed)


Question 5 is good to go!
</details>

On to Question 6: what IP address did this other user log-in from? i bet we can answer this with `grep`:


<details markdown="1">
  <summary>SPOILER</summary>

`cat vsftpd.log | grep 'jimmy' | grep -i 'login'`

![image](https://github.com/user-attachments/assets/06021aa3-2f2b-4334-a5a9-bfd79e2ce56d)

in the video, i used `cat vsftpd.log| grep 'jimmy' | awk '{print $8,$9,$10,$11,$12}'|sort -u` : 

![image](https://github.com/user-attachments/assets/10520564-a6c2-454b-a712-9d830cd45d6d)

i arrived at the proper answer, but grepping twice ends up being much cleaner than what i did originally.

</details>

Questions 7: how many total bytes did the other user upload?


<details markdown="1">
  <summary>SPOILER</summary>

we know the other user is `jimmy` and we're looking for keyword `upload` so we'll likely need to grep on these two and then `awk` to find the column of bytes. let's do it.

`cat vsftpd.log | grep 'jimmy' | grep -i 'upload' | grep 'bytes' | awk -F',' '{print $3}' | awk '{print $1}' > byteslist.txt `


NOTE: in the video, i did `cat vsfptd.log | grep 'jimmy' | grep 'bytes' | awk -F',' '{print $3}'`, which will give us the total bytes of UPLOAD and DOWNLOAD, (and then subsequently isolated DOWNLOAD by mistake) which is not what we need. you can see how i problem-solve through these mistakes from timestamp 44:00 - 51:00.

now that we have a list of bytes, we can utilize ChatGPT to give us a bash command to sum each line and give us a grand total:

![image](https://github.com/user-attachments/assets/8eae8caf-7054-4c99-88de-b5c7bd94308f)


yoooo this is awesome! `awk` can sum! lets do it:

`awk '{sum+=$1} END {print sum}' byteslist.txt > uploadedbytesbyjimmy.txt; cat uploadedbytesbyjimmy.txt`  

let's break down this command: take the variable `sum`. let `sum` = 0, then add it to the first value in column 1 (sum = 0 + [1st value]). then, sum = sum + [2nd value], and continue this process for each line, then print the final result.

![image](https://github.com/user-attachments/assets/35b349d9-078d-4ea5-9a9a-dc25dece6845)

</details>

Question 8: how many total bytes did 'ftpuser' upload?

this is a similar process to Q7. we just need to replace the other username with 'ftpuser' in our commands. let's do that now:


<details markdown="1">
  <summary>SPOILER</summary>

`cat vsftpd.log | grep 'ftpuser' | grep -i 'upload' | grep 'bytes' | awk -F',' '{print $3}' | awk '{print $1}' > uploadedbytesbyftpuser.txt; cat uploadedbytesbyftpuser.txt `

`awk '{sum+=$1} END {print sum}' uploadedbytesbyftpuser.txt > ftpusertotalupload.txt; cat ftpusertotalupload.txt `

![image](https://github.com/user-attachments/assets/694c0bad-7527-4b22-ae8e-1019e152634e)

i did some slightly different file-names in the video, but the end result is the same.



</details>

Question 9: how many total bytes did ftpuser download?

again, very similar set of commands here as with Q7 and Q8.


<details markdown="1">
  <summary>SPOILER</summary>


`cat vsftpd.log | grep 'ftpuser' | grep -i 'download' | grep 'bytes' | awk -F',' '{print $3}' | awk '{print $1}' > downloadedbytesbyftpuser.txt; cat downloadedbytesbyftpuser.txt `

`awk '{sum+=$1} END {print sum}' downloadedbytesbyftpuser.txt > ftpusertotaldownload.txt; cat ftpusertotaldownload.txt `

![image](https://github.com/user-attachments/assets/6d2b547a-f8e2-40e2-8213-7878e75d16ca)


</details>


Question 10: identify the suspicious IP address.

back to the original file, let's just `grep -i 'login'` (or `grep 'LOGIN'`) and see what we get.

<details markdown="1">
  <summary>SPOILER</summary>

`cat vsftpd.log | grep 'LOGIN'|sort -u`

![image](https://github.com/user-attachments/assets/2014cd4a-1c6e-4cc2-9e22-9e7bb51228be)

it took me a minute to think through this. i tried checking for unique `pid` numbers, then checked for other users using `awk`, then tried `grep '\[username\]'` on the different users to try to find a login without any activity.

finally, after a few minutes & talking through it, i re-grep'd on `LOGIN` and noticed the bottom line:

![image](https://github.com/user-attachments/assets/45d0ccfb-d7ed-4d1c-8836-45382fae1f2a)


grepping on this IP address confirms the answer:


`cat vsftpd.log | grep '"10.3.0.6"'` 

**NOTE:** i put the IP in double-quotes in order to isolate the IP address because, in this file, the syntax is `"IPADDRESS"`. also, without the double-quotes, you will get a few extra lines of `Kbyte/sec` information, which we don't need.

![image](https://github.com/user-attachments/assets/8d0e74a8-ac0a-442d-ab82-3da69f40dbfc)

we've confirmed that 10.3.0.6 has a log-in and no other activity

</details>

VSFTPD Logs? DONE!



**BONUS** awk to get just the IP address and drop it into a file:


<details markdown="1">
  <summary>SPOILER</summary>

`cat vsftpd.log | grep '"10.3.0.6"' | awk '{print $12}' > suspiciousIP.txt; cat suspiciousIP.txt`


![image](https://github.com/user-attachments/assets/f0fa907a-995e-425f-b76d-b288f75511a2)

![image](https://github.com/user-attachments/assets/a78c2116-7c4c-4241-8467-d3b03094d98d)


to get rid of the quotes, simply do this `awk` command:

`awk -F'"' '{print $2}' suspiciousIP.txt > susnoquotes.txt; cat susnoquotes.txt`


![image](https://github.com/user-attachments/assets/67499de9-57df-4dca-986f-5d8dc4ff6b59)

</details>
