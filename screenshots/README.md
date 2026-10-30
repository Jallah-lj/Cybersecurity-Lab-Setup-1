# 📁 screenshots/ — evidence folder

⚠️ **The images in this folder are PLACEHOLDERS.** Each one is a labelled card describing exactly what the real screenshot must show. They exist so the repository layout, tables and image links in the main `README.md` can be reviewed before the real evidence is captured.

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

## Wireshark / SOC lab evidence (WK1-PMx)

| File | Proves |
| --- | --- |
| `wireshark-lab3-01-conversations.png` | Lab 1 — attacker IP identified |
| `wireshark-lab4-01-tcp-stream.png` | Lab 2 — TCP stream reconstructed |
| `wireshark-lab5-01-hidden-data.png` | Lab 3 — hidden data recovered |
| `wireshark-lab6-01-http-credentials.png` | Lab 4 — cleartext credentials |
| `wireshark-lab7-01-bruteforce.png` | Lab 5 — brute-force source identified |
| `wireshark-lab8-01-voip-play.png` | Lab 6 — call scammer identified |
| `99-submission-confirmed.png` | submission form confirmation |

> For each Wireshark lab also keep one `…-02-complete.png` screenshot of the lab's **“10/10 complete”** state — that is what proves the tasks were finished, not just attempted.
