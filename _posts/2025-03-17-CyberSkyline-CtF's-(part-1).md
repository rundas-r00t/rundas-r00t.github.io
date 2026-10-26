---
layout: post
title: "Cyberskyline CTF's (part 1)"
date: 2025-03-17 12:00:00 -0000
categories: ctf writeups
---

# File Forensics

[https://rumble.com/v6qqh7e-rundas-gets-r00t-lets-practice-some-ctfs.html](https://rumble.com/v6qqh7e-rundas-gets-r00t-lets-practice-some-ctfs.html)

worked on some Forensics-based CtF's under chapter "Doctor" through CyberSkyline's gymnasium, of which forensics is not my forte', but i was able to struggle through it.

Download SuperAwesomeDoc.docx, and run exiftool to get some basic info about the document:

![image](https://github.com/user-attachments/assets/3669ec25-6981-4535-b3b5-8f9ed5891bca)



Now, at this point, i'm supposed to know that 


<details>
  <summary>Click to reveal</summary>

the docx is actually a zip file?  ...apparently all docx files are actually zip files with xml content. this is how docx files are structured. a good thing to note and keep in mind for any future docx forensics.

</details>

but until this challenge, i was not aware of this. so naturally, i do the next step, which is to open the docx and check it out. 

Inside we see a series of images related to Halloween:

![image](https://github.com/user-attachments/assets/a53f463c-895d-43d1-891b-0f0dda2aa8c6)


I r-clicked-->save as for each image & ran exiftool on each, but this ended up being a dead end. Out of ideas, i had to check the walkthrough, which mentions the spoiler above and the nature of the docx file format.
..so now that we know a bit better how docx's are formed, we can 

<details>
  <summary>Click to reveal</summary>
rename the file to a .zip extension and extract the data using `unzip` or r-click extract

</details>

Now that we everything extracted, we have more files and folders to research. I then began opening the additional xml files to scan for anything new, which led me to see a few files named image1.png, image2.png, etc. I then noticed a `Media` folder, which i opened, and discovered it housed all the images, including a file that wasn't referenced in the XML:


<details>
  <summary>Click to reveal</summary>
![image](https://github.com/user-attachments/assets/459db64a-057f-4568-a99a-7ee65934e6d3)

</details>

double-clicking this file returns the flag, which then allows us to answer questions "what is the name of the file that holds the flag?" and "what is the flag?"


See? this type of Forensics isn't too bad. Atleast with this one, we don't have to change hex bytes to restore a png! lol



On to the Next Challenge...
