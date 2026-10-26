---
layout: post
title: "Transaction Payment Logs"
author: "rundas-r00t"
date: 2025-03-22 12:00:00 -0000
categories: ctf writeups
---

# Transaction/Payment logs

we are give a file `payments.log` and asked these questions three:

1. How many transactions are contained in the log?
1. What is the transaction ID of the largest purchase made in the log?
1. Which state made the greatest number of purchases?



<details markdown="1">
  <summary>Spoiler</summary>
**NOTE:** per the walkthrough, there is a way to parse the XML inside the log file and convert it to a CSV for use with Excel & filters, etc. I did not do this. i don't know which method is inherently "better" or "more correct". overall this challenge took me a couple hours to complete. if i had to do it again without researching the commands, i'm sure my method would be very fast. but if time is essential, it may very well be faster to try to convert the data to csv.
</details>


i have never done anything with payment logs before so let's stick to the basics and see how it goes.

first, let's `head -n5 payments.log` and see what we get:


<details markdown="1">
  <summary>Click to reveal</summary>


![image](https://github.com/user-attachments/assets/136c7b0b-3516-4697-9c39-4bf0bf7fec83)


ok so, at first glance, this is an ebay payment sent via the PayPal API Service. keywords that pop out to me are `Password`, `PayPalAPI`, `CreditCard` (and credit card info), and some address/shipping information.


</details>


Doing a bit of research with ChatGPT, we discover that the transaction information always begins with `\<ns:DoDirectPaymentReq\>`. so let's try to get a count of this phrase, which should answer Question 1:


<details markdown="1">
  <summary>Click to reveal</summary>

you can grep this in a few different ways. i prefer to grep output to a file and then `wc -l` as we've seen in prior write-ups:

`grep  '<ns:DoDirectPaymentReq>' payments.log > totaltransactions.txt; wc -l totaltransactions.txt`

![image](https://github.com/user-attachments/assets/89ba65e1-abc5-43ac-8db2-e8d75b396680)

if you don't care about writing to a file, you can do `grep -c` to just count the iterations of the phrase:

`grep -c '<ns:DoDirectPaymentReq>' payments.log`

![image](https://github.com/user-attachments/assets/569fcfa3-1355-48f0-9824-9d984469c8ea)


</details>


for the next question, we need to isolate the amounts of the transactions , paired with the transaction Id's, and `sort -nr` to find the largest amount. with the format that this log-file is in, it doesn't look like we'll be able to easily `awk`. but to start, we do see that there is a `currencyID="USD"\>932.56\<` phrasing, so maybe if we grep on "ID", we can pick out some transaction ID's and amounts.

let's do it.

<details markdown="1">
  <summary>Click to reveal</summary>

`grep 'ID' payments.log`

![image](https://github.com/user-attachments/assets/15863a17-6728-4a24-9881-80db44a21f98)

we have a lot of output here!, but thankfully towards the bottom, we can see a few different ID's: 

`CorrelationID`

`currencyID`

`TransactionID`

using ChatGPT for a decent 30 minutes, i was able to iterate through some commands & bad results to finally produce a handy one-liner that will output transaction ID's and amounts to a separate file for further parsing:

`grep -oP '<ebl:OrderTotal[^<]+</ebl:OrderTotal>|<TransactionID>[^<]+</TransactionID>' payments.log | paste -d' ' - - > orders.txt `

![image](https://github.com/user-attachments/assets/fa04b85b-0c22-4e0e-8aa4-023519878827)



</details>

we're getting close. we have the transaction ID's and the order amounts extracted from the log file. now we just need to figure out how to further reduce it to just the dollar amount and the transaction ID.

<details markdown="1">
  <summary>Click to reveal</summary>

`awk -F'[><]' '{print $3,$7}' orders.txt | sort -nr > 2goOrders.txt; head 2goOrders.txt`

![image](https://github.com/user-attachments/assets/09f5abb2-fea0-409c-92ed-0b2a9ca79d27)

</details>

alright Alright AlRiGhT!! now on to question 3




with ChatGPT's help on how to delimit the file, i was able to produce a one-liner to isolate the State field for each line:

<details>
  <summary>Click to reveal</summary>

`grep -oP '<ebl:StateOrProvince>\K[^<]+' payments.log > states.txt`

</details>

now that we have the full transaction list, with only the state abbreviation, we can do a bash loop to iterate through all the states, `wc -l`'ing and seeing the total transaction counts of each state:

<details markdown="1">
  <summary>Click to reveal</summary>

`for state in AK AL AR AZ CA CO CT DE FL GA HI IA ID IL IN KS KY LA MA MD ME MI MN MO MS MT NC ND NE NH NJ NM NV NY OH OK OR PA RI SC SD TN TX UT VA VT WA WI WV WY; do echo -n "$state: "; grep "$state" states.txt | wc -l; done > TcountStates.txt; cat TcountStates.txt`



![image](https://github.com/user-attachments/assets/22c2aac4-eef6-470a-8357-b5841087a702)

</details>

now we can easily scan through the output and see which state has the highest number, answering the last question.




Payment logs are messy, but we got through it!!
