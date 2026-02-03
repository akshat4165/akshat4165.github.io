---
title: "TryHackMe Airplane Walkthrough"
date: 2026-02-03
tags:
  - tryhackme
  - ctf
  - medium
  - Application Security
  - walkthrough
  - guide
draft: false
cover:
  image: images/SSTI_Guide.png
  alt: This is Thumbnail
  caption: ' '
---

SSTI means to take advantage of an insecure implementation of the template engine.

what is a Template engine?

Template engine allows you to create static template files which can be re-used in your application.

so look at the code below:

`````````python
from flask import Flask, render_template_string app = Flask(__name__)  @app.route("/profile/<user>") def profile_page(user):     template = f"<h1>Welcome to the profile of {user}!</h1>"      return render_template_string(template)  app.run()
`````````

this code creates a template string, and concatenates the user input into it.

It can be exploited in multiple ways, but main goal is to get the RCE. 

This vulnerability is very critical as it not a client side vulnerability, but is a server-side vulnerability.


### Detection

the exploit must be inserted somewhere which is called ```injection point.```

```look at the url or an input box - make sure to check the hidden input boxes```

#### Fuzzing
fuzzing is a technique to determine whether the server is vulnerable by sending multiple characters in hopes to interfere with the backend system.

It can be done ```manually``` or by the use of application ```burpsuite's intruder``` which is generally used for brute-forcing.

the following characters are known to be used in quite a few template engines:
```${{<%[%'"}}%```


Continue with this process until you either get an error, or some characters start disappearing from the output.

so ```{{``` caused an error.

#### Identification

Now that we have found out what is causing the error, now we have to find out which template is being used.

In the best case scenario, the error message will include the template engine name.

if now we can use this decision tree:

![](SSTI%20Template%20decision%20tree.png)

so start at the very left of the decision tree and include the variable in the request. follow the arrow depending upon the output.

- Green Arrow - if the expression is evaluated.
- Red Arrow - The expression in shown in the output as it is.

#### Syntax

So once we identified the template engine name, we need to learn its syntax.

we can obviously use the official documentation for learning it.

but no matter what the language is, we always look for:

- How to start a print statement
- How to end a print statement
- How to start a block statement.
- How to end a block statement.

In Jinja2:

- ```{{``` used to mark start of print statement
- ```}}``` used to mark end of print statement.
- ```{%``` used to mark start of block statement.
- ```%}``` used to mark end of block statement.

### Exploitation

So for exploitation, we will always think of running a shell code command, we will search every possibility to run shell code in python as jinja2 is python based.

`````````python
# Method 1 
import os os.system("whoami")  
# Method 2 
import os os.popen("whoami").read()  
# Method 3 
import subprocess subprocess.Popen("whoami", shell=True, stdout=-1).communicate()
`````````


so now we will craft a proof of concept.

```http://10.49.140.97:5000/profile/{% import os %}{{ os.system("whoami") }}```


as jinja is sub language of python it does not support ```import```.

Python allows us to call the current class instance with [.__class__](https://docs.python.org/release/2.6.4/library/stdtypes.html#instance.__class__), we can call this on an empty string:

Payload: ```http://10.49.140.97:5000/profile/{{ ''.__class__ }}```.

Classes in Python have an attribute called [.__mro__](https://docs.python.org/release/2.6.4/library/stdtypes.html#class.__mro__) that allows us to climb up the inherited object tree:

Payload: ```http://10.49.140.97:5000/profile/{{ ''.__class__.__mro__ }}```.

Since we want the root object, we can access the second property (first index):

Payload: ```http://10.49.140.97:5000/profile/{{ ''.__class__.__mro__[1] }}```

this is the payload cheat sheet which can be used [Github For SSTI Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)

### Remediation
- Secure methods like ```user=user``` can be used.
- regex methods can be used for sanitization.
