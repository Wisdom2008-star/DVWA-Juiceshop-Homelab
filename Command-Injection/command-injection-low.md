# Command Injection — Low

## Setup

- Target: DVWA running locally (`http://localhost:8080/vulnerabilities/exec/`)
- Security level: **Low**
- Environment: Kali Linux (VirtualBox), Firefox with Burp Suite/FoxyProxy

## Step 1: Baseline test

The page has a simple form: "Ping a device" — you enter an IP address and hit
Submit. It's meant to just ping that address and show you the result.

Entering a normal IP address like `127.0.0.1` works exactly as expected — it
pings the address and shows a standard ping result (4 packets sent, 0% loss).

This confirms the app is actually running a real `ping` command behind the
scenes using whatever we type into the box. That's the detail worth digging
into next — if it just hands our input to the system shell, what happens if
we sneak an *extra* command in alongside the IP?
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20easy%20pic%201.png">
## Step 2: View the source

```php
$target = $_REQUEST['ip'];

if( stristr( php_uname('s'), 'Windows NT') ) {
    $cmd = shell_exec( 'ping  ' . $target );
} else {
    $cmd = shell_exec( 'ping  -c 4 ' . $target );
}

echo "<pre>{$cmd}</pre>";
```
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20easy%20pic%202.png">

**Why this is vulnerable:**

- `$target` comes directly from user input (`$_REQUEST['ip']`) with **no
  validation or sanitization**.
- It's concatenated straight into a shell command string and passed to
  `shell_exec()`.
- The shell has no way to tell "user data" apart from "command syntax" — so
  if the input contains characters like `;`, `&&`, or `|`, the shell
  interprets them as **command separators**, not as part of an IP address.

This means we can submit an IP *plus* an extra command, and the server will
run both.

## Step 3: Craft and run the payload

Instead of a plain IP address, we submit:

```
127.0.0.1 ; whoami
```

The `;` character is a shell command separator — it tells the shell "run
this command, then run the next one," regardless of whether the first one
succeeds. Since our input gets pasted straight into a shell command with no
filtering, the server happily runs both `ping 127.0.0.1` **and** `whoami`.
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20easy%20pic%203.png">

**Result:** the page shows the normal ping output, followed by an extra line:

```
www-data
```

This is the output of `whoami` — it tells us the web server process is
running as the user `www-data`. We've successfully broken out of the
intended "ping" functionality and executed an arbitrary command of our
choosing.

## Takeaway

Because user input is concatenated directly into a shell command without
validation, an attacker isn't limited to pinging IPs — they can chain on
**any** command the web server's user account is allowed to run (e.g.
`cat /etc/passwd`, or worse). This is why the Low level is considered fully
vulnerable to command injection.
