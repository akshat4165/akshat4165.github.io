---
title: "TryHackMe Signed Messages Walkthrough"
date: 2026-02-17
tags:
  - tryhackme
  - ctf
  - medium
  - cryptography
  - walkthrough
  - love at first breach
draft: true
cover:
  image: images/signed_messages.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Signed Message](https://tryhackme.com/room/lafb2026e8)
Let's visit the URL and see what we got.

upon `gobuster` scan I found out an endpoint `/debug`

![](Pasted%20image%2020260215065815.png)

here I found out something which should have been shared.

according to it, I wrote a python script which will find the HEX to verify the initial message that was sent by the admin.

```python
#!/usr/bin/env python3
from hashlib import sha256

from sympy import nextprime, mod_inverse
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import rsa, padding


USERNAME = "admin"

# IMPORTANT: must match byte-for-byte what /messages shows in the browser (rendered text).
# Use the rendered apostrophe (') not the HTML entity (&#39;).
ADMIN_PUBLIC_MESSAGE = (
    "Welcome to LoveNote! Send encrypted love messages this Valentine's Day. "
    "Your communications are secured with industry-standard RSA-2048 digital signatures."
)

def derive_keypair(username: str):
    seed = f"{username}_lovenote_2026_valentine".encode()

    p_base = int.from_bytes(sha256(seed).digest(), "big")
    p = int(nextprime(p_base))

    q_base = int.from_bytes(sha256(seed + b"pki").digest(), "big")
    q = int(nextprime(q_base))

    n = p * q
    e = 65537
    phi = (p - 1) * (q - 1)
    d = int(mod_inverse(e, phi))

    pub = rsa.RSAPublicNumbers(e=e, n=n)
    priv = rsa.RSAPrivateNumbers(
        p=p,
        q=q,
        d=d,
        dmp1=d % (p - 1),
        dmq1=d % (q - 1),
        iqmp=int(mod_inverse(q, p)),
        public_numbers=pub,
    ).private_key()

    return priv

def sign_pss_sha256(privkey, message: str) -> str:
    msg_bytes = message.encode("utf-8")  # must match server encoding assumption; UTF-8 is the sane default
    sig = privkey.sign(
        msg_bytes,
        padding.PSS(
            mgf=padding.MGF1(hashes.SHA256()),
            salt_length=padding.PSS.MAX_LENGTH,  # common “PSS default”
        ),
        hashes.SHA256(),
    )
    return sig.hex()

def main():
    priv = derive_keypair(USERNAME)
    sig_hex = sign_pss_sha256(priv, ADMIN_PUBLIC_MESSAGE)

    print("username=admin")
    print("message (exact) =", ADMIN_PUBLIC_MESSAGE)
    print("signature hex =", sig_hex)

if __name__ == "__main__":
        main()
```


![](Pasted%20image%2020260215065519.png)

scroll down and you will get the flag.

