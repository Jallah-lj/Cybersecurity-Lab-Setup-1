# 📸 Screenshot & Evidence Guide — how to capture proof for every step

> **Why this file exists:** the lab was originally completed on a hosted platform where local screenshots were not possible. This guide makes rebuilding it on a personal machine (or re-capturing evidence) a **30-minute checklist**: exactly which window to open, what must be legible in the frame, the file name to save it as, and how to redact anything sensitive.
>
> Every file name below is the one already referenced by `README.md`, `docs/01-…` and `docs/02-…`. Drop the PNGs into `screenshots/` with these names and the whole repository renders with its evidence in place.

---

## 1 · Ground rules for evidence

1. **One requirement per screenshot.** A grader must be able to point at your image and say “there it is”. Blurry collages lose marks.
2. **Nothing important may be cropped.** Version numbers, IP addresses and checkboxes must be readable at 100% zoom.
3. **No secrets.** Hide real public IPs, hostnames, usernames, e-mail addresses, MAC addresses and serial numbers — **except** the lab addresses (`10.0.0.x`), which must stay visible.
4. **PNG, not JPG.** Text stays sharp.
5. **Same theme, same machine.** Keep the Kali wallpaper and window layout consistent; it reads as one coherent piece of work.
6. **Timestamps are free credibility.** The Kali desktop clock in the corner proves the work was done on your machine.
7. **Never fabricate.** A screenshot of a running VM cannot be faked convincingly — that is the point.

---

## 2 · The tools (pick one, use it consistently)

| OS | Tool | Shortcut | Note |
| --- | --- | --- | --- |
| Windows | **Snipping Tool** | `Win + Shift + S` | rect / window / full screen; also **window mode** = one click |
| Windows | **Print Screen** | `Win + PrtScn` | full screen, saved to `Pictures\Screenshots` |
| Windows | **Xbox Game Bar** | `Win + G` | can capture a *video* of the VM (great for the Wireshark labs) |
| Kali | **Flameshot** | `flameshot gui` (bind `PrtScn`) | annotate arrows/boxes before saving |
| Kali | **GNOME Screenshot / Spectacle** | `PrtScn` | simple region capture |
| Any | **OBS Studio** | — | record the whole lab build as a video, then extract frames |

### Recommended capture recipe for VirtualBox

- Use **windowed** mode (`Host + F` toggles fullscreen) so the VirtualBox title bar — machine name + version — is in frame.
- Maximise the *guest* resolution first so terminal text is large:
  ```bash
  xrandr --output Virtual1 --mode 1920x1080    # or use the View → Virtual Screen menu
  ```
- Raise the terminal font (`Ctrl + Shift + +` in XFCE Terminal) before capturing.
- Capture the **VirtualBox Manager window** itself for settings screenshots — that is where the graded options live (`Settings → Network`, `Shared Folders`, `General → Advanced`).

---

## 3 · Screenshot checklist — WK1-PM1 (lab setup)

Save into `screenshots/`. The middle column is what a grader must be able to read.

| File name | Window to open | Must be legible |
| --- | --- | --- |
| `01-7zip-installed.png` | File Explorer right-click menu, or 7-Zip **About** | 7-Zip present / version |
| `02-virtualbox-version.png` | VirtualBox **Help → About** + terminal `VBoxManage --version` | version string |
| `03-natnetwork-settings.png` | **File → Tools → Network Manager → NAT Networks** | name `NatNetwork`, `10.0.0.0/24`, DHCP ticked, gateway `10.0.0.1` |
| `04a-kali-sha256-check.png` *(optional)* | terminal with `certutil -hashfile` output | hash + file name |
| `05-shared-folder-settings.png` | **Settings → Shared Folders** | `downloads`, path, **Auto-mount ✔**, **Make Permanent ✔** |
| `05b-shared-folder-mounted.png` | Kali terminal `ls -l /media/sf_downloads` | real host files listed |
| `06-clipboard-dragdrop.png` | **Settings → General → Advanced** | **Shared Clipboard: Bidirectional**, **Drag'n'Drop: Bidirectional** |
| `07-network-adapter.png` | **Settings → Network → Adapter 1** | *Attached to:* NAT Network, *Name:* NatNetwork |
| `08-kali-static-ip.png` | Kali terminal `ip -4 addr show` + `ip route` | `inet 10.0.0.2/24`, `default via 10.0.0.1` |
| `08b-kali-ip-gui.png` | Kali **Settings → Network → IPv4 Manual** | same four fields |
| `09-verification-terminal.png` | Kali terminal, the Step 8 block | `ping 8.8.8.8` 0 % loss, `ping kali.org` 0 % loss, `apt update` Done |
| `10-clipboard-bidirectional.png` | host Notepad + Kali terminal | same text visible in both |
| `11-clean-snapshot.png` | VirtualBox **Snapshot Manager** | snapshot name `01-Clean-Kali-Network-Baseline` + description |
| `12-ping-test-target-vm.png` *(optional, Phase 2)* | two terminals | `ping 10.0.0.10` and `ping 10.0.0.2`, both replying |

### The four screenshots that decide the mark

If you can only capture four, capture these — one per written requirement:

1. `03-natnetwork-settings.png` → “NAT Network in `10.0.0.0/24`”
2. `06-clipboard-dragdrop.png` → “clipboard **and** drag/drop enabled”
3. `05-shared-folder-settings.png` → “downloads folder shared”
4. `08-kali-static-ip.png` + `09-verification-terminal.png` → “IP `10.0.0.2/24` and full Internet”

---

## 4 · Screenshot checklist — WK1-PMx (Wireshark / SOC labs)

### ✅ Already captured (real, in `screenshots/`)

| File | Lab | What it proves |
| --- | --- | --- |
| `wireshark-lab-1-attacker-ip.png` | 1 — Identify the Attacker's IP | `RST, ACK` burst, 8 service ports probed, `192.168.10.66` → `192.168.10.20` |
| `wireshark-lab-1-host-services-http-https.png` | 1 — role confirmation | `.20` serves HTTP `80` to `.31` and TLS 1.2 `443` to `.32` |
| `wireshark-lab-2-tcp-stream-chat.png` | 2 — Follow TCP Stream 1 | `tcp.stream eq 5`, plaintext chat with temp password, guest Wi-Fi PSK, ticket |
| `wireshark-lab-2-tcp-stream-index.png` | 2 — stream on the wire | frame 87 `[Stream index: 5]`, custom service on TCP `5000`, client `.45` → server `.20` |
| `wireshark-lab-3-tcp-stream-http-download.png` | 3 — Follow TCP Stream, Part 2 | `tcp.stream eq 11`, `GET /download/report.txt` on `intranet.company.local` |
| `wireshark-lab-4-http-credentials.png` | 4 — HTTP Plain Text Credentials | `tcp.stream eq 4`, login form + `POST /login` urlencoded body |
| `wireshark-lab-5-bruteforce-ftp.png` | 5 — Catch the Brute Force Attacker | `tcp.stream eq 13`, 4 × `530` then `230 Login successful` |
| `wireshark-lab-6-voip-rtp-player.png` | 6 — Catch the Call Scammer | RTP Player, 2 × G.711 PCMU, 24 s call between `.45` and `.52` |

### ⬜ Still to capture (2 minutes per lab)

| File name | Capture | Must show |
| --- | --- | --- |
| `wireshark-lab-1-complete.png` … `wireshark-lab-6-complete.png` | lab page after task 10 | “10/10” / “Lab complete” for **each** of the six labs |
| `local-wireshark-01-follow-stream.png` *(bonus)* | your own Kali Wireshark | proves the skill works outside the browser |

### Optional upgrades to the existing six

| Upgrade | Why |
| --- | --- |
| re-shoot Lab 4 with the **HTML Form URL Encoded** pane expanded | the exact `username=`/`password=` values are the graded answer; the current shot shows the POST but the body is below the fold |
| re-shoot Lab 3 with the exported `report.txt` open next to the stream | proves the object was recovered, not just that it was requested |
| crop or annotate the WhatsApp / watermark overlays | cleaner, more professional for a portfolio |
| add one SIP screenshot (`sip` → `From:` header) for Lab 6 | ties the media stream to a caller identity |

> The lab answers themselves belong in **`docs/02-wireshark-labs-step-by-step.md` §10 answer sheet**, not in the LinkedIn post.
> 🔒 These captures contain training credentials (`Tiger#4821`, `BlueHarbor77`, `Sunshine99`). They are lab values, but state that clearly if you publish them.

---

## 5 · How to take clean screenshots of a VM (step by step)

### Windows host

1. Open the VM, arrange the window you want to prove (e.g. *Settings → Network*).
2. Press **`Win + Shift + S`** → drag a rectangle tightly around the dialog, including its title bar → the snip goes to the clipboard.
3. Click the notification → **Snipping Tool** opens → **Save as** → `screenshots/03-natnetwork-settings.png`.
4. For terminal output that is taller than the screen: `ip a` etc. are short — but if needed, use the terminal's **scrollback** and take two overlapping shots (`…-a.png`, `…-b.png`) rather than shrinking the font.

### Kali guest

```bash
sudo apt install -y flameshot
flameshot gui          # drag region → Ctrl+S to save into ~/Pictures
```

Then move it to the shared folder so it lands on the host immediately:

```bash
cp ~/Pictures/*.png /media/sf_downloads/screenshots/
```

> Both directions of the shared folder are a *great* screenshot on their own: capturing inside Kali and finding the file in `C:\Users\<you>\Downloads` proves the shared folder works.

### Full-screen and window-mode tricks

| Action | Shortcut |
| --- | --- |
| Fullscreen the guest | `Host + F` (Host = right-Ctrl) |
| Toggle scaled mode | `Host + C` |
| Take a snapshot instantly | `Host + T` |
| Open Snapshot Manager | `Host + S` |
| Auto-resize guest display | `Host + G` |

---

## 6 · Redaction (do this before you publish)

| What | Why | How |
| --- | --- | --- |
| Public WAN IP / router address | privacy + it is a real target marker | crop, or black box in Snipping Tool / Flameshot |
| Hostname and username (`C:\Users\jallah\…`) | identifies you | crop the path, or use a generic path in the report |
| MAC addresses | device fingerprinting | black box |
| E-mail addresses in the VM (browser, apt sources) | spam/phishing | box it |
| Real credentials *(lab-only anyway)* | obvious | replace with `Password123` |
| Personal files visible in `Downloads` | embarrassment/leak | put the folder in a clean state first, or use a dedicated `C:\Lab\Downloads` |

**Keep visible:** `10.0.0.x` addresses, the NAT Network settings, the Kali IP, snapshot name. Those are the graded facts.

> The program guide is explicit: never post real IP addresses, real person names or any real information on social media. Lab-only `10.0.0.x` ranges are fine because they are RFC1918 private and describe your own VM.

---

## 7 · Filing and naming

```text
screenshots/
├── 01-7zip-installed.png
├── 02-virtualbox-version.png
├── 03-natnetwork-settings.png
├── 04a-kali-sha256-check.png
├── 05-shared-folder-settings.png
├── 05b-shared-folder-mounted.png
├── 06-clipboard-dragdrop.png
├── 07-network-adapter.png
├── 08-kali-static-ip.png
├── 08b-kali-ip-gui.png
├── 09-verification-terminal.png
├── 10-clipboard-bidirectional.png
├── 11-clean-snapshot.png
├── 12-ping-test-target-vm.png
├── 99-submission-confirmed.png
├── wireshark-lab-1-attacker-ip.png            ← real capture
├── wireshark-lab-1-host-services-http-https.png ← real capture
├── wireshark-lab-2-tcp-stream-chat.png        ← real capture
├── wireshark-lab-2-tcp-stream-index.png       ← real capture
├── wireshark-lab-3-tcp-stream-http-download.png ← real capture
├── wireshark-lab-4-http-credentials.png       ← real capture
├── wireshark-lab-5-bruteforce-ftp.png         ← real capture
├── wireshark-lab-6-voip-rtp-player.png        ← real capture
└── wireshark-lab-N-complete.png               ← still to add (one per lab)
```

**Conventions**

- lower-case, `NN-` prefix = build order → the folder lists in the order the work was done.
- no spaces (use `-`), no dates in the name (dates belong in the README table).
- optional shots get a letter suffix (`04a`, `08b`) so they never renumber the mandatory ones.
- keep every image **under ~2 MB** — GitHub renders faster and the repo stays light:
  ```bash
  # shrink a PNG on Kali
  convert big.png -resize 1600x -strip small.png
  # or on Windows with PowerShell + ImageMagick / or use "Resize" in Snipping Tool
  ```

---

## 8 · Embedding them in the README (already wired up)

The README uses relative paths, so nothing else is needed once the files exist:

```markdown
![](screenshots/03-natnetwork-settings.png)
```

For side-by-side proof use an HTML table:

```html
<table>
  <tr>
    <td><img src="screenshots/08-kali-static-ip.png" alt="Kali IP 10.0.0.2/24"></td>
    <td><img src="screenshots/09-verification-terminal.png" alt="ping and apt update"></td>
  </tr>
  <tr>
    <td align="center"><em>Static IP configured</em></td>
    <td align="center"><em>Internet access verified</em></td>
  </tr>
</table>
```

> ⚠️ **Current state of this repo:** the numbered lab-setup files in `screenshots/` are labelled **placeholders** (the six `wireshark-lab-*.png` are real captures) so every layout/table in the README can be checked before the real evidence is captured. Replace each placeholder with the real capture of the same name — no markdown edits required. Delete `screenshots/README.md` when you are done.

---

## 9 · Quality self-check before submitting

- [ ] Every mandatory file in §3 exists (14 for the lab) and all 8 Wireshark captures + 6 completion screens
- [ ] Each file matches its required content column — open it and re-read
- [ ] No personal data visible; lab `10.0.0.x` addresses still visible
- [ ] Images are PNG, under ~2 MB, readable at 100 % zoom
- [ ] README renders on GitHub with **no broken image icons** (test in a private preview or check the repo page)
- [ ] Snapshot screenshot shows the exact name used in the text
- [ ] The Wireshark “complete” screens are from *your* run of the labs
- [ ] Nothing in the images contradicts the tables in the README (IPs, versions, folder names)

---

<div align="center">

**Back:** [← 02 — Wireshark labs](02-wireshark-labs-step-by-step.md) · **Next:** [04 — GitHub & LinkedIn submission →](04-github-linkedin-submission.md)

</div>
