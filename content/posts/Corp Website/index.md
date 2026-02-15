---
title: "TryHackMe Corp Website Walkthrough"
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
  image: images/corp_website.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Corp Website](https://tryhackme.com/room/lafb2026e7)

![](Pasted%20image%2020260215071855.png)

so here with wappalyzer we can see that this is built on `React` and `Next.js` which more likely tell us we should scan for `React2Shell`

I'm using this [scanner](https://github.com/assetnote/react2shell-scanner)

![](Pasted%20image%2020260215072234.png)

here you can see our guess was correct, this website is vulnerable to `React2Shell`

Now to exploit it we will use [Exploit](https://github.com/xalgord/React2Shell)

![](Pasted%20image%2020260215072649.png)

![](Pasted%20image%2020260215073338.png)

here we got the `user.txt` flag.

![](Pasted%20image%2020260215073458.png)

here we can see our `uid` is `daniel` and `sudo -l` tells us something interesting to run.

let's check [GTFO bins](https://gtfobins.org/)

![](Pasted%20image%2020260215081950.png)

after pasting this command we got our `root` flag.