# cURL Challenge Walkthrough: Downloading the Flag

This document details the process of using `curl` to retrieve a flag from a web server in a Hack The Box (HTB) Academy module, including common errors and their troubleshooting steps.

## Objective

Download a file returned by `/download.php` from a target server. The flag is located within the downloaded file.

## Key `curl` Flags Used

* `-O` (`--remote-name`): Saves the fetched resource to a local file named after the remote file.
* `-L` (`--location`): Instructs `curl` to follow HTTP 3xx redirects.
* `-v` (`--verbose`): Displays detailed information about the request and response, including headers, which is invaluable for debugging.
* `-X` (`--request`): Specifies the HTTP method to use (e.g., GET, POST).

## The Process & Troubleshooting Journey

### Step 1: Initial Attempt - Basic Download

Our first attempt was to download the file directly using the example domain `inlanefreight.com`.

```bash
curl -O [http://inlanefreight.com/download.php](http://inlanefreight.com/download.php)
Problem Encountered: 
The download.php file was downloaded, but its content was an HTML page indicating 301 Moved Permanently.
HTML
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<h1>Moved Permanently</h1>
<p>The document has moved <a href="[https://inlanefreight.com/download.php](https://inlanefreight.com/download.php)">here</a>.</p>
...
</body></html>
Reason: The server was sending an HTTP 301 (Moved Permanently) redirect. By default, curl does not follow redirects. The HTML content was the server's way of informing us about the redirect.
Resolution: Use the -L flag to tell curl to follow redirects.
# Clean up the old file first (optional, but good practice)
rm download.php

# Retry with -L to follow the redirect
curl -L -O [http://inlanefreight.com/download.php](http://inlanefreight.com/download.php)
Step 2: Following Redirects - Still Not the Flag
After adding -L, we still didn't get the flag. Inspecting the newly downloaded download.php revealed a 404 Not Found HTML page.
HTML
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html>
<head><title>404 Not Found</title></head>
<body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
...
</body></html>
Reason: While curl -L successfully followed the initial redirect (likely from HTTP to HTTPS, as seen in the 301's href), the problem was deeper. In Hack The Box Academy modules, the domain names (inlanefreight.com in this case) are often placeholders. You must use the specific, ephemeral Target IP Address and Port assigned to your lab instance. Connecting to the example domain will almost always result in a 404 or similar issue, as the actual content is served only from your dedicated lab target.
Resolution:
1.	Identify Your Target: Go to your HTB Academy module page and find your unique Target IP Address and Port. These typically look like 10.10.X.X:YYYYY.
2.	Update curl Command: Replace the placeholder domain with your actual target IP and port. It's often best to start with http:// for lab targets unless specifically told otherwise, as https sometimes has certificate issues in lab environments.
Bash
# Clean up the old file
rm download.php

# Replace YOUR_IP_ADDRESS and YOUR_PORT with your actual lab target
curl -L -O http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
Step 3: Connection Failure - "Failed to connect"
After updating to the correct IP and port, the command resulted in "Failed to connect".
curl: (7) Failed to connect to YOUR_IP_ADDRESS port YOUR_PORT: Connection refused
Reason: A "failed to connect" error indicates that curl couldn't establish a network connection to the specified IP and port. This is distinct from a 404 where the server is reached but the resource isn't found. Common reasons include:
•	Target Machine Status: The HTB target instance might have stopped, timed out, or crashed.
•	Incorrect IP/Port: A typo, or the IP/Port changed (they can be dynamic in HTB labs if the machine is restarted).
•	VPN Disconnection: Your VPN connection to the HTB lab environment might have dropped or become unstable.
Resolution:
1.	Verify Target Status: Return to the HTB Academy module page. Ensure your target machine is "Running". If not, start it.
2.	Confirm IP/Port: Double-check that you are using the absolute latest and correct IP address and port provided for your current running instance.
3.	Check VPN: Confirm your HTB VPN (e.g., OpenVPN) is actively connected and stable. Reconnect if necessary.
4.	(Optional) Ping Test: Use ping to quickly check basic network connectivity to the target IP: ping YOUR_IP_ADDRESS.
Step 4: Success - The Flag is Found!
After resolving the connection issues and using the correct target IP and port with -L, we performed a verbose request to see what was happening.
Bash
curl -vL http://YOUR_IP_ADDRESS:YOUR_PORT/download.php
Output Analysis (Key Lines):
* Connected to YOUR_IP_ADDRESS (YOUR_IP_ADDRESS) port YOUR_PORT (#0)
> GET /download.php HTTP/1.1
> Host: YOUR_IP_ADDRESS:YOUR_PORT
> User-Agent: curl/7.88.1
> Accept: */*
>
< HTTP/1.1 200 OK
< Date: Thu, 19 Jun 2025 21:57:28 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Content-Description: File Transfer
< Cache-Control: no-cache, must-revalidate
< Expires: 0
< Content-Disposition: attachment; filename="flag.txt"
< Content-Length: 20
< Content-Type: text
# ... (Followed by the actual content of the file)
Resolution & Flag Retrieval:
1.	HTTP/1.1 200 OK: This is the success code! The server processed the request and found the resource.
2.	Content-Disposition: attachment; filename="flag.txt": This header is crucial. It tells curl (and browsers) that the actual filename for the content being served is flag.txt, even though we requested /download.php.
3.	Content Displayed: Because curl -vL was used without -O, the file's content was printed directly to the standard output (your terminal) after all the verbose header information.
To get the flag:
•	Option A (If you used -O in the successful attempt): The file was likely saved as flag.txt in your current directory. View its content: 
Bash
cat flag.txt
•	Option B (If you used -vL without -O): Scroll through the terminal output of your curl -vL command. The flag (a 20-byte string) will be the text output immediately after all the < (response header) lines.
The flag was successfully found by scrolling to the end of the curl -vL output!
Key Takeaways
•	Always use -L for web requests: Unless you specifically want to analyze redirects manually, always include -L with curl for web content.
•	HTB Target IP/Port is Paramount: Never rely on example domains; always use the specific IP and port assigned to your lab machine.
•	-v is Your Best Friend: The verbose flag (-v) provides invaluable debugging information. It shows you exactly what curl sent and what the server responded with (status codes, headers, etc.).
•	Understand Content-Disposition: This header tells you the intended filename of the content being served, which might differ from the URL you requested.
•	Check All Output: When debugging, read all the output, especially after verbose flags. The answer (or a clue) might be hidden in plain sight.
