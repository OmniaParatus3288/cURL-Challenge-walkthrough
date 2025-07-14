# 🛰️ cURL Challenge Walkthrough: Downloading the Flag

This document details the process of using `curl` to retrieve a flag from a web server in a Hack The Box (HTB) Academy module, including common errors and troubleshooting steps.

---

## Table of Contents

- [🎯 Objective](#-objective)
- [🔑 Key cURL Flags Used](#-key-curl-flags-used)
- [🚀 The Process & Troubleshooting Journey](#-the-process--troubleshooting-journey)
  - [Step 1: Initial Attempt - Basic Download](#step-1-initial-attempt---basic-download)
  - [Step 2: Following Redirects - Still Not the Flag](#step-2-following-redirects---still-not-the-flag)
  - [Step 3: Connection Failure - "Failed to connect"](#step-3-connection-failure---failed-to-connect)
  - [Step 4: Success - The Flag is Found!](#step-4-success---the-flag-is-found)
- [📝 Key Takeaways](#-key-takeaways)
- [🔗 Resources & References](#-resources--references)

---

## 🎯 Objective

Download a file returned by `/download.php` from a target server. The flag is located within the downloaded file.

---

## 🔑 Key cURL Flags Used

- `-O` (or `--remote-name`) → Saves the fetched resource to a local file named after the remote file.
- `-L` (or `--location`) → Instructs cURL to follow HTTP `3xx` redirects. ([More info](https://everything.curl.dev/usingcurl/followredirects))
- `-v` (or `--verbose`) → Displays detailed request and response info, including headers, useful for debugging.
- `-X` (or `--request`) → Specifies the HTTP method (e.g. GET, POST).

---

## 🚀 The Process & Troubleshooting Journey

---

### Step 1: Initial Attempt - Basic Download

Our first attempt was to download the file directly from the example domain `inlanefreight.com`:

```bash
curl -O http://inlanefreight.com/download.php
```

**Problem Encountered:**  
The file `download.php` was downloaded, but its content was an HTML page indicating a `301 Moved Permanently`.

Example HTML response:

```html
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="https://inlanefreight.com/download.php">here</a>.</p>
</body></html>
```

**Reason:**  
The server was sending an HTTP `301` redirect. By default, `curl` does **not** follow redirects. ([HTTP 3xx status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/3xx))

**Resolution:**  
Use the `-L` flag to follow redirects.

```bash
# Clean up the old file first (optional)
rm download.php

# Retry with -L
curl -L -O http://inlanefreight.com/download.php
```

---

### Step 2: Following Redirects - Still Not the Flag

After adding `-L`, we still didn’t get the flag. Inspecting the new `download.php` revealed a `404 Not Found` page.

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

**Reason:**  
The initial redirect moved us from HTTP to HTTPS. However, in HTB Academy, domains like `inlanefreight.com` are just placeholders. You **must** use your specific lab target’s IP and port. ([HTB Academy Labs](https://academy.hackthebox.com/))

**Resolution:**

1. Identify your target:
   - Check your HTB Academy module page for your unique target IP and port, usually like `10.10.X.X:YYYYY`.

2. Update your cURL command:

```bash
# Clean up the old file
rm download.php

# Replace YOUR_IP_ADDRESS and YOUR_PORT
curl -L -O http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
```

---

### Step 3: Connection Failure - "Failed to connect"

After switching to the correct IP and port, you might see:

```
curl: (7) Failed to connect to YOUR_IP_ADDRESS port YOUR_PORT: Connection refused
```

**Reason:**  
This error means cURL couldn’t establish a network connection—not a `404` or missing file. Possible causes:

- HTB target instance stopped or crashed
- Wrong IP/Port
- VPN disconnected or unstable

**Resolution:**

1. Verify target status in HTB and restart if necessary.
2. Double-check the IP and port.
3. Confirm your VPN is connected. ([HTB VPN Guide](https://help.hackthebox.com/en/articles/5180731-how-to-connect-to-vpn))
4. Optionally, test with ping:

```bash
ping YOUR_IP_ADDRESS
```

---

### Step 4: Success - The Flag is Found!

After resolving connection issues, we used verbose output to see the request/response in detail:

```bash
curl -vL http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
```

Sample key output lines:

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

**Resolution & Flag Retrieval:**

✅ `HTTP/1.1 200 OK` → Success! ([HTTP 200 status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/200))  
✅ `Content-Disposition: attachment; filename="flag.txt"` → Tells you the actual filename. ([Content-Disposition header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition))  
✅ Flag displayed at the end of the verbose output or saved locally if using `-O`.

**To read the flag:**

**Option A:** If you used `-O`:

```bash
cat flag.txt
```

**Option B:** If you used only `-vL` without `-O`:

- Scroll to the end of your curl output. The flag text appears right after all response headers.

---

## 📝 Key Takeaways

✅ Always use `-L` for web requests to handle redirects automatically.  
✅ Never rely on placeholder domains—always use your specific HTB lab target IP and port.  
✅ Use `-v` to debug requests and inspect headers and responses.  
✅ Check `Content-Disposition` headers for true filenames.  
✅ Read all your output! Valuable clues are often hidden in verbose logs.

---

## 🔗 Resources & References

- [Hack The Box Academy](https://academy.hackthebox.com/)  
- [cURL Documentation](https://curl.se/docs/)  
- [cURL Manual - Follow Redirects](https://everything.curl.dev/usingcurl/followredirects)  
- [HTB Help Center - VPN Connection](https://help.hackthebox.com/en/articles/5180731-how-to-connect-to-vpn)  
- [MDN HTTP Status Codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)  
- [MDN Content-Disposition Header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Disposition)

Happy hacking and good luck capturing the flag! 🚩
