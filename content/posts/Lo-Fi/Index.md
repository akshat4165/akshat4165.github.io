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

![](1.png)

Gobuster Scan:

![](2.png)

let's visit the web page and find the vulnerability.

So as a hint it is given that we have LFI in this room. but where?

```shell
http://10.48.130.32/?page=relax.php
```
OR
```shell
http://10.48.130.32/?search=
```

![](3.png)
this returns the same page.

![](4.png)

we got it.

```shell
../../../../flag.txt
```

![](FLAG.png)
