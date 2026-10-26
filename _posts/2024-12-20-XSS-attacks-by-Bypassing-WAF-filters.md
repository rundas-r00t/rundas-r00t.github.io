---
layout: post
title: "XSS Attacks By Bypassing WAF Filters"
author: "rundas-r00t"
date: 2024-12-20 12:00:00 -0000
categories: "webapp-attacks"
---





# XSS WAF bypasses






if you get a block, try different variations of the below, removing each keyword individually (basic troubleshooting steps) so you can figure out which keyword (or combination of) has the block. this will give you clues into how to change the syntax so that you can bypass the WAF

`<img src=x onerror=prompt()>`

`<img src=x onmouseover=prompt()>`

`<im"g s"rc=x on"error=prom"pt()>`

also try with just <> or the payload without <>. this will add to the blocked/passed variations. again, try base64 or URL encodings to see if encoded messages can be passed through.

WAF's many times ignore messages over a certain size (kb/16kb/100kb, etc). so try a large payload of say, 10,000 letter 'a's, then have your payload at the end of all those a's

`[imagine 10,000's a's here] <img-src=x onerror=prompt()>`

there is a way to script out via python or burp to try different amounts of a's, but i do not know how to do this yet (as of 12/20/24). will update this later when i figure it out.

checking other potential vulns, just write plane text with a bunch of special characters on it, and see what gets filtered

`plain text nothing special~!@#$%^&*()_+=-::;""''`

sometimes quotes will be filtered, so this allows you to break up your payload and pass through the checks, as seen in the 3rd example payload above and right here as well:

`<im"g s"rc=x on"error=prom"pt()>`
`//this example is for when the WAF filtered "`

plop payloads into search forms, or any type of input box. you can also add it to URL parameters if the `search?q=` statement is a valid statment in the URL bar of the webapp.

the `prompt()` pop-up is a proof of concept (PoC) for the attack. once the WAF is bypassed and you get the prompt, you can then run simple JS scripts to: create keyloggers; steal session cookies from the server and proceed to session hijacks; hijack the timeout logout message with a poisoned URL that goes to a fake log-in page to log-in again; deface the website by changing the HTML, or scan internal/corporate IP's that the webapp might be connected to and attempt to pivot to those IPs.

it's important to note that XSS is not Remote Code Execution, as you can only interact with the web server. you are limited by the access the webserver has and what services are running. but, XSS will allow you to start looking for open ports & services with vulnerabilties that may allow you to steal credentials and/or phish users in order to gain privilege and potentially a shell.
