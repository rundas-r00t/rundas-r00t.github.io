---
layout: post
title: "Cyberskyline Ctf'S (Part 3)"
date: 2025-03-18 12:00:00 -0000
categories: ctf writeups
---

# Memory Analysis with Volatility

from the last entry, we were able to answer 3 of the 6 questions for the "The Book" Forensics challenge using `strings` and `grep` on the memory dump file `memdump.mem` . offline, i was able to get Volatility3 installed into a virtual environment, by creating a new environment variable stored in `.zshrc`, as well as using that variable to run a custom bash command `vol` so that i can run the commands easily from the command-line without needing to fire up a `venv` . if you need instructions on how to do that, please look at the prior post  [found here](https://github.com/rundas-r00t/blog/wiki/20250317-%E2%80%90-CyberSkyline-CtF's-(part-2)).

from the prior challenge, we've learned:

<details>
  <summary>Click to reveal</summary>

*  the type of OS: win_NT

*  the computer name: DESKTOP-OT97GG3

*  the attacker username: liber8hacker


</details>

the 3 remaining questions are: 

`what is the full filepath and file of the file in interest?` 

`what is the real name of 'cloud'?` 

`what is the password of the currently logged-in user?`

before we run Volatility on the `memdump.mem` file, let's `mkdir` a folder and cd to it:

` mkdir vol-files; cd vol-files `

now that we have a usable installation of Volatility, let's try a handy-dandy bash loop that will give us a bunch of files of different plugins:

 **[~/../memdump/vol-files]$**   `   for plugin in windows.netscan.NetScan windows.pstree.PsTree windows.pslist.PsList windows.cmdline.CmdLine windows.filescan.FileScan windows.dlllist.DllList; do vol -q -f ../memdump.mem $plugin > $plugin.dump.txt; done`

this will take a few minutes to run, as the file is 1GB. if the file has hundreds of GB, it will take quite a long time to parse all the data. But, once complete, you should have files that look like this:

![image](https://github.com/user-attachments/assets/7257e207-8bb6-431a-89ea-8938e0b6a1c6)

now let's go back to grepping through all the files. it is useful to know some basic info prior to doing this. for example, knowing an evil IP or and evil processname definitely helps to save time. since we have neither, let's start with `head` and  `tail` and see what we can see:

<details>
  <summary>Click to reveal</summary>

![image](https://github.com/user-attachments/assets/4407953f-6967-498e-9beb-52eb95304ef8)


</details>

basically what we're seeing is all the commandline commands that started the running processes on the machine. the only thing that jumps out to me is `DB Browser for SQLite.exe` , which is good news. let's look at some other files and see if we can find a SQL database somewhere. we don't know which processes are evil just yet, so let's skip the DllList file and move on to the FileScan file. we want to use `cat`, `grep`, and the evil username to narrow down our results: 


<details>
  <summary>Click to reveal</summary>

`cat windows.filescan.FileScan.dump.txt | grep -i "liber8hacker" `


![image](https://github.com/user-attachments/assets/61cd72fb-d856-4160-a4d9-a314451504c8)



</details>

we have quite a few hits in the AppData folder for the attacker account. a few stand-out files to me are `black_book.db` (the SQL database we were hoping to find), `ntuser.dat.LOG2` , and `ntuser.dat.LOG1`. let's add the `sort -u` to further narrow down the results.

<details>
  <summary>Click to reveal</summary>

`cat windows.filescan.FileScan.dump.txt | grep -i "liber8hacker" | grep -i "desktop" | sort -u`

![image](https://github.com/user-attachments/assets/674ecc9e-8ea5-49ad-a35e-88f11b264b9c)

`cat windows.filescan.FileScan.dump.txt | grep -i "liber8hacker" | grep -i "ntuser.dat" | sort -u`

![image](https://github.com/user-attachments/assets/e465b505-f2f6-4f53-923d-1b5cbef7634f)


</details>


this is pretty good info, but what i really want to do is to extract that `black_book.db` file and work some SQL magic. to do that, let's `cat` , `grep` and `sort -u` and make note of the memory addresses:

<details>
  <summary>Click to reveal</summary>

`cat windows.filescan.FileScan.dump.txt | grep -i "black_book.db" | sort -u`


![image](https://github.com/user-attachments/assets/ca313cdb-164f-4ea9-a6f4-0881b887444a)

**Note:** this is the answer to question 4. Be sure to add `C:` to the beginning of the file path to fulfill the "full filepath" requirement of the question.


</details>

next, let's do another bash loop with the memory addresses and see what we can extract:

<details>
  <summary>Click to reveal</summary>

`mkdir output`

`for addr in 0xe0003d624b90 0xe0003e836f20 0xe0003f5a9d00 0xe0003f861960; do vol -f ../memdump.mem --output-dir output/ windows.dumpfiles --virtaddr $addr; done`


**full disclosure**: this command tripped me up because of the order of the command. some Volatility plugins have extraction commands that are specific to the plugin, while others utilize a global extract command. if you get some errors with trying to extract files from memory addresses, coordinate with ChatGPT to easily try different orders of the commands until you get something that's error-free.

![image](https://github.com/user-attachments/assets/ef9bd35b-fd9e-408c-9032-39f713482525)

![image](https://github.com/user-attachments/assets/98b40953-a45c-46d9-a492-1ac5cc3fde42)


</details>

now let's load up one up in `sqlite3` and see what we can see. i've chosen the `SharedCacheMap` file on whim, but my gut (and the file size) says that we can possibly get more info out of this type of file versus the others:

<details>
  <summary>Click to reveal</summary>

`sqlite3 file.0xe0003d624b90.0xe0003f6d25a0.SharedCacheMap.black_book.db.vacb `


![image](https://github.com/user-attachments/assets/249623e4-32f7-480e-ab5f-73280d44f2d1)

`sqlite> .tables`

aliases  book   

`sqlite> PRAGMA table_info(aliases);`

0|id|INTEGER|1||1

1|alias|TEXT|1||0

`sqlite> select * from aliases LIMIT 20;`

1|grace

2|wasp

3|ace

4|torpedo

5|diablo

6|liber8

7|alpha

8|cloud

9|indie

10|nobody


`sqlite> PRAGMA table_info(book);`

0|id|INTEGER|0||1

1|first_name|TEXT|1||0

2|last_name|TEXT|1||0


`sqlite> SELECT * FROM book LIMIT 20;`

1|robert|sturm

2|laura|replogle

3|delmar|couch

4|troy|enriquez

5|leslie|mullis

6|tom|hess

7|steve|scott

8|gloria|hampton

9|barbara|walters

10|johanna|weiland

from here we can answer the 5th question, "what is the real name of "cloud"?", by doing a nice join:

`SELECT book.id, book.first_name, book.last_name, aliases.alias FROM book LEFT JOIN aliases ON book.id = aliases.id; `

![image](https://github.com/user-attachments/assets/cdfd1521-7284-437c-8eb4-59de6688508a)

random thought: this looks like it could be a list of customers, employees, or targets. this info should be included in the final report to a client.

</details>

On to the last question.....

one thing we didn't look at yet with our original `vol` bash loop is password hashes. let's do that now:

<details>
  <summary>Click to reveal</summary>

(exit sqlite by pressing ^C)

`cd ..`

`vol -q -f ../memdump.mem windows.hashdump.Hashdump > hashes.txt`

`cat hashes.txt`

![image](https://github.com/user-attachments/assets/bc0b0c27-4c9d-49ee-9972-54dd95ca4f5c)


</details>

as you can see, we need to clean up the file a bit in order to have hashcat process it properly. since we only have one user that we're interested in, we can clean this up fairly quickly using `grep` and `awk`

`grep "liber" hashes.txt | awk '{print $1 ":" $3 ":" $4}' > crackthis.txt `

<details>
  <summary>Click to reveal</summary>

![image](https://github.com/user-attachments/assets/58cef7ed-b542-4aad-a4cd-2d576a02fb56)



</details>

since we know this is a Windows_NT machine, hashes come in the form of `user:lanmanhash:NThash`. therefore, it's likely that we're going to use hashcat's NT Hash mode `-m1000`. so now, let's crack:

<details>
  <summary>Click to reveal</summary>

`hashcat -a0 -m1000 --user crackthis.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule`

![image](https://github.com/user-attachments/assets/79c6aea1-bd54-4c5b-a4de-dcfc232538ac)


this throws a token-length exception error, and i'm not sure why. i poked around with a few different options and couldn't get around the token-length exception. 



</details>

since hashcat doesn't seem to like our file, let's try John. first, let's isolate the hash into it's own file:

`cat crackthis.txt | awk -F: '{print $3}' > 1crack.txt`

and now let's `john` it:


<details>
  <summary>Click to reveal</summary>

`john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt 1crack.txt`

![image](https://github.com/user-attachments/assets/9081a8dd-617b-4211-83c8-d78c9aecbe53)




`john --format=NT 1crack.txt --show   `

![image](https://github.com/user-attachments/assets/33df69a9-f85c-4e1b-93cb-df4816d1a819)





</details>


and there we go! that's the answer to the last question. "The Book"? DONE!
