# 🪜 WK1-PM1 — Complete Step-by-Step Lab Build (VirtualBox + Kali Linux)

> **Who this is for:** anyone who has to reproduce the Week 1 lab from a clean Windows laptop, and anyone who already did it on a hosted platform and needs to rebuild it on their own machine to capture evidence.
>
> **Time needed:** ~60–90 minutes (most of it is the Kali download).
> **Result:** Kali Linux at `10.0.0.2/24` on a `10.0.0.0/24` NAT Network, with Internet access, bidirectional clipboard, drag-and-drop, a shared `downloads` folder and a clean snapshot.

**Task requirements checklist (from the official WK1-PM1 brief):**

- [ ] VirtualBox as the base (latest version)
- [ ] Kali Linux as the attacker machine
- [ ] Network subnet `10.0.0.0/24`
- [ ] `NATNetwork` type network
- [ ] Clipboard **and** drag-and-drop enabled in the VM settings
- [ ] Shared folder with the host's **downloads** folder
- [ ] Kali IP **`10.0.0.2/24`**
- [ ] Kali has **full Internet access**

---

## 📑 Table of contents

1. [Before you start (host preparation)](#1--before-you-start-host-preparation)
2. [Step 1 — Install 7-Zip](#step-1--install-7-zip)
3. [Step 2 — Install VirtualBox](#step-2--install-virtualbox)
4. [Step 3 — Create the NAT Network `10.0.0.0/24`](#step-3--create-the-nat-network-1000024)
5. [Step 4 — Download, verify, extract and import Kali](#step-4--download-verify-extract-and-import-kali)
6. [Step 5 — Clipboard, drag-and-drop and shared folder](#step-5--clipboard-drag-and-drop-and-shared-folder)
7. [Step 6 — Attach the VM to the NAT Network](#step-6--attach-the-vm-to-the-nat-network)
8. [Step 7 — Configure the static IP `10.0.0.2/24`](#step-7--configure-the-static-ip-1000224)
9. [Step 8 — Verify everything](#step-8--verify-everything)
10. [Step 9 — Take the clean snapshot](#step-9--take-the-clean-snapshot)
11. [Step 10 (optional, Phase 2) — Add a target VM](#step-10-optional-phase-2--add-a-target-vm)
12. [Troubleshooting matrix](#-troubleshooting-matrix)
13. [Snapshot / rollback procedure](#-snapshot--rollback-procedure)
14. [Evidence to capture (screenshot slots)](#-evidence-to-capture)

---

## 1 · Before you start (host preparation)

### 1.1 Hardware

| Resource | Minimum | Recommended |
| --- | --- | --- |
| RAM | 8 GB | 16 GB |
| Disk free | 40 GB | 60 GB+ (SSD) |
| CPU | Core i3 / any 64-bit with VT-x or AMD-V | i5/i7, 4 cores+ |

### 1.2 Enable hardware virtualization (do this first — it is the #1 setup blocker)

1. Press `Ctrl+Shift+Esc` → **Performance** → **CPU** → look at **Virtualization**.
2. If it says **Enabled** → continue.
3. If it says **Disabled**: reboot → press `F2` / `F10` / `Del` / `Esc` (vendor dependent) at the boot logo → enable **Intel Virtualization Technology (VT-x)** or **AMD-V / SVM Mode** → `F10` to save → reboot.
4. On Windows, also check **Windows Features**: `Hyper-V`, **Virtual Machine Platform**, **Windows Subsystem for Linux** and **Core Isolation → Memory Integrity** all hold the virtualization extensions. If the VM refuses to start with `VERR_VMX_NO_VMX`, disable them and reboot.

### 1.3 Record your host details (you will need them for the report)

```cmd
systeminfo | findstr /C:"OS Name" /C:"Total Physical Memory"
wmic cpu get name
```

Write down: host OS + version, RAM, CPU. These go into the *Lab Configuration* table of the README.

---

## Step 1 — Install 7-Zip

**Why:** the official Kali VirtualBox image is published as a `.7z` archive and Windows cannot extract it natively.

1. Open <https://7-zip.org/download.html>.
2. Under *Download*, click **64-bit x64** (`.exe`, ~1.5 MB).
3. Run the installer → **Install** → **Close**.
4. Verify: right-click the Desktop → the **7-Zip** submenu appears.

**Expected result:** `C:\Program Files\7-Zip\7zFM.exe` exists and the shell context menu has a 7-Zip entry.

> ⬛ **SCREENSHOT #01:** the 7-Zip context menu (or the *About* window showing the version).
> Save as `screenshots/01-7zip-installed.png`.

---

## Step 2 — Install VirtualBox

**Why:** VirtualBox is the hypervisor — the layer that lets a full operating system run as a program.

1. Open <https://virtualbox.org/wiki/Downloads>.
2. Click **Windows hosts** under *VirtualBox 7.x.x platform packages*.
3. Run the `.exe`:
   - **Next** → **Next** (default features) → **Yes** on the *Network Interfaces* warning (VirtualBox installs a virtual adapter — this is expected and required).
   - Accept the Oracle driver security prompts → **Install**.
4. Leave **Launch VirtualBox** ticked → **Finish**.
5. *(Optional)* Also download and install the **Oracle VM VirtualBox Extension Pack** of the same version — needed only for USB 2.0/3.0 passthrough and VRDP. Not required for this task.
6. Verify from a **new** Command Prompt:

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" --version
```

```text
7.1.6r167084
```

**Expected result:** the version prints without error. `VBoxManage` will be used for several checks later.

> ⬛ **SCREENSHOT #02:** VirtualBox main window with **Help → About VirtualBox** open, showing the version (and, ideally, the `VBoxManage --version` output in a terminal next to it).
> Save as `screenshots/02-virtualbox-version.png`.

> 💡 If the installer fails with *"Installation failed! Error: ... rollback"*, reboot and run the installer **as Administrator**; a partially failed previous install must be removed from *Apps & Features* first.

---

## Step 3 — Create the NAT Network `10.0.0.0/24`

### 3.1 Understand the choice (this is a marked requirement)

| Mode | VMs can reach each other? | VMs can reach Internet? | Internet can reach VMs? |
| --- | :---: | :---: | :---: |
| **NAT** (per-VM) | ❌ no | ✅ yes | ❌ no |
| **NAT Network** (shared) | ✅ yes | ✅ yes | ❌ no |
| **Bridged** | ✅ yes | ✅ yes | ⚠️ exposes the lab to the real LAN |
| **Host-only** | ✅ yes | ❌ no | ❌ no |
| **Internal Network** | ✅ yes | ❌ no | ❌ no |

A cybersecurity lab needs **VM ↔ VM** communication *and* Internet for tool updates, without exposing anything to the real network. That is exactly **NAT Network**.

### 3.2 Create it

1. VirtualBox main window → **File → Tools → Network Manager** (`Ctrl+H`).
2. Open the **NAT Networks** tab → click **Create** (green ➕).
3. Set:

```text
Name:             NatNetwork
IPv4 Prefix:      10.0.0.0/24
IPv4 Prefix, DHCP: ✔ Enable DHCP
IPv6 Prefix:      (leave blank → IPv6 disabled)
```

4. Click the network → **Properties** / edit, and confirm:
   - **Gateway / server address:** `10.0.0.1` (VirtualBox picks the first host address automatically)
   - **DHCP range:** starts at `10.0.0.3`. If the UI lets you edit it, set the **lower bound to `10.0.0.3`** (or higher) so DHCP can never hand `10.0.0.2` to another VM — that address is reserved for Kali.
5. **Apply → Close**.

### 3.3 Verify

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" list natnets
```

```text
NetworkName:    NatNetwork
IP:             10.0.0.1
Network:        10.0.0.0/24
IPV6 Enabled:   No
Enabled:        Yes
Dhcp Enabled:   Yes
```

Alternative check: `ipconfig` on the host shows an adapter whose description contains *VirtualBox Host-Only* / *NAT* — the NAT Network itself does not appear on the host, which is correct.

> ⬛ **SCREENSHOT #03:** the Network Manager → **NAT Networks** tab with `NatNetwork`, `10.0.0.0/24` and DHCP visible.
> Save as `screenshots/03-natnetwork-settings.png`.

---

## Step 4 — Download, verify, extract and import Kali

### 4.1 Download

1. Open <https://kali.org/get-kali> → tab **Virtual Machines**.
2. Choose **VirtualBox 64-bit** → download the `.7z` (2025.x / 2026.x, ~3 GB).
3. On the same page copy the **SHA256SUMS** value for your file.

### 4.2 Verify the download (professional habit — do not skip)

```powershell
certutil -hashfile "C:\Users\<you>\Downloads\kali-linux-2026.1-virtualbox-amd64.7z" SHA256
```

Compare the printed hash with the one on kali.org. **They must match character for character.** A corrupt image produces the weirdest VM failures later.

> ⬛ **SCREENSHOT #04a (optional):** the hash comparison in a terminal.
> Save as `screenshots/04a-kali-sha256-check.png`.

### 4.3 Extract

1. Right-click the `.7z` → **7-Zip → Extract to "kali-linux-…\"**.
2. You now have a folder containing (names approximate):

```text
kali-linux-2026.1-virtualbox-amd64/
├── Kali-Linux-2026.1-virtualbox-amd64.vbox
├── Kali-Linux-2026.1-virtualbox-amd64.vdi     ← the virtual disk (large)
└── Kali-Linux-2026.1-virtualbox-amd64.vbox-prev (sometimes)
```

> ⚠️ **Do not rename, move or delete the `.vdi`.** The `.vbox` file is a small XML descriptor that points to it. Move the *folder* as a unit.

### 4.4 Import / register

- **Method A (fastest):** double-click the `.vbox` file → VirtualBox opens and the machine appears in the list.
- **Method B:** VirtualBox → **Machine → Add…** → select the `.vbox` → **Open**.
- **If you were given an `.ova` instead:** **File → Import Appliance…** → select the `.ova` → **Next → Import**.

### 4.5 Tune the VM (Settings, VM selected & powered off)

| Section | Setting | Value |
| --- | --- | --- |
| System → Motherboard | Base Memory | `2048` MB (4096 if you have 16 GB) |
| System → Processor | CPUs | `2` |
| Display | Video Memory | `128` MB (or default `VMSVGA`) |
| Display | Graphics Controller | `VMSVGA` |
| Audio | — | disable (not needed, saves CPU) |

### 4.6 First boot

1. Select the VM → **Start**.
2. Login: **`kali` / `kali`** (pre-built image default).
3. Immediately change it: `passwd` → set a strong password.

```bash
passwd
```

**Expected result:** a Kali XFCE desktop, and the top-right network icon shows a wired connection.

---

## Step 5 — Clipboard, drag-and-drop and shared folder

### 5.1 Enable clipboard + drag-and-drop (marked requirement)

1. **Shut down the guest** (`sudo poweroff` or *Machine → Close → Power off*). The menu options are greyed out while the VM runs.
2. VM → **Settings → General → Advanced**:

```text
Shared Clipboard:  Bidirectional
Drag'n'Drop:       Bidirectional
```

3. **Start** the VM.

`Bidirectional` is what the brief asks for: text copies host → Kali *and* Kali → host; files can be dragged in both directions.

### 5.2 Confirm Guest Additions are present (the hidden dependency)

Everything in this step — clipboard, drag-and-drop, auto-mount, screen resize — is provided by **VirtualBox Guest Additions**. They ship pre-installed in the official Kali VirtualBox image.

```bash
lsmod | grep -i vbox
```

```text
vboxguest             425984  1
vboxsf                 90112  1
vboxvideo              ...
```

If nothing appears, install them:

```bash
sudo apt update
sudo apt install -y virtualbox-guest-x11 virtualbox-guest-utils
sudo reboot
```

*(If you use the Kali **installer ISO** instead of the pre-built image, mount the Guest Additions CD from the VM menu: **Devices → Insert Guest Additions CD image**, then run `sudo /media/cdrom/VBoxLinuxAdditions.run`.)*

### 5.3 Create the shared folder

1. Power the VM **off**.
2. VM → **Settings → Shared Folders** → click the **➕** icon.

```text
Folder Path:     C:\Users\<you>\Downloads      ← pick your real Downloads folder
Folder Name:     downloads                     ← this becomes /media/sf_downloads
☑ Read-only:     leave UNCHECKED               (need bidirectional transfer)
☑ Auto-mount:    checked
☑ Make Permanent: checked
```

3. **OK** → start the VM.

### 5.4 Grant the guest access to it

```bash
sudo usermod -aG vboxsf $USER
```

Log out and back in (or `sudo reboot`) — group membership is only read at login.

```bash
ls -l /media/sf_downloads
groups | grep vboxsf
```

**Test it in both directions:**

```bash
# host -> guest (visible immediately)
echo "hello from Kali" | sudo tee /media/sf_downloads/from-kali.txt
# now look in C:\Users\<you>\Downloads on the host: from-kali.txt is there
```

> 💡 If `sudo` is required to write into the folder, your user is still not in `vboxsf`, or *Read-only* is ticked.

> ⬛ **SCREENSHOT #05:** Settings → **Shared Folders** panel showing `downloads`, **auto-mount** ✔, **make permanent** ✔ — plus (better) a second terminal showing `ls -l /media/sf_downloads` listing real host files.
> Save as `screenshots/05-shared-folder-settings.png` and `screenshots/05b-shared-folder-mounted.png`.

> ⬛ **SCREENSHOT #06:** *Settings → General → Advanced* with **Shared Clipboard: Bidirectional** and **Drag'n'Drop: Bidirectional**.
> Save as `screenshots/06-clipboard-dragdrop.png`.

---

## Step 6 — Attach the VM to the NAT Network

VM **powered off** → **Settings → Network → Adapter 1**:

```text
☑ Enable Network Adapter
Attached to:   NAT Network
Name:          NatNetwork
Advanced
  Adapter Type:      Intel PRO/1000 MT Desktop
  Promiscuous Mode:  Allow All      (lets a future VM sniff traffic — useful later)
  Cable Connected:   ✔
```

**Expected result:** inside Kali, `ip link` shows `eth0` as `state UP`.

> ⬛ **SCREENSHOT #07:** the *Network → Adapter 1* panel showing **NAT Network / NatNetwork**.
> Save as `screenshots/07-network-adapter.png`.

---

## Step 7 — Configure the static IP `10.0.0.2/24`

### 7.1 Find the connection profile name

```bash
nmcli connection show
```

```text
NAME                UUID                                  TYPE      DEVICE
Wired connection 1  9c8f...                                ethernet  eth0
```

`Wired connection 1` is the profile name used in every command below. **If yours differs, substitute it.**

### 7.2 Configure it (recommended: CLI — repeatable and screenshot-friendly)

```bash
sudo nmcli connection modify "Wired connection 1" \
  ipv4.method manual \
  ipv4.addresses 10.0.0.2/24 \
  ipv4.gateway 10.0.0.1 \
  ipv4.dns "8.8.8.8,1.1.1.1" \
  ipv4.dad-timeout 0
```

Apply it:

```bash
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up   "Wired connection 1"
```

### 7.3 GUI alternative

**Kali menu → Settings → Network → Wired → ⚙ (gear) → IPv4 tab → Method: Manual → Add:**

```text
Address:   10.0.0.2
Netmask:   255.255.255.0
Gateway:   10.0.0.1
DNS:       8.8.8.8, 1.1.1.1
```

→ **Apply** → toggle the wired connection off/on in the top-right menu.

### 7.4 Verify the address

```bash
ip -4 addr show
ip route
nmcli -f ipv4.addresses,ipv4.gateway,ipv4.dns,ipv4.dad-timeout connection show "Wired connection 1"
```

```text
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    inet 10.0.0.2/24 brd 10.0.0.255 scope global eth0

default via 10.0.0.1 dev eth0 proto static metric 100
```

**Expected result:** `10.0.0.2/24` on `eth0`, default route through `10.0.0.1`, DNS `8.8.8.8 1.1.1.1`.

> 💡 `ip addr add 10.0.0.2/24 dev eth0` also "works" but is **not persistent** — it vanishes on reboot and is not what a professional report should show.
>
> 💡 **DHCP conflict check:** if `ip a` shows a second address such as `10.0.0.5/24`, another VM or the DHCP server took `.2` first. Fix in Network Manager, or shut down the other VM, then re-apply.

> ⬛ **SCREENSHOT #08:** terminal with `ip -4 addr show` and `ip route` output clearly showing `10.0.0.2/24` and `10.0.0.1`.
> Save as `screenshots/08-kali-static-ip.png`.
> ⬛ **SCREENSHOT #08b:** the Kali GUI *Network → IPv4 Manual* panel with the same values (shows both methods).
> Save as `screenshots/08b-kali-ip-gui.png`.

---

## Step 8 — Verify everything

Run the block below **in this order** and capture one combined screenshot. Each line maps to a requirement in the brief.

```bash
# 1. address + mask
ip -4 addr show eth0

# 2. route / gateway
ip route

# 3. gateway reachable (VirtualBox NAT)
ping -c 4 10.0.0.1

# 4. Internet by IP (proves NAT translation)
ping -c 4 8.8.8.8

# 5. Internet by name (proves DNS)
ping -c 4 kali.org

# 6. DNS configuration
resolvectl status | head -20        # or: cat /etc/resolv.conf

# 7. package repositories reachable
sudo apt update | tail -5

# 8. tooling present
nmap --version | head -1
wireshark --version | head -1

# 9. shared folder mounted with real host content
ls -l /media/sf_downloads

# 10. guest additions loaded (clipboard / drag&drop support)
lsmod | grep -c vbox
```

Expected summary:

| Check | Expected |
| --- | --- |
| `ip -4 addr` | `inet 10.0.0.2/24` |
| `ip route` | `default via 10.0.0.1` |
| `ping 10.0.0.1` | `0% packet loss` |
| `ping 8.8.8.8` | `0% packet loss` |
| `ping kali.org` | `0% packet loss` |
| `resolvectl status` | `8.8.8.8`, `1.1.1.1` |
| `apt update` | `Reading package lists... Done` with no `Could not resolve` |
| `nmap --version` | `Nmap version 7.9x` |
| `ls /media/sf_downloads` | your host Download files |

> ⬛ **SCREENSHOT #09:** this verification block, with the important lines legible (do it in two terminal tabs if it is too long — `09a-network-verification.png`, `09b-internet-and-dns.png`).
> Save as `screenshots/09-verification-terminal.png`.

> ⬛ **SCREENSHOT #10 (clipboard proof):** host Notepad containing a line of text, and the same text pasted into a Kali terminal via `Ctrl+Shift+V`. Two screenshots side by side prove *bidirectional*.
> Save as `screenshots/10-clipboard-bidirectional.png`.

---

## Step 9 — Take the clean snapshot

Snapshots capture disk + memory state so you can return to this exact working lab.

**GUI:** VM running (or off) → **Machine → Take Snapshot…** (`Host + T`, i.e. `Right-Ctrl + T`)

```text
Name:         01-Clean-Kali-Network-Baseline
Description:  Kali 2026.1 — static 10.0.0.2/24 on NatNetwork 10.0.0.0/24,
              gateway 10.0.0.1, DNS 8.8.8.8/1.1.1.1, Guest Additions OK,
              clipboard+drag&drop bidirectional, /media/sf_downloads auto-mounted,
              Internet verified (ping 8.8.8.8 + kali.org), apt update OK.
```

**CLI equivalent (from the host):**

```cmd
"C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" snapshot "Kali-Linux-2026.1-virtualbox-amd64" take "01-Clean-Kali-Network-Baseline" --description "Static 10.0.0.2/24 on NatNetwork, GA OK, shared folder mounted, Internet verified"
```

Verify: **Snapshot Manager** (`Host + S`) lists the snapshot.

> ⬛ **SCREENSHOT #11:** Snapshot Manager showing `01-Clean-Kali-Network-Baseline` with the description visible.
> Save as `screenshots/11-clean-snapshot.png`.

> 💡 **Name two snapshots per project:** one *clean baseline* before risky work, and one *post-exercise* checkpoint. Restore, never "repair".

---

## Step 10 (optional, Phase 2) — Add a target VM

Only if you plan to continue into scanning/exploitation modules. The brief marks this **optional**.

1. Download a Windows 10/11 evaluation ISO (Microsoft Evaluation Center) or a vulnerable-by-design VM (Metasploitable, DVWA, VulnHub) — never a licensed Windows you do not own.
2. **New** → type/version → attach the ISO → install.
3. Networking: **Settings → Network → Adapter 1 → NAT Network → NatNetwork**, identical to Kali.
4. Static IP in the same subnet, e.g. `10.0.0.10/24`, gateway `10.0.0.1`, **no DNS needed** (or `8.8.8.8`).
5. Disable Windows Defender **in the isolated lab only**, host firewall sharing off, and never bridge this VM to your real LAN.
6. Ping test both directions and snapshot:

```bash
# from Kali
ping -c 4 10.0.0.10
```

```cmd
:: from the Windows target
ping 10.0.0.2
```

Suggested addressing plan (from the program diagram):

| Machine | IP | Purpose |
| --- | --- | --- |
| Kali Linux | `10.0.0.2/24` | Attacker / analyst |
| Ubuntu / Server | `10.0.0.7/24` | target |
| Windows Server 2016 | `10.0.0.9/24` | target |
| Windows 10 | `10.0.0.10/24` | target |
| Android 9x | `10.0.0.11/24` | target |
| Windows 7 | `10.0.0.16/24` | target |

> ⬛ **SCREENSHOT #12 (optional):** `ping` between Kali and the target VM, both directions.
> Save as `screenshots/12-ping-test-target-vm.png`.

---

## 🩺 Troubleshooting matrix

Work top to bottom — these are ordered by how often they cause the failure.

### A. Kali has no Internet

| # | Check | Command / place | Fix |
| --- | --- | --- | --- |
| 1 | NAT Network exists and is named exactly `NatNetwork` | `VBoxManage list natnets` | re-create it (Step 3) |
| 2 | Adapter attached to **NAT Network**, not NAT | Settings → Network | change *Attached to* |
| 3 | Correct IP / mask / gateway | `ip a`, `ip route` | re-apply Step 7 |
| 4 | Gateway really is `10.0.0.1` | `ip route` | if `10.0.0.1` fails, the brief's own note says try `10.0.0.1` as gateway — if you had entered something else, switch back to `10.0.0.1` |
| 5 | Nobody else holds `10.0.0.2` | `ip a` shows one address only | shut down the conflicting VM / raise DHCP lower bound |
| 6 | **DAD timeout bug** (VirtualBox 7 + Kali 2025.1/2026.x) | — | `sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0` then `down`/`up` |
| 7 | Restart | — | reboot Kali, then reboot the host |
| 8 | Ask the community | project video comments | read pinned comments — many students post the same fix |

```bash
# The three commands from the official brief:
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up   "Wired connection 1"
```

### B. Internet works, DNS does not (`ping 8.8.8.8` OK, `ping kali.org` fails)

```bash
cat /etc/resolv.conf
resolvectl status | head
```

- Empty `resolv.conf` → the profile has no DNS. Set it: `sudo nmcli connection modify "Wired connection 1" ipv4.dns "8.8.8.8,1.1.1.1"` and re-apply.
- `resolv.conf` managed by `systemd-resolved` and pointing at `127.0.0.53` → that is normal; check `resolvectl status` for the upstream servers instead.
- Test bypass: `dig @8.8.8.8 kali.org +short` — if this resolves, it is purely a DNS config problem, not connectivity.

### C. Clipboard / drag-and-drop do nothing

1. Were they set while the VM was **off**? (Yes → correct.)
2. Guest Additions loaded? `lsmod | grep vbox` → must show `vboxguest` and `vboxsf`.
3. Is the VM menu item *Devices → Shared Clipboard* set to **Bidirectional** too? (Per-VM setting can override.)
4. Drag-and-drop additionally requires a running desktop session and does **not** work from a Wayland session in some versions — log in with the XFCE/X11 session.
5. Last resort: reinstall Guest Additions in the guest (`sudo apt install --reinstall virtualbox-guest-x11`) and reboot.

### D. `/media/sf_downloads` missing

| Symptom | Cause | Fix |
| --- | --- | --- |
| Folder `sf_*` absent | user not in `vboxsf` | `sudo usermod -aG vboxsf $USER` + re-login |
| Folder absent after group fix | `vboxsf` module not loaded / GA missing | install Guest Additions, reboot |
| `Permission denied` on write | *Read-only* ticked in the shared folder dialog | untick and restart |
| Folder exists but empty | host folder actually empty | put a test file in `C:\Users\<you>\Downloads` |

### E. VM will not start

| Error | Meaning | Fix |
| --- | --- | --- |
| `VERR_VMX_NO_VMX`, “VT-x is disabled” | virtualization off / held by Hyper-V | enable in BIOS, disable Hyper-V + Memory Integrity |
| `VERR_VMX_MSR_LOCKED_OR_DISABLED` | Hyper-V/VBS holding VT-x | `bcdedit /set hypervisorlaunchtype off` (Admin) + reboot |
| `E_FAIL (0x80004005)` | corrupted VM state | restore snapshot; or `VBoxManage modifyvm <vm> --long-mode` checks |
| Black screen after boot | graphics controller / video memory | Display → VMSVGA, 128 MB; boot with `nomodeset` |
| Kali boots to shell (no GUI) | XFCE crash | `startxfce4`; check `~/.xsession-errors` |
| “Cannot register the hard disk … already exists” | stale UUID from a copied `.vdi` | `VBoxManage internalcommands sethduuid "<path>.vdi"` |

### F. Everything is extremely slow

- Give the VM 2 CPUs and ≥ 2 GB RAM, but never give it more than ~half of the host's cores.
- Ensure the VM disk is on an SSD.
- Disable 3D acceleration if the host GPU driver is unstable; keep audio disabled.
- Close browsers on the host while Kali boots.

---

## ♻️ Snapshot / rollback procedure

**Restore the baseline before every new exercise:**

1. VM powered off → **Machine → Snapshots** (`Host + S`).
2. Select `01-Clean-Kali-Network-Baseline` → **Restore** → confirm.
3. Start the VM and re-verify `ip a` shows `10.0.0.2/24`.

**Delete a snapshot only when you no longer need the rollback point** — deleting it merges the differencing disk and cannot be undone.

---

## 📸 Evidence to capture

The complete, submission-ready evidence plan (file names, what must be visible in each shot, how to take them on Windows/Kali, and how to redact sensitive data) is in **[03-screenshot-evidence-guide.md](03-screenshot-evidence-guide.md)**.

Minimum set for WK1-PM1:

| # | File | Shows |
| --- | --- | --- |
| 01 | `01-7zip-installed.png` | 7-Zip present |
| 02 | `02-virtualbox-version.png` | VirtualBox version |
| 03 | `03-natnetwork-settings.png` | NatNetwork `10.0.0.0/24`, DHCP |
| 05 | `05-shared-folder-settings.png` | `downloads`, auto-mount |
| 05b | `05b-shared-folder-mounted.png` | `/media/sf_downloads` with host files |
| 06 | `06-clipboard-dragdrop.png` | both **Bidirectional** |
| 07 | `07-network-adapter.png` | NAT Network / NatNetwork |
| 08 | `08-kali-static-ip.png` | `10.0.0.2/24` + gateway |
| 09 | `09-verification-terminal.png` | ping 8.8.8.8, ping kali.org, apt update |
| 10 | `10-clipboard-bidirectional.png` | paste both ways |
| 11 | `11-clean-snapshot.png` | snapshot name + description |

---

<div align="center">

**Next:** [02 — Wireshark SOC labs step by step →](02-wireshark-labs-step-by-step.md)

</div>
