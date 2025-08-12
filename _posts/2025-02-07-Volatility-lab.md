---
layout: post
title: "Memory Forensics - Volatility"
date: 2024-01-27
tags:
  - Jekyll
  - Markdown
author: Mers Wafula
category: Topics
---

### Diving into Volatility: A 13Cubed Challenge Adventure

Introduction

Today’s lab is all about digital forensics fun! I’m tackling one of 13Cubed’s memory [dumps](https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbThmSEVVREpWbUJwNDk1ejhrRVBNU2ZFc1hQZ3xBQ3Jtc0tuRHNib1I1Zy1Fd01zbWRhTkllVUZxbVVfLTdnb2k0QnlxYnBqRnpWMTNjbzNmVHNxam1XNFdORzNMYUlzUHRxSThRaF9mZElyOWRrLWZrMldXa2VHN2lhZTRUNlFiTVJ2SGloaktYU0RWcEJlLWJ5Yw&q=https%3A%2F%2Fdigital-forensics.sans.org%2Fmedia%2Fposter_2014_find_evil.pdf&v=s98_p3bheL0), where the challenge is simple—extract the password for the user 13cubed. Easy, right? Well, let’s go!

Tools:Volatility

Step 1: Checking for Suspicious Processes

First things first—time to analyze the processes running on the system. Using Volatility’s pslist plugin, I spotted something odd:

All svchost.exe processes have a PPID of 796 (services.exe) except one. Suspicious? You bet! Time to investigate.

![](assets/3.png)
Finding the Parent Process

Let’s check which process is associated with PID 7416:

vol.py -f memdump_Win10x64_15063.mem windows.pstree | grep 7416

![](assets/4.png)

This revealed three related processes, including svchost.exe and lsaiso.exe, both chilling in the wrong directory. Normally, these should be nestled safely inside %SystemRoot%\System32\

Malicious? Or Just Bad at Hiding?

Could these be actual threats, or just decoys for the exercise? Either way, let’s dump their files and hash them for further analysis.

Step 2: Dumping Suspicious Files

Using dumpfiles, we extract the binaries:

vol.py -f memdump_Win10x64_15063.mem -o proc windows.dumpfiles --pid 3456

![](assets/5.png)

![](assets/6.png)

you can also check the cmdline:

![](assets/7.png)

With these files in hand, I can now check their hashes to determine if they match any known malware samples.

Step 3: Tracking Network Connections

Time to investigate network activity using netscan:

vol.py -f memdump_Win10x64_15063.mem windows.netscan

![](assets/8.png)

Wow, look at that—tons of connections! To cut through the noise, I filter out common web traffic (ports 80 and 443) and focus on ESTABLISHED connections. 

![](assets/9.png)

Step 4: The Grand Finale—Dumping Hashes!

Finally, it’s time for the big reveal. Using the hashdump plugin, I extract stored password hashes:

vol.py -f memdump_Win10x64_15063.mem windows.hashdump

![](assets/34.png)

💀 BOOM! Three password hashes appear. One of them belongs to the 13cubed account.

Now comes the fun part—cracking it!

Step 5: Cracking the Password with John the Ripper

I save the dumped hash to a file then John the Ripped did the magic



13Cubed challenge is complete!
