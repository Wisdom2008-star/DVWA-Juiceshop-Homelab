# Command Injection — Medium

## Setup

- Target: DVWA running locally (`http://localhost:8080/vulnerabilities/exec/`)
- Security level: **Medium**
- Environment: Kali Linux (VirtualBox), Firefox with Burp Suite/FoxyProxy

## Step 1: Baseline test

Same form as Low — "Ping a device" with a single IP input. Submitting a
normal IP like `127.0.0.1` still works exactly as expected, producing a
standard ping result. This tells us the underlying ping functionality
hasn't changed — only the filtering around it has.

## Step 2: View the source

```php
$target = $_REQUEST['ip'];

// Set blacklist
$substitutions = array(
    '&&' => '',
    ';'  => '',
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

**Why this is still vulnerable:**

- This time there's an actual filter — a **blacklist** that strips out `&&`
  and `;` from the input before it's used.
- The problem is the filter is naive: it only removes those two specific
  strings, once, and doesn't touch any other shell separator.
- The pipe character `|` isn't blacklisted at all, so it still works exactly
  like it did on Low.
- Blacklisting specific "bad" strings instead of only allowing known-good
  input (a whitelist / strict validation) is a common and fragile approach —
  it's easy to forget a character, and this is a textbook example.

## Step 3: Craft and run the payload

Since `&&` and `;` get stripped, we swap to the pipe character instead:

```
127.0.0.1 | whoami
```

**Result:** the page shows the normal ping output, followed by an extra
line:

```
www-data
```

Exactly like on Low, `whoami` runs successfully and reveals the web server's
user. The blacklist did nothing to stop this because `|` was never on its
list.

## Takeaway

A blacklist that blocks specific characters or substrings is only as strong
as the list itself — miss one separator (here, `|`) and the whole defense is
bypassed. This is why blacklisting is generally considered a weak approach
to input sanitization compared to strict whitelisting or proper input
validation (e.g. confirming the input actually matches the shape of a valid
IP address before using it at all).
