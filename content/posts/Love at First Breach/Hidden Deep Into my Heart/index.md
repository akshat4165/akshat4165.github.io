---
title: "TryHackMe Hidden Deep Into My Heart Walkthrough"
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
  image: images/hidden_deep.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Hidden Deep Into My Heart](https://tryhackme.com/room/lafb2026e9)

![](Pasted%20image%2020260214005041.png)

upon `gobuster` scan we found out `robots.txt`

![](Pasted%20image%2020260214005124.png)

let's visit the `/cupids_secret_vault/`

![](Pasted%20image%2020260214005226.png)

let's run a `gobuster` scan on this directory

![](Pasted%20image%2020260214005744.png)

![](Pasted%20image%2020260214005725.png)


here we have the login to `Cupid's Vault`

you can use simple credentials and we already have the hint for the password.




