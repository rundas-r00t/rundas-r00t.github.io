---
layout: post
title: "Squid Logs"
date: 2025-03-22 12:00:00 -0000
categories: ctf writeups
---

# Squid Log Forensics with `awk`, `grep`, and `sort`

we are given a `squid_access.log` squid proxy log file and are asked the following questions:

1. In what year was this log saved?
1. How many milliseconds did the fastest request take?
1. How many milliseconds did the longest request take?
1. How many different IP addresses did the proxy service in this log?
1. How many GET requests were made?
1. How many POST requests were made?
1. What company created the antivirus used on the host at 192.168.0.224?
1. What url is used to download an antivirus update?

just for my own sanity, and to easily answer some of the date/time related questions, let's convert all the timestamps from Epoch time to regular timestamps:

<details>
  <summary>Click to reveal</summary>

`awk '{ cmd="date -d @"$1" \"+%Y-%m-%d %H:%M:%S\""; cmd | getline date; close(cmd); print date, $0 }' squid_access.log > new.log`

`head new.log`

![image](https://github.com/user-attachments/assets/04f1abd2-45ff-4f32-8ca3-75f9c5bb64e4)

looks pretty good. now we can easily answer Question 1. 

</details>

On to Question 2. in order to answer this one, we need to understand the structure of squid logs. squid logs don't have column headers, but if they did, they would look like this:

<html><body>
<!--StartFragment-->

Timestamp	Response_Time_ms	Client_IP	Result_Code	Size_Bytes	HTTP_Method	URL	User_Ident	Hierarchy_Status	Content_Type

1619654401.123	456	192.168.1.100	TCP_MISS/200	512	GET	http://example.com/index.html	-	HIER_DIRECT/93.184.216.34	text/html

</tbody><!--EndFragment-->
</body>
</html>


with each column being space delimited. again, that's:

**`Timestamp Response_Time_ms Client_IP Result-Code Size_Bytes HTTP_Method URL User_Ident Hierarchy_Status Content_Type`**

10 columns total.


To answer Question 2, we need to isolate column 2 in the original log file (the one that's properly space-delimited) using `awk`, and then `sort -n` to "sort by **n**umber smallest to largest". 

<details>
  <summary>Click to reveal</summary>

`awk '{print $2}' squid_access.log | sort -n | head -1`

![image](https://github.com/user-attachments/assets/5ff73df3-7f6a-4a73-85a0-b9a1ee09935a)



</details>

to get the next question, we do a similar statement, but use `sort -nr` instead ("sort by **n**umber smallest to largest, but in **r**everse)


<details>
  <summary>Click to reveal</summary>

`awk '{print $2}' squid_access.log | sort -nr | head -1`

![image](https://github.com/user-attachments/assets/188807f0-cee6-401f-b33e-65da8dfedd7b)

</details>


for this next question, we're going to `awk` again, `sort -u` to grab unique IP addresses, and then `wc -l` to grab a total number of unique IPs:

<details>
  <summary>Click to reveal</summary>


`awk '{print $3}' squid_access.log | sort -u > IPs.txt;  wc -l IPs.txt`  (remember, column 3 is the Client_IP column)

![image](https://github.com/user-attachments/assets/8f448707-626f-4f33-8828-4a00a8c2fd4a)

</details>

Easy enough. For the Question 5, let's continue `awk`ing, this time isolating column 6 (HTTP_method), and then grepping with `wc -l` to find the specifics:


<details>
  <summary>Click to reveal</summary>


`awk '{print $6}' squid_access.log > requests.log`

`cat requests.log| grep 'GET' | wc -l `

![image](https://github.com/user-attachments/assets/499e8cb8-15cf-4f3a-b3c4-dbb1393cbd40)

and again but for POST for Question 6:

`cat requests.log| grep 'POST' | wc -l`

![image](https://github.com/user-attachments/assets/ea7cbf9b-bf44-4c8c-880e-a87f03e39434)


</details>

for Question 7, i honestly have no idea how to find this, so let's just `grep` for it and see if we get lucky:

<details>
  <summary>Click to reveal</summary>


`grep '192.168.0.224' squid_access.log  `

![image](https://github.com/user-attachments/assets/904a3a53-28e4-49b8-852f-97cb87f18142)

we can clearly see in the URL that this is a `symantec` domain and with a keen eye, we can see `norton` in the URL. 

**NOTE:** if you get a question similar to this in the future, always do \<company\> \<software\>, similar to how you'd do \<make\> \<model\> for a car. 

</details>

Making good progress. `awk` and `grep` are our friends when parsing through squid logs, for sure!

On to the last question. we already have the answer on our screen, we just need a keen eye once more. updates are files that get unpacked and installed. so using the above command again, but adding `zip`, we can further isolate and be sure to copy/paste the proper information:


<details>
  <summary>Click to reveal</summary>

`grep '192.168.0.224' squid_access.log  | grep 'zip'`

![image](https://github.com/user-attachments/assets/544e5857-2583-410f-8974-8d2e30ee5fd0)

</details>

And there we have it! Squid Logs? DONE!
