---
name: har-net-client-creation
description: Derive a reusable HTTP client from a browser network recording in HAR format. Use when the user wants a client or CLI for a website based on its observed requests.
---

# Create a Client from a HAR Recording

Use an authorized browser network recording to identify the requests needed for the user's client or CLI. Keep raw captures private because they may contain credentials and personal data; use sanitized examples in generated artifacts.

## Source note

https://x.com/thdxr/status/2078727284865827140

dax

@thdxr
used a trick 
@jlongster
 came up with

agents can control browsers but you can also ask it to record network requests into a HAR file

then it can derive a client for any website which is more efficient than browser controlling it every time

made it build a quick uber eats cli

![alt text](image.png)
