---
title: "TryHackMe Operation Slither Walkthrough"
date: 2026-02-04
tags:
  - tryhackme
  - ctf
  - easy
  - walkthrough
  - OSINT
draft: false
cover:
  image: images/OperationSlither.png
  alt: This is Thumbnail
  caption: ' '
---

![](Pasted%20image%2020260204142507.png)

let's see what info do we have here.

so the name of the leader here is:

`@v3n0mbyt3_` 

On a simple google search we can see the X account under the name.

but we have to find another social media where `@v3n0mbyt3_` is used as a username.

just scroll down more and you will see a new social media platform from meta.

![](Pasted%20image%2020260204143204.png)

going to the replies we can see there is a base64 encoded text here

let's decode it using terminal.

```shell
echo "VEhNe3NsMXRoM3J5X3R3MzN0el80bmRfbDM0a3lfcjNwbDEzcyF9" | base64 -d
```

and we got the flag.

### Task 2

![](Pasted%20image%2020260204143427.png)

we can see from the Task 1 replies,

The account talked to the leader.

let's check the user's Instagram.

![](Pasted%20image%2020260204150733.png)

we can see the sound cloud link here. let's visit

![](Pasted%20image%2020260204151004.png)

this audio is the what we needed. visit it and you will find another base64 encoded text 

```shell
echo "VEhNe3MwY20xbnRfMDBwczNjX2Yxbmczcl9tMXNjbDFja30=" | base64 -d
```


and we got the flag.
### Task 3

![](Pasted%20image%2020260204151236.png)


![](Pasted%20image%2020260204151736.png)

i think we can identify the suspicious follower here.

we got our first flag here.

we can identify the answer for second question with the 3rd point in recon guide. or even a simple google search for username will also do.

![](Pasted%20image%2020260204153146.png)

we found this GitHub repo here

![](Pasted%20image%2020260204153604.png)

under commits you can find this file which has shadow password in the form of base64.

```shell
echo "VEhNe3NoNHJwX2Y0bmd6X2wzNGszZF9ibDAwZHlfcHd9" | base64 -d
```

BOOM! we got our flag here.

