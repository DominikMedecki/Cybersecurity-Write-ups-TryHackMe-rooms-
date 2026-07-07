# ♟️ TryHackMe - Fool's Mate Write-up

> **Room:** Fool's Mate  
> **Platform:** TryHackMe  
> **Difficulty:** Easy  
> **Category:** Web Exploitation | Client-Side Security

---

## 📖 Overview

This room demonstrates the dangers of relying on **client-side validation** for application security.

The web application is a chess endgame trainer that prevents the player from making a checkmate move through JavaScript. However, because the validation only exists in the browser, it can be bypassed by sending requests directly to the backend API.

---

## 🎯 Objectives

- Perform reconnaissance on the target
- Enumerate the web application
- Analyze the client-side JavaScript
- Identify the security flaw
- Bypass the frontend validation
- Capture the flag

---

# 🔍 Reconnaissance

I began by scanning the target using **Nmap**.

```bash
nmap -sC -sV -A <target-ip>
```

### Results

```
22/tcp open  ssh   OpenSSH 9.6p1 Ubuntu
80/tcp open  http  Node.js Express Framework
```

The scan revealed:

- SSH running on port **22**
- A **Node.js Express** web application running on port **80**

### Nmap Output

![Nmap Scan](images/nmap.png)

---

# 🌐 Web Enumeration

Next, I fingerprinted the web application.

```bash
whatweb http://<target-ip>
```

The scan identified the application as running on:

- Node.js
- Express
- JavaScript frontend

Opening the website revealed a chess puzzle named **Endgame Trainer**.

---

# 🔎 Source Code Analysis

Inspecting the JavaScript source revealed the following code:

```JavaScript
if (probe.isCheckmate()) {
    showSystemNotice("I'll shut down your PC if you play that.");
    return false;
}
```

The application prevented the player from making a winning move by checking for checkmate **inside the browser**.

This immediately suggested that:

- The validation occurs only on the client.
- The backend likely trusts whatever move is submitted.
- Sending requests directly to the server should bypass the restriction.

### Application Screenshot

![Endgame Trainer](images/Application.png)

---

# ⚡ Exploitation

Since the browser was blocking the move—not the server—I bypassed the frontend entirely by sending my own request.

The winning move was moving the rook from **a1** to **a8**.

Example request:

```bash
curl -X POST http://<target-ip>/move \
-H "Content-Type: application/json" \
-d '{
    "from":"a1",
    "to":"a8"
}'
```

Because the backend performed no server-side validation, the move was accepted.

---

# 🚩 Flag

The server responded with:

```json
{
    "status": "checkmate",
    "winner": "white",
    "flag": "THM{cl13nt_s1d3_ch3ckm4t3}"
}
```

---

# 🧠 Lessons Learned

This room demonstrates a common web security vulnerability:

> **Never trust client-side validation.**

JavaScript exists solely for improving the user experience and **must never be relied upon for enforcing security controls**.

An attacker can easily:

- Disable JavaScript
- Modify browser code
- Intercept requests
- Send requests directly with tools like:
  - `curl`
  - Burp Suite
  - Postman

Security-sensitive validation should **always** occur on the server.

---

# 🛠️ Tools Used

- Nmap
- WhatWeb
- Browser Developer Tools
- curl

---

# 📚 Key Takeaways

- Perform reconnaissance before interacting with an application.
- Always inspect client-side JavaScript.
- Client-side restrictions can often be bypassed.
- Server-side validation is essential for application security.
- Never trust data received from the client.
---

## Disclaimer

This write-up is intended for educational purposes only as part of the **TryHackMe** platform. All actions were performed inside a legal, controlled lab environment.

