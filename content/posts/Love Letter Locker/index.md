---
title: "TryHackMe Love Letter Locker Walkthrough"
date: 2026-02-17
tags:
  - tryhackme
  - ctf
  - medium
  - web
  - walkthrough
  - love at first breach
draft: true
cover:
  image: images/loveletter.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Love Letter Locker](https://tryhackme.com/room/lafb2026e2)

Let's visit the website and analyse the interface.

we have sign in and login buttons as always we will register and new account.

![](Pasted%20image%2020260214231632.png)

let's login now and find write a new letter
![](Pasted%20image%2020260214231750.png)

and let's open it now

![](Pasted%20image%2020260214231836.png)

here in this link you can notice that the `3` in the link matches the letter number as well, so let's try to change it and try to read the other letters.

![](Pasted%20image%2020260214232124.png)

and here we go there is an **Insecure Direct Object Reference(IDOR)** vulnerability in this website.

change the link number to `1` and you can see that we found out the flag.

