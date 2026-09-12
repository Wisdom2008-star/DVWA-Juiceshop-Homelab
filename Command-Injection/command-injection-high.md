# Command Injection — High

## Setup

- Target: DVWA running locally (`http://localhost:8080/vulnerabilities/exec/`)
- Security level: **High**
- Environment: Kali Linux (VirtualBox), Firefox with Burp Suite/FoxyProxy

## Step 1: Baseline test

Same "Ping a device" form as before. A normal IP like `127.0.0.1` still
pings successfully, confirming the core functionality is unchanged — only
the filtering has been tightened further.
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20high%20pic%201.png">

## Step 2: View the source

```php
$target = trim($_REQUEST['ip']);

// Set blacklist
$substitutions = array(
    '&'  => '',
    ';'  => '',
    '| ' => '',
    '-'  => '',
    '$'  => '',
    '('  => '',
    ')'  => '',
    '`'  => '',
    '||' => '',
);

// Remove any of the characters in the blacklist (blacklist function)
$target = str_replace( array_keys( $substitutions ), $substitutions, $target );

if( stristr( php_uname('s'), 'Windows NT') ) {
    $cmd = shell_exec( 'ping  ' . $target );
} else {
    $cmd = shell_exec( 'ping  -c 4 ' . $target );
}

echo "<pre>{$cmd}</pre>";
```
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20high%20pic%202.png">

**Why this is still vulnerable:**

- The blacklist is much bigger now, blocking `&`, `;`, `-`, `$`, backticks,
  parentheses, and `||`.
- Crucially, it blocks `'| '` — a pipe **followed by a space** — but not a
  bare pipe with no trailing space.
- Same root problem as Medium: this is still a blacklist trying to guess
  every dangerous pattern, rather than validating that the input is
  actually a well-formed IP address. One missed pattern and the whole thing
  is bypassable again.

## Step 3: Craft and run the payload

Since `'| '` (pipe + space) is blocked but a pipe with no space isn't, we
drop the space:

```
127.0.0.1 |whoami
```

**Result:** the ping output appears as usual, followed by an extra line:

```
www-data
```
<img src="https://github.com/Wisdom2008-star/DVWA-Juiceshop-Homelab/blob/main/Command-Injection/Screenshots/Command%20injection%20high%20pic%20%203.png">

The filter never matches because the exact string `'| '` (with a space)
doesn't appear in our payload — only `|whoami`, which slides through
untouched.

## Takeaway

Even a large, seemingly thorough blacklist can be undone by a single
formatting quirk — here, requiring a space after the pipe character left an
opening. This reinforces the same lesson as Medium, just with a subtler
gap: pattern-matching against "bad" input is fragile, because attackers only
need to find the one variation the list didn't anticipate. The reliable fix
is strict input validation (e.g. `escapeshellarg()` and/or confirming the
input matches a valid IP format) rather than trying to block "bad" input.
