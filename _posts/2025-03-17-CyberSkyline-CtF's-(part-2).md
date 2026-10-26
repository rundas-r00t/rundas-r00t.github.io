---
layout: post
title: "Cyberskyline Ctf'S (Part 2)"
date: 2025-03-17 12:00:00 -0000
categories: ctf writeups
---

# Memory Analysis with `cat`, `grep`, and `strings`

https://rumble.com/v6qqh7e-rundas-gets-r00t-lets-practice-some-ctfs.html ~00:20:52 timestamp


The next Forensics challenge is named `The Book`, and it deals with using Volatility to parse a memory capture file.

Unfortunately, i could not get Volatility installed on my VM due to an error:

![image](https://github.com/user-attachments/assets/f6071ae8-351d-45ac-ad06-e93131d7e9d2)

After the stream had ended, I was able to fix this error by utilizing ChatGPT to help me troubleshoot and fix the issues. If you run into this error, here's how to get Volatility installed:

<details>
  <summary>Click to reveal</summary>

first, create a virtual environment:  ` python3 -m venv venv-vol3       `

then, source the binaries to activate it: ` source venv-vol3/bin/activate  `

finally, now that you're inside your `venv` (notice the prompt change), you can install volatility: ` pip install volatility3  `

Now, this is great, but you have to source and activate your `venv` in order to use this. this is lame. we want to be able to just type `vol [command details]` in a regular terminal. to do that, we want to make `vol` a sort of alias/global command by creating a bash file. 

At this point, ChatGPT had me re-clone the git, and recreate the `venv` just for a clean start. run each of these commands separately:

` git clone https://github.com/volatilityfoundation/volatility3.git `

` cd volatility3` 

` python3 -m venv venv `

` source venv/bin/activate `

at this point, inside your `venv`, you can run volatility without issue. however, we just want to run it globally, and not in a `venv`. to do that, we make a `vol` bash file with these contents: 

`#!/bin/bash`

`# Change this to the actual path of your cloned volatility3 folder:`

`VOL_PATH="$HOME/volatility3"`

`source "$VOL_PATH/venv/bin/activate"`

`python3 "$VOL_PATH/vol.py" "$@"`

`deactivate `

make sure that your `vol` bash file is listed in `~/bin`. we will reference this file in our `vol` command. 

make the file executable: ` chmod +x ~/bin/vol`

add this statement to your `.zshrc` or `.bashrc` file, either via a terminal command: `$ echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc` 

or open the file and copy/paste this at the bottom: `export PATH="$HOME/bin:$PATH"`

source your `.zshrc` file: `$ source ~/.zshrc` to reload the file into the run-time environment. and at this point, you should be good to run:

 `vol -f memdump.mem <plugin.PlugIn> `

hooray!!



</details>



Running into this error, and being on a live-stream, i bailed on using Volatility and instead tried to parse the memory file using `strings` and `grep` and was able to answer a few of the questions, "what OS was this dump taken from?", "What is the name of the computer?", and "What is the name of the user that was logged in?".

download the memdump.mem.xz file and extract it using `unxz memdump.mem.xz`. this command will automatically delete the file.xz file, so don't fret if it disappears after extration. 

![image](https://github.com/user-attachments/assets/eb9da741-df31-4207-a225-203f32516588)

timestamp 00:44:53

![image](https://github.com/user-attachments/assets/d32efa3d-c4a3-42b5-8703-e839d52d7a0c)

![image](https://github.com/user-attachments/assets/48896ea5-15b7-470a-be7e-a28dd3fa44d3)

i took a moment to scan through the output to find a handful of Trojans from AndroidOS, as well as one line showing a MacOS. further guessing some strings/grep commands, i then tried 

<details>
  <summary>Click to reveal</summary>

` strings memdump.mem | grep 'OS= ' 

</details>

, which i guessed that the ` = ` would provide me with more solid info:

<details>
  <summary>Click to reveal</summary>

![image](https://github.com/user-attachments/assets/32cc6694-6a00-4048-9512-22a4ed998eb2)


</details>

As you can see, we have one line showing again and again and again. So i took a guess that this was the proper machine, and it was! hooray! one question down.

continuing my ` strings|grep ` investigation, i then tried 
<details>
  <summary>Click to reveal</summary>

` strings memdump.mem | grep 'NAME=' ` 

to see if we could get a computer name:

![image](https://github.com/user-attachments/assets/0ec6065c-581e-4e2f-a4c6-602385065ea0)


</details>


This actually provided the answer to two questions: "What is the name of the computer?" and "What is the name of the user that was logged in?".

I feel like i was getting a bit too lucky with these, and my luck had run out, because i could not find the answers to the final 3 questions utilizing this `strings|grep` method. I needed volatility.


At this point, i ended the stream because I was out of time, and I wanted to troubleshoot my Volatility issues offline. We will finish up this challenge next time...on....D R A G O N B A L L ~!~!~!~!~! :-D
