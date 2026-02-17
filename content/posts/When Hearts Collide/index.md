---
title: "TryHackMe When Hearts Collide Walkthrough"
date: 2026-02-17
tags:
  - tryhackme
  - ctf
  - medium
  - web
  - walkthrough
  - love at first breach
draft: false
cover:
  image: images/when_hearts_collide.png
  alt: This is Thumbnail
  caption: ' '
---

Let's visit the website and see how it is and what it says.

![](Pasted%20image%2020260215045807.png)

this is enough to give the hint that this is classic MD5 collision problem.

for this challenge we will use `fastcoll`

To it hassle free i downloaded the pre-made hash collision MD5 hash file
```shell
wget https://www.mathstat.dal.ca/~selinger/md5collision/hello
wget https://www.mathstat.dal.ca/~selinger/md5collision/erase

md5sum hello erase

mv hello a.jpg
mv erase b.jpg

```

now upload both one by one.

![](Pasted%20image%2020260215054646.png)

here you can see the match is completed and now you can scroll down and capture the flag.