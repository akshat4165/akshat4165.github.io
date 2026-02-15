---
title: "TryHackMe Try Heart Me Walkthrough"
date: 2026-02-17
tags:
  - tryhackme
  - ctf
  - easy
  - web
  - walkthrough
  - love at first breach
draft: true
cover:
  image: images/tryheartme.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [TryHeartMe](https://tryhackme.com/room/lafb2026e5)

![](Pasted%20image%2020260214011737.png)

upon visiting the website we can see that there are 4 products and we can notice here that there is login and signup options  as well,

so let's sign up.

after signing up we will click on some product.

![](Pasted%20image%2020260214011947.png)

here our role says `user` so we can assume that to buy the hidden product we have to change our role to someone else.

let's intercept this with burpsuite and check what request it is sending to the website.

![](Pasted%20image%2020260214012138.png)

here we can see there is a `JWT` token i.e `Json Web Token` so were definitely correct about the role changing guess.

![](Pasted%20image%2020260214012256.png)

even on the home page we can see there is `JWT` so let's see what it decode to.

![](Pasted%20image%2020260214012453.png)

so here in cyberchef we can see this `JWT` says our role is `user` so we have to change it to admin.

but the JWT tokens are signed by a secret key, so let's see what is the secret key here.

![](Pasted%20image%2020260214012744.png)


here we didn't have to do much as there was hint in the `theme` that the secret key could be valentine and yes it was.

![](Pasted%20image%2020260214012844.png)

here you can see we can see the `ValenFlag` product which is only accessible by staff.


while clicking on the product it will show you error as you token will reset again so just intercept every click and change the token and you can proceed further easily.


![](Pasted%20image%2020260214013059.png)


here you can see our role is changed to `admin`.

and click on `Buy` and repeat the same procedure and you will find the flag.




