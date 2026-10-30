# 📁 screenshots/ — evidence folder

⚠️ **The numbered lab-setup images in this folder are PLACEHOLDERS** (the six `wireshark-lab-*.png` files are real captures). Each one is a labelled card describing exactly what the real screenshot must show. They exist so the repository layout, tables and image links in the main `README.md` can be reviewed before the real evidence is captured.

**To finish the repository:** rebuild the lab on your own machine (or re-capture the hosted-lab screens), then **overwrite each file with your real PNG using the same file name**. No markdown edits are needed anywhere.

```text
Replace  screenshots/03-natnetwork-settings.png
with     your real capture of Network Manager → NAT Networks
keep     the same file name
```

Full capture checklist, capture tools, redaction rules and quality self-check: **[../docs/03-screenshot-evidence-guide.md](../docs/03-screenshot-evidence-guide.md)**

---

## Mandatory evidence (WK1-PM1)

| File | Proves |
| --- | --- |
| `00-host-virtualization-enabled.png` | hardware virtualization is enabled *(optional)* |
| `01-7zip-installed.png` | 7-Zip installed |
| `02-virtualbox-version.png` | VirtualBox version |
| `03-natnetwork-settings.png` | NAT Network `10.0.0.0/24`, DHCP, gateway `10.0.0.1` |
| `04-kali-imported.png` | Kali imported into VirtualBox |
| `04a-kali-sha256-check.png` | download integrity *(optional)* |
| `05-shared-folder-settings.png` | `downloads` shared, auto-mount + permanent |
| `05b-shared-folder-mounted.png` | `/media/sf_downloads` mounted in Kali |
| `06-clipboard-dragdrop.png` | clipboard **and** drag-and-drop = Bidirectional |
| `07-network-adapter.png` | adapter attached to NAT Network `NatNetwork` |
| `08-kali-static-ip.png` | `10.0.0.2/24` + default gateway `10.0.0.1` |
| `08b-kali-ip-gui.png` | same values set through the GUI |
| `09-verification-terminal.png` | gateway, Internet, DNS, `apt update`, nmap |
| `10-clipboard-bidirectional.png` | clipboard works both directions |
| `11-clean-snapshot.png` | snapshot `01-Clean-Kali-Network-Baseline` |
| `12-ping-test-target-vm.png` | optional Phase-2 target VM reachable |

## Wireshark / SOC lab evidence (WK1-PMx) — ✅ real captures

| File | Proves |
| --- | --- |
| `wireshark-lab-1-attacker-ip.png` | Lab 1 — RST/ACK burst from the scanner, 8 service ports probed |
| `wireshark-lab-2-tcp-stream-chat.png` | Lab 2 — `tcp.stream eq 5` plaintext chat with passwords |
| `wireshark-lab-3-tcp-stream-http-download.png` | Lab 3 — `tcp.stream eq 11`, `GET /download/report.txt` |
| `wireshark-lab-4-http-credentials.png` | Lab 4 — `tcp.stream eq 4`, login form + `POST /login` |
| `wireshark-lab-5-bruteforce-ftp.png` | Lab 5 — `tcp.stream eq 13`, FTP guesses → `230 Login successful` |
| `wireshark-lab-6-voip-rtp-player.png` | Lab 6 — RTP Player, two G.711 PCMU streams |
| `99-submission-confirmed.png` | submission form confirmation *(placeholder)* |

> Still missing: one **completion screen** per lab — `wireshark-lab-1-complete.png` … `wireshark-lab-6-complete.png` — showing all 10 tasks answered. Nothing else is needed for the Wireshark module.

> 🔒 **Before publishing:** three screenshots contain credential-like strings (`Tiger#4821`, `BlueHarbor77`, `Sunshine99`). They belong to a training capture, so they are safe to publish — but if you reuse these images in a public post, say clearly that they come from an isolated lab capture and are not production secrets.

