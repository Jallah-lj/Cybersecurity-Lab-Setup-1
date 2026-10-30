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

## 9 · Answer sheet (fill this in as you go)

| Lab | Task | Question (short) | My answer | Evidence (filter / window) |
| --- | --- | --- | --- | --- |
| 1 | 1 | Attacker IP | | `Statistics → Conversations` |
| 1 | 2 | | | |
| 2 | 1 | Stream index | | `tcp.stream eq N` |
| 2 | 2 | Text recovered | | Follow → TCP Stream |
| 3 | 1 | Transport used | | `Export Objects` |
| 3 | 2 | Hidden data | | Hex dump / `base64 -d` |
| 4 | 1 | Username | | `http.request.method == "POST"` |
| 4 | 2 | Password | | HTML Form URL Encoded |
| 5 | 1 | Attacker IP | | `Statistics → Endpoints` |
| 5 | 2 | Attempt count | | `http.response.code == 401` |
| 6 | 1 | Caller number | | `sip` → `From:` |
| 6 | 2 | Scam claim | | `Telephony → RTP → Play` |

> Keep this table in the repo (or in your notes) — the **same rows** become the “findings” section of your submission and of your LinkedIn post (without real IPs).

### Final submission checklist

- [ ] All 6 labs opened and completed (10/10 each)
- [ ] “Lab complete” screen captured for each
- [ ] Answer sheet filled for every task
- [ ] Ticked all six labs on <https://networkwalks.com/submission/>
- [ ] Local Wireshark exercises A–E done with screenshots (optional but strongly recommended)

---

## 10 · Report-ready findings template

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
