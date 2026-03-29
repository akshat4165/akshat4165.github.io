---
title: "TryHackMe Lo-Fi Walkthrough"
date: 2026-03-29
tags:
  - tryhackme
  - ctf
  - easy
  - Application Security
  - walkthrough
draft: false
cover:
  image: images/lofi.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Lo-Fi](https://tryhackme.com/room/lofi)

NMAP Scan:

![](Pasted%20image%2020260329133438.png)

Gobuster Scan:

![](Pasted%20image%2020260329133506.png)

let's visit the web page and find the vulnerability.

So as a hint it is given that we have LFI in this room. but where?

```shell
http://10.48.130.32/?page=relax.php
```
OR
```
http://10.48.130.32/?search=
```

![](Pasted%20image%2020260329142320.png)
this returns the same page.

![](Pasted%20image%2020260329142403.png)

we got it.

```
../../../../flag.txt
```

![](FLAG.png)
