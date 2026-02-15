---
title: "TryHackMe Committed Walkthrough"
date: 2026-02-15
tags:
  - tryhackme
  - ctf
  - easy
  - git commit
  - walkthrough
draft: false
cover:
  image: images/committed.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Committed](https://tryhackme.com/room/committed)

unzip the `committed.zip` 

let's check the logs for the git commit.

```shell
git log --all
```

![](Pasted%20image%2020260215214844.png)

straightaway we can see that there is a file named `DB check` which looks very interesting.

let's see check it out.

```shell
git checkout 3a8cc16f919b8ac43651d68dceacbb28ebb9b625

ls

cat main.py
```

![](Pasted%20image%2020260215215323.png)

and here we go we found the flag.

