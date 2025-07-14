# 🛰️ cURL Challenge Walkthrough: Downloading the Flag

Hey there! 👋  

This write-up walks you through how I used `curl` to grab a flag from a web server in a Hack The Box (HTB) Academy module. I’ve included some of the hiccups I hit along the way and how I fixed them—because let’s be real, things rarely work perfectly on the first try!

---

## Table of Contents

- [🎯 Objective](#-objective)
- [🔑 Key cURL Flags Used](#-key-curl-flags-used)
- [🚀 My Process & Troubleshooting Journey](#-my-process--troubleshooting-journey)
  - [Step 1: Basic Download Attempt](#step-1-basic-download-attempt)
  - [Step 2: Following Redirects, but Still No Flag](#step-2-following-redirects-but-still-no-flag)
  - [Step 3: Connection Refused Frustration](#step-3-connection-refused-frustration)
  - [Step 4: Success—Finding the Flag!](#step-4-successfinding-the-flag)
- [📝 Key Takeaways](#-key-takeaways)
- [🔗 Resources & References](#-resources--references)

---

## 🎯 Objective

The mission: download a file served from `/download.php` on a target server. Inside that file hides the flag I’m after!

---

## 🔑 Key cURL Flags Used

Here’s a quick rundown of the cURL options I leaned on:

- `-O` (or `--remote-name`) → Save the fetched file using the remote filename.
- `-L` (or `--location`) → Follow HTTP redirects automatically. ([Learn more](https://everything.curl.dev/usingcurl/followredirects))
- `-v` (or `--verbose`) → Show all the gory details of the HTTP request and response—super handy for debugging.
- `-X` (or `--request`) → Explicitly choose the HTTP method (GET, POST, etc.).

---

## 🚀 My Process & Troubleshooting Journey

---

### Step 1: Basic Download Attempt

I started out by trying to download the file straight from the example domain given in the HTB module:

```bash
curl -O http://inlanefreight.com/download.php
```

**What went wrong:**  
Instead of my precious flag, I ended up with an HTML page saying “301 Moved Permanently.” Boo.

Example of what I got:

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://inlanefreight.com/download.php">here</a>.</p>
</body></html>
```

**Why:**  
The server was sending an HTTP 301 redirect. By default, `curl` doesn’t follow redirects unless you tell it to. ([Read about HTTP 3xx](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/3xx))

**How I fixed it:**  
Added the `-L` flag to make `curl` follow the redirect:

```bash
# Clean up the old file first (optional)
rm download.php

# Retry with -L
curl -L -O http://inlanefreight.com/download.php
```

---

### Step 2: Following Redirects, but Still No Flag

Cool, so the redirect worked… but the downloaded file was now a “404 Not Found” page instead of my flag. Ugh.

Example HTML:

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>404 Not Found</title></head>
<body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
```

**Why:**  
Here’s the catch: domains like `inlanefreight.com` in HTB modules are just placeholders. They don’t actually serve your lab content. You **must** use the unique target IP and port that HTB assigns your specific lab instance. ([HTB Academy Labs](https://academy.hackthebox.com/))

**How I fixed it:**  
1. Found my target’s IP and port on my HTB Academy lab page (it looked something like `10.10.x.x:yyyyy`).

2. Updated my `curl` command:

```bash
# Clean up the old file
rm download.php

# Replace YOUR_IP_ADDRESS and YOUR_PORT
curl -L -O http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
```

---

### Step 3: Connection Refused Frustration

I thought I’d nailed it, but then I hit this lovely message:

```
curl: (7) Failed to connect to YOUR_IP_ADDRESS port YOUR_PORT: Connection refused
```

**Why:**  
This isn’t a “404 file not found.” It means curl couldn’t even connect to the target. Reasons could be:

- Your HTB target instance crashed or timed out
- Typo in your IP or port
- Your VPN connection dropped

**How I fixed it:**

✅ Went back to the HTB Academy page and confirmed my target instance was still running. Restarted it just to be safe.  
✅ Double-checked my IP and port.  
✅ Reconnected my VPN. ([HTB VPN Help](https://help.hackthebox.com/en/articles/5180731-how-to-connect-to-vpn))  
✅ Ran a quick ping test:

```bash
ping YOUR_IP_ADDRESS
```

---

### Step 4: Success—Finding the Flag!

Once everything was up and running, I tried again with verbose output to see exactly what was happening:

```bash
curl -vL http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
```

This time I saw glorious lines like:

```
* Connected to YOUR_IP_ADDRESS (YOUR_IP_ADDRESS) port YOUR_PORT (#0)
> GET /download.php HTTP/1.1
> Host: YOUR_IP_ADDRESS:YOUR_PORT
> User-Agent: curl/7.88.1
> Accept: */*
< HTTP/1.1 200 OK
< Content-Disposition: attachment; filename="flag.txt"
< Content-Length: 20
```

**Victory!** 🎉  

**What it all means:**

- `HTTP/1.1 200 OK` → It worked! ([HTTP 200 status](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200))
- `Content-Disposition: attachment; filename="flag.txt"` → Tells me the real name of the file being downloaded. ([Learn about Content-Disposition](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition))

---

## How to Get Your Flag

✅ **If you used `-O`:**  
Check your current directory for `flag.txt` and read it like this:

```bash
cat flag.txt
```

✅ **If you just ran `curl -vL` without saving the file:**  
Scroll through the verbose output in your terminal—the flag will appear after the headers.

---

## 📝 Key Takeaways

✅ Always add `-L` if your download isn’t working—it’s often a redirect problem.  
✅ Ignore placeholder domains—use your specific HTB lab IP and port.  
✅ Use `-v` when things get weird; it’s your best friend for debugging.  
✅ Look at the `Content-Disposition` header—it can tell you the real filename.  
✅ Don’t panic when stuff breaks. It usually just means you’re one step closer to learning something new!

---

## 🔗 Resources & References

- [Hack The Box Academy](https://academy.hackthebox.com/)  
- [cURL Documentation](https://curl.se/docs/)  
- [cURL Manual - Follow Redirects](https://everything.curl.dev/usingcurl/followredirects)  
- [HTB Help Center - VPN Connection](https://help.hackthebox.com/en/articles/5180731-how-to-connect-to-vpn)  
- [MDN HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)  
- [MDN Content-Disposition Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)

Thanks for reading—and happy hacking! 🚩
