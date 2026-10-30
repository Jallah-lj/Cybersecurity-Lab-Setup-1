# 📁 screenshots/ — evidence

This folder contains **only real captures** from the six Wireshark SOC labs (WK1-PMx). There are no placeholder or mock-up images anywhere in this repository.

## Wireshark / SOC lab evidence (WK1-PMx)

| File | Lab | What it proves |
| --- | --- | --- |
| `wireshark-lab-1-attacker-ip.png` | 1 — Identify the Attacker's IP | `RST, ACK` burst, 8 service ports probed, `192.168.10.66` → `192.168.10.20` in ~110 ms |
| `wireshark-lab-1-host-services-http-https.png` | 1 — role confirmation | `.20` serves HTTP `80` to `.31` (`200 OK text/html`) and TLS 1.2 `443` to `.32` (`Server Hello`) |
| `wireshark-lab-2-tcp-stream-chat.png` | 2 — Follow TCP Stream 1 | `tcp.stream eq 5`, plaintext chat: temp password, guest Wi-Fi PSK, ticket number |
| `wireshark-lab-2-tcp-stream-index.png` | 2 — stream on the wire | frame 87 `[Stream index: 5]`, custom service on TCP `5000`, client `.45` → server `.20` |
| `wireshark-lab-3-tcp-stream-http-download.png` | 3 — Follow TCP Stream, Part 2 | `tcp.stream eq 11`, `GET /download/report.txt` on `intranet.company.local` |
| `wireshark-lab-4-http-credentials.png` | 4 — HTTP Plain Text Credentials | `tcp.stream eq 4`, login form + `POST /login` urlencoded body |
| `wireshark-lab-5-bruteforce-ftp.png` | 5 — Catch the Brute Force Attacker | `tcp.stream eq 13`, 4 × `530` then `230 Login successful` |
| `wireshark-lab-6-voip-rtp-player.png` | 6 — Catch the Call Scammer | RTP Player, 2 × G.711 PCMU, 24 s call between `.45` and `.52` |

Full analysis of each capture: **[../docs/02-wireshark-labs-step-by-step.md §9](../docs/02-wireshark-labs-step-by-step.md#9--captured-evidence-and-findings-all-six-labs)**.

## Why there are no lab-build screenshots

The VirtualBox + Kali build was completed on the program's **hosted lab platform**, which does not permit screenshots from inside the environment. The README proves every requirement instead with the exact setting and the console output it produces — `VBoxManage list natnets`, `ip -4 addr show eth0`, `ping 8.8.8.8`, `ls -l /media/sf_downloads`, and so on.

If you rebuild the lab on your own machine, **[../docs/03-screenshot-evidence-guide.md](../docs/03-screenshot-evidence-guide.md)** lists exactly which windows to capture and what must be legible in each.

> 🔒 Three captures contain training credentials (`Tiger#4821`, `BlueHarbor77`, `Sunshine99`). They are lab values from an isolated capture, so they are safe to publish — but say so if you reuse the images in a social post.
