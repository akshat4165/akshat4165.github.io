---
title: "TryHackMe Crypto Failures Walkthrough"
date: 2026-02-13
tags:
  - tryhackme
  - ctf
  - medium
  - cryptography
  - walkthrough
draft: false
cover:
  image: images/crypto_failures.png
  alt: This is Thumbnail
  caption: ' '
---

Room Link: [Crypto Failures](https://tryhackme.com/room/cryptofailures)

upon visiting the web of the IP of target machine we can see:

![](Pasted%20image%2020260209210529.png)

so here it says we are logged in as `guest` 
and upon visiting the source code of this website.

we can see there is a comment in the file which says:

![](Pasted%20image%2020260209210624.png)

so let's run a `Gobuster` scan and see what we get here.

```shell
gobuster dir -u http://10.48.162.78 -w /usr/share/wordlists/dirb/common.txt -x bak,php,js,txt
```

![](Pasted%20image%2020260209210706.png)

here we have the `index.php.bak`

let's visit it and download the following.

let's read the file contents.

![](Pasted%20image%2020260209210833.png)

here in this code we can analyse and find our end goal which is to get the `admin` login as it says if we have admin login we can get the flag.

okay so here this code is using the PHP feature known as `crypt()`.

`crypt()` is an in-built feature in PHP which takes the input and returns the hashed strings.

there is no decrypt function as crypt() uses only one way hashing algorithm

so how we can use it on our advantage:

you can see the code and figure out there is a 2 byte salt in the start of the string and the secure_cookie format is: 

![](Pasted%20image%2020260210000221.png)

also the `make_secure_cookie` says each cipher must be 8 byte long.
and in the code for verification we can see it only checks if the user is `guest` or `admin`.

first we will intercept the request of the website on burpsuite and send it to repeater.

![](Pasted%20image%2020260210001320.png)

now we can notice a pattern here in the cookie i.e `iE` is the salt which is added to the cipher.

so let's what it says.

remember `crypt()` is a one way hashing algorithm but now as we know the salt and what the value can be for the cipher we will check it ourselves.

![](Pasted%20image%2020260210002012.png)

you can see and compare it with `secure_cookie` in the repeater request

now lets change it and see what `admin` cookie looks like.

![](Pasted%20image%2020260210002201.png)

this is the `cipher` for the admin

change user to admin and replace the cipher and you will find the flag.

### Part 2

```PHP
<?php
/**
 * Block-alignment oracle attack (PHP version)
 * Direct port of the provided Python script.
 */

error_reporting(E_ALL);
ini_set('display_errors', 1);

// --- Point it at the target ---
$IP  = "10.48.163.71";
$URL = "http://{$IP}/";

// Use all printable ASCII characters (same as Python's string.printable)
$CHARSET = '';
for ($i = 32; $i <= 126; $i++) {
    $CHARSET .= chr($i);
}
// Python's string.printable also includes \t \n \r \x0b \x0c
$CHARSET .= "\t\n\r\x0b\x0c";

function get_cookie($url, $userAgent) {
    $ch = curl_init($url);

    curl_setopt_array($ch, [
        CURLOPT_RETURNTRANSFER => true,
        CURLOPT_HEADER         => true,
        CURLOPT_FOLLOWLOCATION => false,
        CURLOPT_TIMEOUT        => 5,
        CURLOPT_HTTPHEADER     => [
            "User-Agent: {$userAgent}"
        ],
    ]);

    $response = curl_exec($ch);

    if ($response === false) {
        curl_close($ch);
        return null;
    }

    $headerSize = curl_getinfo($ch, CURLINFO_HEADER_SIZE);
    $headers    = substr($response, 0, $headerSize);

    curl_close($ch);

    // Extract secure_cookie
    if (preg_match('/secure_cookie=([^;]+)/', $headers, $matches)) {
        return urldecode($matches[1]);
    }

    return null;
}

function find_key($URL, $CHARSET) {
    $found_key = "";

    echo "Alright, let's find that secret key. This might take a minute.\n\n";

    while (true) {
        $next_char_num = strlen($found_key) + 1;
        echo "[*] Searching for character #{$next_char_num}...\n";

        // 1. Align the block
        for ($pad_length = 1; $pad_length <= 8; $pad_length++) {
            $user_agent = str_repeat('x', $pad_length);
            $plaintext_to_align = "guest:{$user_agent}:{$found_key}*";

            if (strlen($plaintext_to_align) % 8 === 0) {
                // '*' is last byte of block
                $block_prefix = substr($plaintext_to_align, -8, 7);
                break;
            }
        }

        // 2. Get real cookie
        $real_cookie = get_cookie($URL, $user_agent);

        if ($real_cookie === null) {
            echo "\n[!] Failed to get a cookie from the server\n";
            return null;
        }

        $salt = substr($real_cookie, 0, 2);
        $found_next_char = false;

        // 3. Brute-force character
        for ($i = 0; $i < strlen($CHARSET); $i++) {
            $char_guess = $CHARSET[$i];
            $test_block = $block_prefix . $char_guess;

            $hashed_block = crypt($test_block, $salt);

            if (strpos($real_cookie, $hashed_block) !== false) {
                $found_key .= $char_guess;
                $found_next_char = true;

                echo "\r[+] Key found so far: {$found_key}";
                flush();
                break;
            }
        }

        echo "\n";

        if (!$found_next_char) {
            echo "[*] No more characters found. Assuming the key is complete.\n";
            break;
        }
    }

    return $found_key;
}

// --- Run ---
$key = find_key($URL, $CHARSET);

if ($key !== null && $key !== "") {
    echo "\n=======================================\n";
    echo "  Jackpot! The secret key is:\n";
    echo "  {$key}\n";
    echo "=======================================\n";
}

```

this code is inspired by the python code of [cilgin](https://cilginc.github.io/posts/TryHackMe-Crypto_Failures/)
Replace your IP of target machine and voila!

how it works is, this code changes `user-agent` field one character at a time and get all the 8 byte chunks and get the fresh cookie to mark out the fresh 8 bytes of the cookie and try bruteforcing it and display the final flag.

run it and you will get the flag.


