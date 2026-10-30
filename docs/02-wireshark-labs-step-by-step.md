# 🦈 WK1-PMx — Wireshark SOC Labs: Step-by-Step Walkthrough

> **Extra module.** Six browser-based SOC investigations, **10 tasks each**. Every lab hands you a packet capture and asks you to answer questions like a SOC analyst: *who attacked, what was hidden, what password was sent, who is brute-forcing, who is scamming.*
>
> You do **not** need to install Wireshark for the hosted labs — but the same skills are reproduced locally further down, because an analyst who cannot open a `.pcapng` in their own Wireshark is only half-trained.

---

## 0 · Ground rules (read once)

- Work **only** inside the provided labs. Never capture traffic on a network you do not own or have written permission to test.
- Try every task **without hints first**, then use hints.
- Answer format matters: an IP is `10.0.0.5`, not “the attacker” — graders and the lab checker both expect the exact value.
- Write your findings down as you go (lab, task number, answer, evidence). You will need the table at the end.

### The six labs

| # | Lab | Focus | Link |
| --- | --- | --- | --- |
| 1 | Identify the Attacker's IP | endpoint / conversation analysis | <https://networkwalks.com/wireshark-lab3/> |
| 2 | Follow TCP Stream 1 | reassembling a conversation | <https://networkwalks.com/wireshark-lab4/> |
| 3 | Follow TCP Stream, Part 2 | hidden data in a stream | <https://networkwalks.com/wireshark-lab5/> |
| 4 | HTTP Plain Text Credentials | cleartext credentials | <https://networkwalks.com/wireshark-lab6/> |
| 5 | Catch the Brute Force Attacker | repeated auth attempts | <https://networkwalks.com/wireshark-lab7/> |
| 6 | Catch the Call Scammer | VoIP / RTP analysis | <https://networkwalks.com/wireshark-lab8/> |

**Workflow per lab:** open link → read the scenario → run filters → answer 10 tasks → screen-record or screenshot the final “10/10 complete” screen → next lab. After all six, submit at <https://networkwalks.com/submission/> with each lab ticked **only** when all 10 tasks are done.

---

## 1 · The 12 skills these labs actually test (learn these first)

| Skill | What it gives you |
| --- | --- |
| **Statistics → Conversations** | Instant ranking of who talked to whom, how much, on which port |
| **Statistics → Endpoints** | Every IP in the capture with byte/packet counts |
| **`ip.addr == x` filter** | Isolate one host, both directions |
| **`tcp.stream eq N`** | Follow exactly one TCP conversation |
| **Follow → TCP Stream** | Rebuild the application-layer dialogue as text |
| **`http.request.method == "POST"`** | Where logins and uploads live |
| **`http contains "password"`** | Find credentials in cleartext |
| **`ftp` / `telnet`** | Cleartext protocols with visible credentials |
| **`tcp.flags.syn == 1 && tcp.flags.ack == 0`** | Connection attempts — brute force volume |
| **`http.response.code == 401` / `200`** | Failed vs successful authentication |
| **`Statistics → HTTP → Requests`** | Paths hit, counts per URI |
| **`Telephony → RTP → Show Streams` → Play** | Listen to a VoIP call |

### Filter cheat sheet (memorise ~10, look up the rest)

```text
# one host
ip.addr == 10.0.0.5
ip.src == 10.0.0.5
ip.dst == 10.0.0.5

# one conversation
tcp.stream eq 3
udp.stream eq 1

# ports / services
tcp.port == 80
http
dns
ftp
telnet
tls
sip || rtp

# HTTP specifics
http.request
http.request.method == "POST"
http.request.uri contains "login"
http.response.code == 401
http.response.code == 200
http.user_agent contains "sqlmap"
http.host contains "example"
http contains "passwd"

# credentials in cleartext
ftp.request.command == "USER" || ftp.request.command == "PASS"
telnet
http.authorization

# brute force shape
tcp.flags.syn == 1 && tcp.flags.ack == 0
tcp.flags.reset == 1

# malformed / suspicious
http.request.uri contains ".."
frame contains "PK"          # zip magic bytes
dns.qry.name contains "long-random-string"
!(arp || dns || icmp)
```

> 💡 Type `Ctrl+/` in the filter bar to see saved/valid filters, and press the **Expression…** button for a browsable field tree. Green bar = valid syntax, red = error.

---

## 2 · Lab 1 — Identify the Attacker's IP

**Scenario:** traffic from a small network was captured during an attack. Find the source.

**Method — three independent routes, all should agree:**

1. **Conversations** — `Statistics → Conversations` → **IPv4** tab → sort by **Packets**. The noisy outlier is usually the attacker. Note the A → B direction.
2. **Protocol hierarchy** — `Statistics → Protocol Hierarchy`: a scan looks like thousands of TCP SYNs, lots of RSTs, almost no application data.
3. **Filter to confirm** — apply `ip.addr == <candidate>` and inspect what it does:
   ```text
   ip.addr == 10.0.0.5 && tcp.flags.syn == 1 && tcp.flags.ack == 0
   ```
   A single source hammering many destination ports = port scan = the attacker.

**How to tell attacker from victim:**

| Attacker | Victim |
| --- | --- |
| one source → many destinations/ports | one destination → many sources |
| high SYN count, many SYNs without ACK | SYN-ACK replies, RSTs |
| small packets, no application payload | large data payloads, error pages |
| odd/suspicious user-agent, off-hours | normal user-agent, regular traffic |

**Answer shape:** `10.0.0.x` (the constant source of the scan).

> 📌 Record: `Lab 1 — attacker IP = ...; evidence = Statistics → Conversations, sort by packets; filter used = ...`

---

## 3 · Lab 2 — Follow TCP Stream 1

**Scenario:** something was sent inside a TCP session. Rebuild it and read it.

**Method:**

1. Pick the conversation: `Statistics → Conversations → TCP` → sort by **Bytes** → double-click the busiest stream (this applies `tcp.stream eq N` automatically).
2. Right-click any packet of the stream → **Follow → TCP Stream**.
3. In the Follow window:
   - **red** = client → server, **blue** = server → client (defaults are *entire conversation*; switch to “server only” if the payload is easier to read alone).
   - Change **Show data as** from `ASCII` to **`Raw`** or **`Hex Dump`** to spot binary/hidden content.
   - **Save as…** keeps the raw bytes for offline analysis.
4. Read the request/response lines. HTTP shows the request line, headers and the body — that body often holds the answer (a flag, a message, a filename, credentials).

**Shortcut filters:**

```text
tcp.stream eq 0
http.request or http.response
```

**Traps:**

- Long streams may include `Content-Encoding: gzip` — the visible body is then compressed garbage. Export the object instead (`File → Export Objects → HTTP`) and open the file.
- A stream can be split across multiple `tcp.stream` IDs if the connection was reset and reopened — search the payload text globally instead: `frame contains "searchterm"`.

> 📌 Record: `Lab 2 — stream index, what was transmitted, the decoded text.`

---

## 4 · Lab 3 — Follow TCP Stream, Part 2 (hidden data)

**Scenario:** a second capture, data hidden more carefully.

**Method — escalate in this order:**

1. **Follow TCP Stream → Show data as: Hex Dump.** Look for readable ASCII embedded in binary — file headers are your friends:

   | Magic (ASCII) | Magic (hex) | Type |
   | --- | --- | --- |
   | `PK` | `50 4b 03 04` | ZIP / DOCX / XLSX / APK |
   | `%PDF` | `25 50 44 46` | PDF |
   | `GIF8` | `47 49 46 38` | GIF |
   | `\x89PNG` | `89 50 4e 47` | PNG |
   | `MZ` | `4d 5a` | Windows executable |
   | `Rar!` | `52 61 72 21` | RAR |

2. **Export the object** and analyse it properly:
   ```text
   File → Export Objects → HTTP   → select the transfer → Save
   ```
   Then on your own machine: `file <exported>`, `binwalk <exported>`, `unzip -l`, `strings | less`.

3. **Search the payload**, not the packet list: `Ctrl+F` → **Packet bytes** → **String** → search the keyword. Or filter `frame contains "confidential"`.

4. **Check other carriers** if the stream looks empty: `Statistics → Protocol Hierarchy` for `SMTP`, `FTP-DATA`, `SMB2`, `DNS` (data can ride in DNS query names — look for long random subdomains).

5. **Base64 / URL encoding:** if the payload is a wall of `A-Za-z0-9+/=` or `%xx`, decode it:
   ```bash
   echo 'BASE64STRING' | base64 -d
   ```

> 📌 Record: `Lab 3 — stream, transport (HTTP/FTP/SMB/DNS), encoded form, decoded answer.`

---

## 5 · Lab 4 — HTTP Plain Text Credentials

**Scenario:** users logged into a site over plain HTTP. Find the username and password.

**Method:**

1. Find the login transaction:
   ```text
   http.request.method == "POST"
   ```
2. Add `http contains "password"` to jump straight to credential-bearing requests:
   ```text
   http.request.method == "POST" && http contains "password"
   ```
3. Select the POST → in the **packet details pane** expand:
   `Hypertext Transfer Protocol → HTML Form URL Encoded` (or `Line-based text data: application/x-www-form-urlencoded`) → the credentials are right there in `username=...&password=...`.
4. Cross-check with **Follow → TCP Stream** for the full request/response (and to see whether the login succeeded: `HTTP/1.1 302 Found` with a `Set-Cookie` = success).
5. Alternative filter for `GET`-style credentials: `http.request.uri contains "?"`, or `http.authorization` for **HTTP Basic** auth — the header is Base64, decode it:
   ```bash
   echo 'dXNlcjpwYXNz' | base64 -d      # user:pass
   ```

**Also look for:** FTP (`ftp.request.command == "PASS"`), Telnet (`telnet` → read the session), POP3/IMAP/SMTP without TLS, SNMP community strings (`snmp.community`).

**Why this matters (report line):** credentials sent over HTTP/FTP/Telnet are readable by anyone on the path; the fix is TLS everywhere, HSTS, and disabling cleartext protocols.

> 📌 Record: `Lab 4 — username, password, protocol, the exact filter, and whether the request succeeded.`

---

## 6 · Lab 5 — Catch the Brute Force Attacker

**Scenario:** repeated failed logins, one source. Find who is guessing passwords.

**Method — quantify, then attribute:**

1. **Volume by source:** `Statistics → Endpoints → IPv4` → sort by packets. The brute-forcer has a huge packet count towards one port (80/443/22/3389).
2. **Failed responses:** filter the authentication result.
   ```text
   http.response.code == 401
   http.response.code == 403
   ```
   Or for SSH/RDP: `tcp.port == 22` / `tcp.port == 3389` and look at the SYN rate.
3. **Count the attempts** — use the status bar with the filter applied (it prints “Displayed: N”), or:
   ```text
   Statistics → HTTP → Requests     (see which URI repeats hundreds of times)
   Statistics → Conversations → sort by Packets
   ```
4. **Spot the one success:** compare `401` (fail) vs `200`/`302` (success) in `Statistics → HTTP → Load Distribution` or filter `http.response.code == 200 && http.request.uri contains "login"`. A single 200 after 300× 401 = **successful compromise** — that is the finding a SOC reports as critical.
5. **Confirm request shape:** `http.request.method == "POST" && http.request.uri contains "login"`, then check timing: `Statistics → Conversations → TCP` shows duration; ~50 attempts/second is automation, not a human.
6. **Fingerprint the tool** (nice-to-have): `http.user_agent contains "hydra"`, `"sqlmap"`, `"nikto"`, `"python-requests"`, `"curl"`.

**Report-ready numbers to extract:**

| Metric | Where |
| --- | --- |
| attacker IP | Endpoints, highest packet count |
| total attempts | count of `401` responses |
| time window | first and last packet timestamp of the attack stream |
| targeted account | the repeated `username=` value in POST bodies |
| success? | presence of `200`/`302` among the responses |

> 📌 Record: `Lab 5 — attacker IP, attempt count, time window, targeted account, success yes/no.`

---

## 7 · Lab 6 — Catch the Call Scammer

**Scenario:** the capture contains a phone call. Play it and identify the scammer.

**Method:**

1. Enable the VoIP dissectors: **Edit → Preferences → Protocols** → make sure `SIP`, `RTP`, `SDP` are enabled (default on).
2. Find the signalling: filter `sip`. Read `INVITE` / `200 OK` / `BYE` to get caller and callee identities (`From:`, `To:`, `Contact:`, and the SDP block with the RTP media address/port).
3. Find the media: `Telephony → RTP → Show All Streams` (or `RTP → Stream Analysis`).
   - Select the stream (highest packet count / matching the SDP port) → **Analyze** to see jitter, packet loss and the codec (usually `G.711 PCMU/PCMA`).
   - Click **Play Streams** → Wireshark decodes the RTP to an `.au`/`.wav` and plays it in your media player. **Listen** — the scammer's claims and often their number are the answer.
4. If playback fails: **Telephony → VoIP Calls** → select the call → **Play Streams**. If still silent, save the audio (`Save Payload…` / **Save as…**) and open it in VLC/Audacity.
5. Confirm identity details from the SIP headers and from the audio: the name/number the caller claims, the bank they pretend to be from, the callback number, the amount requested.
6. Corroborate with the metadata: `Statistics → Endpoints` → the RTP source IP is the caller's media address.

**Scam indicators to note for the write-up:** urgency, authority impersonation, request for OTP/PIN/credentials, request for gift cards/crypto, spoofed caller ID, callback number mismatch.

> 📌 Record: `Lab 6 — caller identity/number, callee, codec, duration, the scam claim, and the giveaway phrase heard in the audio.`

---

## 8 · Reproduce every skill locally (so it is *your* evidence)

The hosted labs are fine for practice, but a portfolio needs work done on your own tools. Wireshark is pre-installed on Kali — do these five exercises and screenshot them. All traffic is generated by you, on your own machine.

### 8.1 Capture your own traffic (loopback — nothing leaves the machine)

```bash
# terminal 1 — capture
sudo tshark -i lo -w /tmp/demo.pcapng
# or open Wireshark GUI and double-click the "lo" interface

# terminal 2 — generate traffic
curl -s http://neverssl.com > /dev/null
ping -c 3 8.8.8.8
dig kali.org
```

### 8.2 Exercise A — HTTP credentials (local, harmless)

```bash
# a throwaway local HTTP server that logs a POST
python3 -m http.server 8000 &
curl -s -X POST -d 'username=student&password=LabPassword123' http://127.0.0.1:8000/login
```

In Wireshark: filter `http.request.method == "POST"` → expand *HTML Form URL Encoded* → the credentials are in cleartext. **This is exactly what Lab 4 asks you to find.**

### 8.3 Exercise B — Follow TCP Stream

```text
filter: http
right-click a packet → Follow → TCP Stream → Show data as: ASCII / Hex Dump
Save As... → stream.raw
```
Then `strings stream.raw | head` in a terminal.

### 8.4 Exercise C — Detect a “brute force” you create yourself

```bash
for i in $(seq 1 20); do
  curl -s -u "admin:wrong$i" http://127.0.0.1:8000/ > /dev/null
done
```

Filter `http.authorization` (Basic auth) → 20 attempts from one source in a few seconds. Then: `Statistics → Endpoints` to attribute them. **This is Lab 5's pattern in miniature.**

### 8.5 Exercise D — DNS and TLS reconnaissance

```text
dns.qry.name
tls.handshake.extensions_server_name      (SNI = which HTTPS site)
```
Note: with TLS you see the *name* being visited, never the credentials — the perfect “before/after” argument for encryption in your write-up.

### 8.6 Exercise E — Export an object

```text
File → Export Objects → HTTP → select a transfer → Save
file <saved>     # verify the type before opening
```

### 8.7 Command-line equivalents (great for the report — SOCs use these at scale)

```bash
# list packets with key fields
tshark -r demo.pcapng -Y 'http.request.method == "POST"' -T fields -e ip.src -e http.host -e http.file_data

# all IPs and their packet counts
tshark -r demo.pcapng -q -z endpoints,ip

# all conversations
tshark -r demo.pcapng -q -z conv,tcp

# extract HTTP objects to ./out
tshark -r demo.pcapng --export-objects http,out

# all usernames/passwords in cleartext HTTP
tshark -r demo.pcapng -Y 'http contains "password"' -T fields -e ip.src -e text
```

---

## 9 · Captured evidence and findings (all six labs)

The screenshots below were taken during the labs. Each one is also the **worked example** for the method described above — compare your own capture against it.

### Lab 1 — Identify the Attacker's IP

![Lab 1: RST/ACK replies to a port scan, filter ip.src == 192.168.10.20 and ip.dst == 192.168.10.66 and tcp.flags.reset == 1](../screenshots/wireshark-lab-1-attacker-ip.png)

```text
Filter:  ip.src == 192.168.10.20 and ip.dst == 192.168.10.66 and tcp.flags.reset == 1
Displayed: 37 of 309 packets (12.0 %)
```

**What the capture shows.** Frames 88–108 are a burst of `[RST, ACK]` replies, all inside ~110 ms, from `192.168.10.20` to `192.168.10.66`. The *source port* changes on every packet and reads like a service catalogue — `3306` (MySQL), `445` (SMB), `8080` (HTTP-alt), `21` (FTP), `3389` (RDP), `139` (NetBIOS), `25` (SMTP), `5900` (VNC) — while the destination port stays fixed at `5231`.

**How to read it.** A `RST, ACK` with `Seq=0 Ack=1` is a *refusal*: the receiving stack is saying “nothing is listening on that port”. Because the odd port numbers appear as the **source** port of the refusal, the probes were sent **from `192.168.10.66:5231` to those eight ports on `192.168.10.20`** — one source host sweeping many service ports in a fraction of a second is the signature of an automated **port scan**, not of a user browsing.

| Item | Value |
| --- | --- |
| Host that initiated the scan | **`192.168.10.66`** (source port `5231`) |
| Scanned host | **`192.168.10.20`** |
| Ports probed | `21, 25, 139, 445, 3306, 3389, 5900, 8080` |
| Duration of the burst | ~110 ms (frames 88 → 108) |
| Verdict | Automated reconnaissance / port scan |

> 🔎 **Confirm the direction in one filter** (recommended for the report — it shows the SYNs that started it):
> ```text
> ip.src == 192.168.10.66 && ip.dst == 192.168.10.20 && tcp.flags.syn == 1 && tcp.flags.ack == 0
> ```
> If that returns the matching SYN packets, `192.168.10.66` is unambiguously the scanner and the RST filter above is simply the *reply* view of the same scan.

#### Role confirmation — the scanned host is a web server

![Lab 1 extra: ip.src == 192.168.10.20 showing HTTP port 80 to .31 and TLS 1.2 port 443 to .32](../screenshots/wireshark-lab-1-host-services-http-https.png)

```text
Filter:  ip.src == 192.168.10.20
Displayed: 116 of 309 packets (37.5 %)
```

| Frame | Flow | What it is |
| :---: | --- | --- |
| 8 | `.20:80` → `.31:50100` `[SYN, ACK]` `MSS=1460` | the server accepts an HTTP connection |
| 12 | `.20:80` → `.31:50100` `HTTP/1.1 200 OK (text/html)` | page served — in the clear |
| 15 | `.20:80` → `.31:50100` `[FIN, ACK]` | connection closed cleanly |
| 22 | `.20:443` → `.32:50113` `[SYN, ACK]` | the server accepts a **TLS** connection |
| 25 | `.20:443` → `.32:50113` `TLSv1.2 · Server Hello` | handshake only — no readable content |
| 26–27 | `.20:443` → `.32:50113` `Len=1380` `[TCP segment of a reassembled PDU]` | encrypted payload |

**What this adds.** The same host answers on **`80` (HTTP) and `443` (TLS 1.2)** for two different clients, so `192.168.10.20` is confirmed as a **service provider**, not a scanner — which is exactly the role attribution the first screenshot needed. It also shows the contrast the whole module is built on:

| | Port 80 (HTTP) | Port 443 (HTTPS) |
| --- | --- | --- |
| Request / response line | ✅ visible (`GET`, `200 OK`) | ❌ encrypted |
| Hostname / URI | ✅ visible | ⚠️ SNI only |
| Body / credentials | ✅ fully readable | ❌ opaque |
| What an analyst can prove | everything | metadata only |

That is the argument for Lab 4's finding: the portal on port 80 leaks credentials because the traffic is not encrypted.

---

### Lab 2 — Follow TCP Stream 1

![Lab 2: Follow TCP Stream tcp.stream eq 5 showing an internal office chat with passwords](../screenshots/wireshark-lab-2-tcp-stream-chat.png)

```text
Filter:  tcp.stream eq 5
View:    Follow → TCP Stream → Entire conversation → Show data as: ASCII
Footer:  4 client pkts, 5 server pkts, 9 turns.
```

**Recovered conversation (an internal "Office Chat v1.2" session, plaintext, no TLS):**

```text
Welcome to Office Chat v1.2
Ali (Accounts):  Hi, this is Ali from Accounts. I cannot open the shared drive.
Sara (IT):       Hello Ali, this is Sara from IT. Which drive is it?
Ali:             The Finance drive.
Sara:            OK, I reset your access. Your temporary password is Tiger#4821
Ali:             Thanks. Do you also have the guest Wi-Fi password for our visitors?
Sara:            Yes. Guest Wi-Fi password: BlueHarbor77
Ali:             Perfect. Thank you Sara, bye.
Sara:            Bye Ali. Your ticket number is HD-20931.
```

| Item | Value found in the stream |
| --- | --- |
| Chat application | Office Chat v1.2 (custom/plaintext protocol) |
| Users | `Ali` — Accounts · `Sara` — IT |
| Temporary password (Finance drive) | **`Tiger#4821`** |
| Guest Wi-Fi password | **`BlueHarbor77`** |
| Ticket number | **`HD-20931`** |
| Stream | `tcp.stream eq 5` (9 turns) |

**Why this is a finding, not trivia:** credentials and an internal Wi-Fi PSK travelled in clear text over a chat protocol with no encryption. Anyone able to capture that segment (a mirrored port, a rogue AP, an ARP-spoofed host — all normal in this lab) gets working access. **Fix:** TLS on the chat service, a secrets/ticketing workflow instead of pasting passwords in chat, and rotation of both leaked secrets.

#### Stream 5 on the wire — the chat runs on TCP/5000

![Lab 2 extra: tcp.stream eq 5 packet list, frame 87 selected, stream index 5, port 5000](../screenshots/wireshark-lab-2-tcp-stream-index.png)

```text
Filter:   tcp.stream eq 5
Frame 87: 192.168.10.20:5000 → 192.168.10.45:49731   TCP   54 bytes
Details:  Source Port: 5000 · Destination Port: 49731 · [Stream index: 5]
Displayed: 24 of 246 packets (9.8 %)
```

| Frame | Flow | Meaning |
| :---: | --- | --- |
| 81 | `.45:49731` → `.20:5000` `[SYN]` | client opens the connection |
| 82 | `.20:5000` → `.45:49731` `[SYN, ACK]` | server accepts |
| 83 | `.45:49731` → `.20:5000` `[ACK]` | handshake complete |
| 84, 86 | `.45` → `.20` `[PSH, ACK]` `Len=29`, `Len=64` | the client's chat lines |
| 87 | `.20:5000` → `.45:49731` `[ACK]` `Len=0` | pure acknowledgement — **frame 87 is where `[Stream index: 5]` is read** |
| 88 | `.20` → `.45` `[PSH, ACK]` `Len=53` | the server's reply |

**What this adds.** The conversation is a **custom application on TCP port `5000`** — not HTTP, not a known service — which is why no dissector decoded it and why you must use **Follow → TCP Stream** to read it. It also fixes the direction of the conversation: `192.168.10.45` is the **client** (Ali's workstation) and `192.168.10.20` is the **server** running Office Chat (Sara's IT side) — the same host that later appears as the web server in Lab 1 and the intranet server in Lab 3.

> 💡 Port `5000` is worth remembering during a scan: a non-standard listener on a high port is exactly the kind of service that hides from a quick glance at `Statistics → Protocol Hierarchy`.


---

### Lab 3 — Follow TCP Stream, Part 2 (data pulled from the server)

![Lab 3: Follow TCP Stream tcp.stream eq 11 showing GET /download/report.txt from intranet.company.local](../screenshots/wireshark-lab-3-tcp-stream-http-download.png)

```text
Filter:   tcp.stream eq 11
Frame 168: 192.168.10.45:49828 → 192.168.10.20:80   HTTP   GET /download/report.txt HTTP/1.1
                                    192.168.10.20:80 → 192.168.10.45:49828   HTTP/1.1 200 OK (text/plain)
```

**What the capture shows.** A complete, readable HTTP transaction in the packet-details pane:

```http
GET /download/report.txt HTTP/1.1
Host: intranet.company.local
```

```http
HTTP/1.1 200 OK
Content-Type: text/plain
```

| Item | Value |
| --- | --- |
| Client | `192.168.10.45` (source port `49828`) |
| Server | `192.168.10.20` (port `80`) |
| Host / vhost | `intranet.company.local` |
| Resource requested | **`/download/report.txt`** |
| Response | `200 OK`, `text/plain` — the file was delivered |
| Stream | `tcp.stream eq 11` |

**Analyst notes.** The download runs over **plain HTTP on port 80**, so the URI, the hostname and the full body are visible on the wire. The follow-up step for this lab is to reconstruct the payload: right-click the `200 OK` packet → **Follow → TCP Stream**, or **File → Export Objects → HTTP** → save `report.txt` and read it. The MAC addresses (`Intel_…`, `VMware_…`) also mark this as a virtualised lab segment.

---

### Lab 4 — HTTP Plain Text Credentials

![Lab 4: Follow TCP Stream tcp.stream eq 4 showing the login form and the POST /login request](../screenshots/wireshark-lab-4-http-credentials.png)

```text
Filter:   tcp.stream eq 4
Frame:    3 client pkts, 3 server pkts, 6 turns (1372 bytes)
```

**What the capture shows — the classic three-step login exchange:**

```http
GET /login HTTP/1.1
Host: portal.company.local
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/118.0.0.0
```

```http
HTTP/1.1 200 OK
Content-Type: text/html
Server: Apache/2.4.57 (Ubuntu)

<body data-rsssl=1><h1>Staff portal</h1><form method="post" action="/login">
Username <input name="username"> Password <input name="password">
<button>Login</button></form></body>
```

```http
POST /login HTTP/1.1
Host: portal.company.local
Content-Type: application/x-www-form-urlencoded
Content-Length: 36
```

| Item | Value |
| --- | --- |
| Portal | `portal.company.local` — "Staff portal" |
| Client | Windows 10/11, Chrome 118 |
| Server | Apache `2.4.57` on Ubuntu |
| Method | **`POST`** to `/login`, `application/x-www-form-urlencoded` |
| Body length | 36 bytes — a `username=…&password=…` pair |
| Stream | `tcp.stream eq 4` |

**Read the credential values from the packet, not the stream window.** The Follow window above is scrolled to the headers; the form body is in the packet-details pane of the `POST` packet:

```text
Select the POST packet → Hypertext Transfer Protocol
   → HTML Form URL Encoded: application/x-www-form-urlencoded
   → username=...  &  password=...
```

or entirely in one command:

```bash
tshark -r capture.pcapng -Y 'http.request.method == "POST" && http.host contains "portal"' \
       -T fields -e ip.src -e http.file_data
```

**Why it matters:** the password crosses the wire unencrypted. On any shared segment it is readable without touching the server — the single strongest argument for HTTPS + HSTS on internal portals, and for never reusing an internal password elsewhere.

---

### Lab 5 — Catch the Brute Force Attacker

![Lab 5: Follow TCP Stream tcp.stream eq 13 showing repeated FTP USER/PASS attempts until 230 Login successful](../screenshots/wireshark-lab-5-bruteforce-ftp.png)

```text
Filter:   tcp.stream eq 13
Footer:   12 client pkts, 13 server pkts, 25 turns (493 bytes)
```

**Reconstructed session (FTP, cleartext port 21):**

| # | Username | Password tried | Server reply |
| --- | --- | --- | --- |
| 0 | — | — | `530 Login incorrect` *(tail of a previous attempt)* |
| 1 | `admin` | `Winter2025` | `530 Login incorrect` |
| 2 | `admin` | `Company1` | `530 Login incorrect` |
| 3 | `admin` | `Admin@123` | `530 Login incorrect` |
| 4 | `admin` | **`Sunshine99`** | **`230 Login successful`** |
| 5 | `admin` | — | `PWD` → `257 "/" is the current directory` |
| 6 | `admin` | — | `QUIT` → `221 Goodbye` |

```text
USER admin → 331 Please specify the password → PASS <guess> → 530 Login incorrect   (×4)
USER admin → 331 Please specify the password → PASS Sunshine99 → 230 Login successful
PWD → 257 "/" is current directory → QUIT → 221 Goodbye
```

| Item | Value |
| --- | --- |
| Protocol | **FTP** (port 21, no TLS) |
| Targeted account | **`admin`** |
| Attempts | 5 in this stream (`530` × 4, then `230`) |
| Attack succeeded? | **Yes — password `Sunshine99` was accepted** |
| Impact | Interactive shell-level file access as `admin`; `PWD` shows the root directory `/` was reached |
| Stream | `tcp.stream eq 13` |

**Detection logic to report:** a burst of `USER`/`PASS` pairs against one account with `530` responses, followed by a single `230`, is a textbook password-guessing attack that **succeeded**. A SOC would raise this as high severity, force a password reset, hunt for what was transferred after the `230`, and add an FTP/SSH lockout rule (e.g. fail2ban, or an IDS signature on `530` repetition).

**Related filters to show the scale beyond one stream:**

```text
ftp.response.code == 530            # count the failures across the whole capture
ftp.request.command == "USER"       # every account that was targeted
tcp.port == 21 && tcp.flags.syn == 1 && tcp.flags.ack == 0   # connection attempts per second
```

---

### Lab 6 — Catch the Call Scammer

![Lab 6: RTP Player with two G.711 PCMU streams between 192.168.10.45 and 192.168.10.52](../screenshots/wireshark-lab-6-voip-rtp-player.png)

```text
Path:    Telephony → RTP → RTP Player
Codec:   G.711 PCMU · 2 streams · duration 0:24.1 s
Packets: 1051 total, 7 displayed (0.7 %)
```

**What the capture shows — both directions of a phone call:**

| Stream | Path | SSRC |
| --- | --- | --- |
| Caller → callee | `192.168.10.45:16384` → `192.168.10.52:18000` | `0x3A7F21C4` |
| Callee → caller | `192.168.10.52:18000` → `192.168.10.45:16384` | `0x5B9D08E2` |

| Item | Value |
| --- | --- |
| Media protocol | **RTP** over UDP, `G.711 PCMU` (8 kHz, uncompressed telephony audio) |
| Call ends / endpoints | `192.168.10.45` ⇄ `192.168.10.52` |
| Ports | `16384` and `18000` (dynamic RTP pairs) |
| Call duration | ≈ 24 s (`0/24.1 s`) |
| Waveform | four speech bursts separated by silence — a two-party conversation |
| Streams | 2 SSRCs = one leg each way |

**How to identify the scammer.** The waveform only proves *that* a call happened; the identity comes from **(a) the audio** — press **Play** and listen to the claims, the number they give, the "your account is frozen" script — and **(b) the signalling**. For the report, add the SIP leg:

```text
sip                        → From: / To: / Contact: identities, INVITE and BYE
sip.Method == "INVITE"
rtp                        → confirm both media streams belong to that call
```

**Scam indicators to write down while listening:** urgency ("within 30 minutes"), authority impersonation (bank / police / Microsoft), a request for an OTP, PIN, gift card or crypto transfer, and a callback number that differs from the caller ID.

> 📝 **Add your audio findings here** — the exact claims, the number quoted and the giveaway phrase are what the lab marks. The capture above is the evidence that you reached the media; the transcript is the answer.

---

### Evidence still to capture (2 minutes per lab)

Everything in this section is a real capture, but each lab also has a **completion screen** that proves all 10 tasks were answered. Add one screenshot per lab:

```text
screenshots/wireshark-lab-1-complete.png … wireshark-lab-6-complete.png
```

If you want to close the loop on this repository for good, re-run the six labs once more and capture the sequence *filter → answer* for each; the filters used above are the exact ones to reproduce.

---

## 10 · Answer sheet

Filled from the captures in §9. Replace or extend the rows with the exact wording your lab checker expects.

| # | Lab | Question (short) | Answer | Evidence (filter / window) |
| --- | --- | --- | --- | --- |
| 1 | Identify the Attacker's IP | Attacker / scanner IP | `192.168.10.66` (source port `5231`) | `ip.src==192.168.10.20 and ip.dst==192.168.10.66 and tcp.flags.reset==1` |
| 1 | ” | Scanned host | `192.168.10.20` | same filter, destination column |
| 1 | ” | Ports probed | `21, 25, 139, 445, 3306, 3389, 5900, 8080` | source ports in the RST,ACK replies |
| 1 | ” | What happened | automated port scan, ~110 ms | frames 88–108 |
| 2 | Follow TCP Stream 1 | Stream index | `tcp.stream eq 5` | 4 client / 5 server pkts, 9 turns |
| 2 | ” | Temporary password | `Tiger#4821` | Follow → TCP Stream (ASCII) |
| 2 | ” | Guest Wi-Fi password | `BlueHarbor77` | same stream |
| 2 | ” | Ticket number | `HD-20931` | same stream |
| 3 | Follow TCP Stream, Part 2 | Client → server | `192.168.10.45:49828` → `192.168.10.20:80` | `tcp.stream eq 11` |
| 3 | ” | Resource requested | `/download/report.txt` on `intranet.company.local` | frame 168, HTTP GET |
| 3 | ” | Server response | `200 OK`, `text/plain` — file delivered | frame 170 |
| 4 | HTTP Plain Text Credentials | Login endpoint | `POST /login` on `portal.company.local` | `tcp.stream eq 4` |
| 4 | ” | Body type / length | `application/x-www-form-urlencoded`, 36 bytes | POST packet headers |
| 4 | ” | Username / password | *read from the packet body* — expand **HTML Form URL Encoded** | `tshark … -e http.file_data` |
| 4 | ” | Server software | Apache `2.4.57` (Ubuntu) | `200 OK` response headers |
| 5 | Catch the Brute Force Attacker | Protocol / targeted account | FTP (port 21) / `admin` | `tcp.stream eq 13` |
| 5 | ” | Attempts in the stream | 5 (`530` × 4 → `230`) | Follow → TCP Stream |
| 5 | ” | Did it succeed? | **Yes — `Sunshine99` → `230 Login successful`** | same stream |
| 5 | ” | Post-compromise action | `PWD` → `/`, then `QUIT` | same stream |
| 6 | Catch the Call Scammer | Media / codec | RTP over UDP, **G.711 PCMU** | `Telephony → RTP → RTP Player` |
| 6 | ” | Call endpoints | `192.168.10.45:16384` ⇄ `192.168.10.52:18000` | RTP Player, 2 streams |
| 6 | ” | Duration | ≈ 24 s (`0/24.1`) | RTP Player footer |
| 6 | ” | Scammer identity / claims | *from the audio (`Play`) and the SIP `From:` header* | `Telephony → VoIP Calls`, `sip` |

> The three italic cells are the only answers that cannot be read off a screenshot: the credential values (they sit in the POST body pane) and the spoken content of the call. Everything else is proven by the images above.
>
> Keep this table in the repo (or in your notes) — the **same rows** become the “findings” section of your submission and of your LinkedIn post (without IPs).


### Final submission checklist

- [ ] All 6 labs opened and completed (10/10 each)
- [ ] Investigation screenshot captured for each lab (§9 — done)
- [ ] “Lab complete” screen captured for each (`…-complete.png`)
- [ ] Answer sheet filled for every task
- [ ] Ticked all six labs on <https://networkwalks.com/submission/>
- [ ] Local Wireshark exercises A–E done with screenshots (optional but strongly recommended)

---

## 11 · Report-ready findings template

```markdown
### Lab N — <name>
- **Objective:** <what the SOC was asked>
- **Capture:** <file / lab reference>, <packet count>, <time window>
- **Filters used:** `...`
- **Key observation:** <the pattern found>
- **Finding:** <exact answer: IP / credential / stream / number>
- **Impact:** <what an attacker could do with it>
- **Recommendation:** <TLS everywhere / MFA / lockout / IDS rule / user awareness>
```

The *Impact + Recommendation* pair is what turns a homework answer into an analyst's report — and it is what recruiters look for in the GitHub repo.

---

<div align="center">

**Back:** [← 01 — Lab setup step by step](01-lab-setup-step-by-step.md) · **Next:** [03 — Screenshot evidence guide →](03-screenshot-evidence-guide.md)

</div>
