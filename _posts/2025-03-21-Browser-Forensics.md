---
layout: post
title: "Browser Forensics"
author: "rundas-r00t"
date: 2025-03-21 12:00:00 -0000
categories: ctf writeups
---

# Firefox Browser Forensics with SQLite

given a `browser.sqlite` file, answer the following questions:

1. What did the user search for on craigslist? 
2. What was the current price of bitcoin when the user was browsing? 
3. What Bitcoin exchange did the user log in to? 
4. What is the email that was used to log into the exchange? 
5. What was the ID of the Bitcoin transaction that the user looked at? 
6. What was the total BTC of all the inputs of the Bitcoin transaction?
7. Which bitcoin address received the majority of the Bitcoin in the transaction?


before getting too deep into `sqlite3`, lets see if we can `strings | grep` and answer some of these questions. First, let's try:

<details markdown="1">
  <summary>Click to reveal</summary>

`strings browser.sqlite|grep 'craigslist'`

![image](https://github.com/user-attachments/assets/57cc59b2-fd93-44fc-89bb-4b80013a6288)

with a keen eye, we see the `&query=bitcoin` part of the URL in the 2nd line and in the 6th from the bottom. to confirm, let's redo our grep statement:

`strings browser.sqlite|grep 'craigslist|grep 'query'  `

![image](https://github.com/user-attachments/assets/0d2a4ec1-dc9d-46e1-930d-6255cb7f2290)

looks good.

</details>

for the second question, let's continue our grepping skills, this time using a `strings|grep|grep` combo again.

<details markdown="1">
  <summary>Click to reveal</summary>

`strings browser.sqlite|grep 'bitcoin'|grep '\$'` --being sure to escape the `$` with a `\`.

![image](https://github.com/user-attachments/assets/c44bab66-e37e-46b4-b9a3-45eb8fe5de64)

there we go!

</details>

For the 3rd question, let's just look at all the `https` entries and see what we can see:

<details markdown="1">
  <summary>Click to reveal</summary>

`strings browser.sqlite|grep 'https'`

![image](https://github.com/user-attachments/assets/dc91486d-2ca4-4b07-8b0d-6f50a79c03c9)

Coinbase is a fairly large crypto-exchange, and we can tell by the `.com/accounts/528cf016....` web address that the user logged in and viewed his/her wallet.

</details>

we're doing great so far, and haven't even needed `sqlite3` just yet. amazing!

looking further through our current results, we see some hits for `mail.google.com`, so let's grep for `gmail.com` and see what we get:

<details markdown="1">
  <summary>Click to reveal</summary>

`strings browser.sqlite|grep 'https'|grep 'gmail'`

![image](https://github.com/user-attachments/assets/316f532e-185f-4297-853a-4ad9cf26c645)


</details>

ok looks good.

keeping this trend going, let's answer the next question by `grep 'transaction'`:

<details markdown="1">
  <summary>Click to reveal</summary>

`strings browser.sqlite|grep 'transactions'`

![image](https://github.com/user-attachments/assets/3b2e49f1-628c-475b-b7b2-65ef55b07e20)


</details>

Nice.

for the next question, let's finally utilize `sqlite3` and start poking around some tables.

<details markdown="1">
  <summary>Click to reveal</summary>

`sqlite3 browser.sqlite`

sqlite> `.tables`

sqlite> `select * from moz_inputhistory;`

sqlite> `select * from moz_historyvisits;`

![image](https://github.com/user-attachments/assets/8fa67139-436f-4b4a-88c6-0958270c468d)

not exactly sure what i'm looking at here... let's amke this a little more legible:

sqlite> `.headers ON`

sqlite> `.mode column`


sqlite> `select * from moz_historyvisits;`


![image](https://github.com/user-attachments/assets/1009399c-6173-4cb8-a8a7-c96100635077)


that looks better. now we can see that the long string of numbers in indeed [Epoch Time](https://medium.com/@kavindaperera25/what-is-the-epoch-time-f46c94e7a6e7). we also see headings for `place_id`. With ChatGPT's help, let's try a JOIN and see if we can match the `place_id` with a URL, say `craigslist.org`:

![image](https://github.com/user-attachments/assets/c778c562-3baf-40c0-af39-13fc4149460d)

sqlite> `SELECT moz_places.url, datetime(moz_historyvisits.visit_date / 1000000, 'unixepoch') AS visit_time FROM moz_places JOIN moz_historyvisits ON moz_places.id = moz_historyvisits.place_id WHERE moz_places.url LIKE '%craigslist.org%' ORDER BY visit_time DESC LIMIT 10; `


what exactly is this command doing? we are selecting columns `moz_places.url` and `moz_hostiryvisits.visit_date`, while also converting Epoch time to a typical date stamp and renaming the `visit_date` column to `visit_time`. then we're cross-referencing table `moz_historyvisits` with `moz_places` where the id matches, and where the URL is similar to `craigslist.org`, and let's order it newest first. results should look like:

![image](https://github.com/user-attachments/assets/2116d686-8b39-453e-9000-7086494f767a)


sweet! we now know when the user visited some variation of `craigslist.org`. unfortunately, this doesn't really help us answer Question 6. but it is nice to see that we're able to navigate some tables and get the output we need. Since i'm not sure at all how to answer Question 6, i utilized ChatGPT with quite a few prompts to eventually produce a [script that would dump all the information from all the tables to the screen](https://github.com/rundas-r00t/CtfStuff). 

`sqlite3 browser.sqlite < megadump.sql`

![image](https://github.com/user-attachments/assets/37eb4830-0f0e-456c-8878-e2eb9e7afd15)

scrolling further down the results, we see our Epoch Time again:

![image](https://github.com/user-attachments/assets/8779fdb1-1673-4c00-a297-73001a585dc5)

in a real-life scenario, i would look for transaction URL's similar to this:

`sqlite3 browser.sqlite < megadump.sql | grep '/tx/'`

![image](https://github.com/user-attachments/assets/3b020bf0-dbe3-4678-924e-3665936950f1)

Unfortunately, the transaction doesn't actually exist:

![image](https://github.com/user-attachments/assets/d31c183a-869e-4bff-bc4e-4c7b681c4ded)



Feeling pretty stuck on this, i had to turn to the guide, which references a really awesome website https://inloop.github.io/sqlite-viewer/ .

![image](https://github.com/user-attachments/assets/45344ebb-a365-4960-a7ed-c8e5000563a4)


let's upload our database to it and see if we can finally answer Question 6.



![image](https://github.com/user-attachments/assets/a5d6c832-2d6a-4901-86a9-01e89fd45ce1)

checking each table in the dropdown menu produces familiar data:

![image](https://github.com/user-attachments/assets/36b3c505-d0e9-493e-bc7c-acfc03969327)


again, i'm not seeing anything new, so back to the guide, which just so happens to reference the URL that we tried earlier which we thought was invalid. i guess i made a typo or didn't clean it up properly to trying it. but i tried all 3 URLs that show up using this command:


`sqlite3 browser.sqlite < megadump.sql | grep '/tx/'   `

![image](https://github.com/user-attachments/assets/a9be4a56-4066-45fa-91e6-df0d854735f4)

using notepad to clean them up b/c i'm slightly irked by this whole thing now lol:

![image](https://github.com/user-attachments/assets/2b6c1aa6-696a-4e24-b46c-803e4f87b3bc)

and loading anyone of these into a web browser produces a page like this:

<details markdown="1">
  <summary>Click to reveal</summary>


![image](https://github.com/user-attachments/assets/a2afd8c3-fa45-4341-98e2-2222541655d3)

</details>

so basically on this screen we see "2 inputs". scrolling down, we see that this matches to the "From" category:

![image](https://github.com/user-attachments/assets/e464024e-64db-4e44-a64e-41e1f0f9dc69)

adding these numbers up produces the correct answer.

</details>


phew! man Question 6 was a wild ride! but now that we got through it, Question 7 will be a breeze!

all we need to do is look at the "To:" column and copy/paste the correct address into the answer field.

<details markdown="1">
  <summary>Click to reveal</summary>


![image](https://github.com/user-attachments/assets/15cb02c8-fc4e-4cfb-b971-34843af7a654)



</details>


SQLite Forensics? Done!

