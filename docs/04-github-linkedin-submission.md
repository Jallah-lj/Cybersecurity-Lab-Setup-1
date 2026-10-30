# 📤 Submission Playbook — GitHub, LinkedIn, Troubleshooting & the Form

> The program guide is strict: **work counts only when all four submission steps are done.** This file turns those four steps into a checklist you can finish in one sitting.

| # | Step | Deliverable | Where |
| --- | --- | --- | --- |
| 1 | Repository | README + screenshots | GitHub |
| 2 | Post | short write-up + link in first comment | LinkedIn |
| 3 | Troubleshooting | a problem you hit **and** its fix | LinkedIn / repo / group |
| 4 | Form | name, e-mail, week, both links | networkwalks.com/submission |

**Deadline:** every **Friday, 23:59 Dubai time (UTC+4)**. Late = next week's project is not released to you.

---

## Step 1 — GitHub repository

### 1.1 Repo name (fixed format)

```text
NETWORKWALKS-<YOUR NAME>-<BATCH>-WK1-PM1-CYBERSECURITY-LAB-SETUP
```

Example:

```text
NETWORKWALKS-JALLAH-B0XX-WK1-PM1-CYBERSECURITY-LAB-SETUP
```

Rules: uppercase, `NETWORKWALKS-` prefix, your name, your batch code, `WK1-PM1`, then the project name. Keep the structure of the sample repo (`waqaskarimccie/Cybersecurity-Lab-Setup1`) — README at the root, screenshots in the root or `screenshots/`.

### 1.2 Create and push

**Option A — web UI (no tools needed)**

1. <https://github.com/new> → name as above → **Public** → **Create repository**.
2. **uploading an existing file** → drag `README.md`, the `screenshots/` and `docs/` files in.
3. Commit message: `Week 1 Project — Lab setup (VirtualBox + Kali, 10.0.0.0/24)`.

**Option B — git CLI (recommended, and it is itself a cybersecurity skill)**

```bash
git init
git add README.md docs screenshots
git commit -m "Week 1 PM1: VirtualBox + Kali lab on NAT Network 10.0.0.0/24"
git branch -M main
git remote add origin https://github.com/<you>/NETWORKWALKS-<NAME>-<BATCH>-WK1-PM1-CYBERSECURITY-LAB-SETUP.git
git push -u origin main
```

`VBoxManage`-level reproducibility is a bonus: include the exact commands from `docs/01-lab-setup-step-by-step.md`.

### 1.3 README quality bar

Your README is the graded artifact. It must contain:

- [ ] a **title** and one-line description
- [ ] **objective / requirements** list (mirroring the brief)
- [ ] **architecture diagram** or description of `10.0.0.0/24`
- [ ] **configuration table** (hypervisor, OS, IP, gateway, DNS, RAM, shared folder)
- [ ] **numbered steps** — one heading per requirement, with commands and a screenshot under each
- [ ] **verification table** with commands and expected output
- [ ] **problems & solutions** section (this is submission step 3 — duplicate it here)
- [ ] **what I learned** (5 short, specific points — not generic filler)
- [ ] **ethical use / legal notice**
- [ ] **tools & links**, **author** block

> ❌ Common mark deductions: screenshots without labels, no IP visible anywhere, steps copied verbatim from the PDF with no commands, a README that is only prose, broken image links, repo name not matching the format, private repo (graders cannot open it).

### 1.4 Pre-push check

```bash
git status
git ls-files | sort            # every file intended?
grep -o 'screenshots/[^)]*' README.md | while read f; do [ -f "$f" ] || echo "MISSING: $f"; done
```

The last command prints nothing when all image paths resolve.

---

## Step 2 — LinkedIn post

**Non-negotiables from the guide**

- Tag **@NETWORKWALKS** (company page) and your instructor **@Waqas Karim CCIE**.
- Put the **GitHub link in the FIRST COMMENT**, never in the post body.
- Hashtag: **#Networkwalks**.
- **No IP addresses, no links, no real names** in the post body.
- Keep the format close to the sample post.
- Click **Connect** (not *Follow*) when adding people.

### 2.1 Post template (edit, do not copy blindly)

```text
Week 1 of my Cybersecurity Internship at NETWORKWALKS is done ✅

This week I built my own penetration-testing lab from scratch instead of
borrowing someone else's environment:

🔹 Installed VirtualBox + 7-Zip on my laptop
🔹 Created an isolated NAT Network on a private /24 subnet
🔹 Imported Kali Linux as my attack machine
🔹 Assigned it a static address so my notes and scans stay reproducible
🔹 Enabled bidirectional clipboard + drag & drop, and shared my host
   Downloads folder with the VM
🔹 Verified gateway, DNS, Internet access and package updates inside the VM
🔹 Took a clean snapshot so I can roll back in seconds
🔹 Completed 6 Wireshark SOC labs: attacker IP hunting, TCP stream
   reconstruction, cleartext credentials, brute-force detection and a
   VoIP call-scam investigation

Biggest lesson: NAT is not NAT Network. Only a NAT Network lets my future
target VMs talk to each other while still reaching the Internet — that
single detail is what makes a lab a lab.

Everything was done on virtual machines I own, in an isolated environment,
for learning and defensive research only.

Thank you @Waqas Karim CCIE for the guidance and @NETWORKWALKS for the
structure 🙌

#Networkwalks #Cybersecurity #EthicalHacking #KaliLinux #VirtualBox
#Wireshark #SOC #PenetrationTesting #Linux #InfoSec
```

### 2.2 After posting

1. **First comment** (posted by you, immediately):
   ```text
   GitHub repository with the full step-by-step documentation and screenshots: <your repo URL>
   ```
2. Edit the post if a tag did not resolve — LinkedIn tags only work when you pick the name from the dropdown; plain text `@name` does nothing.
3. Optional but strong: add 2–3 screenshots directly to the post (VM settings + verification terminal) — engagement rises sharply, and it does not violate the “no IPs” rule because `10.0.0.x` is a private lab range. If in doubt, blur it.
4. Set the post to **Public** (a “Connections only” post cannot be graded).

### 2.3 Tagging checklist

- [ ] `@NETWORKWALKS` resolved as the **company page** (grey building icon)
- [ ] `@Waqas Karim CCIE` resolved as the **person**
- [ ] repo URL only in the first comment
- [ ] `#Networkwalks` present
- [ ] no IPs / no external links in the body
- [ ] post is public

---

## Step 3 — Share your troubleshooting experience

The guide asks for **a real problem and how you solved it** — this is the part that helps other interns and is read closely.

Format that works:

```text
Problem: After setting a static IP in Kali, ping 8.8.8.8 timed out even
though ip a showed 10.0.0.2/24.

What I checked, in order:
1. The NAT Network existed and was named exactly as in the adapter settings.
2. The adapter was attached to "NAT Network", not to plain "NAT".
3. Gateway was 10.0.0.1 and no other VM held 10.0.0.2.

What fixed it:
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
sudo nmcli connection down "Wired connection 1"
sudo nmcli connection up "Wired connection 1"

Reason: on VirtualBox 7.x with recent Kali images, NetworkManager's
duplicate-address detection keeps the interface in a tentative state for a
moment after the address is applied, and the connection profile was still
holding stale settings. Disabling DAD and cycling the profile brought the
link up immediately.

If you did not hit this: check your gateway first — an incorrect gateway is
the most common cause of "no Internet" in this lab.
```

Where to put it:

- a **`# Problems Encountered & Solutions`** section in `README.md` (already present, expand with your own experience)
- a **comment on the week-1 YouTube video** (<https://www.youtube.com/watch?v=K6rMQdg10Zk>) — that is where the instructor answers questions
- optionally a short LinkedIn post of its own

> If you genuinely had no problems, the guide's fallback applies: **help others in the comments** and note that you did.

---

## Step 4 — Submit the form

1. Open <https://networkwalks.com/submission/>.
2. Fill in:
   - **Name** (exactly as registered)
   - **E-mail** (the one used for the program)
   - **Week** → `Week 1`
   - **GitHub link** → your repo URL
   - **LinkedIn link** → the **post** URL (not your profile, not the repo)
3. Tick the labs you finished (all six Wireshark labs, if you did them).
4. **Submit** → screenshot the confirmation page (`screenshots/99-submission-confirmed.png`).
5. Only form submissions count — a WhatsApp message or a DM is not a submission.

```text
Repo   : https://github.com/<you>/NETWORKWALKS-<NAME>-<BATCH>-WK1-PM1-CYBERSECURITY-LAB-SETUP
Post   : https://www.linkedin.com/posts/<your-post-slug>
Week   : 1
```

---

## Results, resubmission and scoring

- Results: <https://networkwalks.com/results/> (video guide: <https://www.youtube.com/watch?v=DUyy3rIX-es>)
- You may **resubmit** as many times as needed before the weekly deadline — the highest/latest mark counts.
- Final score = Week 1 + Week 2 + Week 3 + Week 4.
- All four weeks submitted properly → certificate eligibility, stipend eligibility for top performers, experience letter / LOR, feature on official pages.

---

## Final pre-submission checklist

**Repository**
- [ ] name follows `NETWORKWALKS-<NAME>-<BATCH>-WK1-PM1-CYBERSECURITY-LAB-SETUP`
- [ ] **public**
- [ ] README has objective, config table, numbered steps, verification, problems, learnings, ethics, author
- [ ] screenshots present, named, readable, redacted; no broken images
- [ ] all 16 lab + 12 Wireshark evidence files in place *(or the subset you can honestly produce)*

**LinkedIn**
- [ ] post public, tags resolved, `#Networkwalks` present
- [ ] repo link in **first comment only**
- [ ] no IPs / links / real names in the body
- [ ] 2–3 screenshots attached (optional)

**Troubleshooting**
- [ ] one concrete problem + root cause + fix written up
- [ ] also posted as a YouTube comment on the week-1 video

**Form**
- [ ] all fields correct, week = 1, both links verified by opening them in a private window
- [ ] confirmation screenshotted

> 💡 **Last sanity test:** open both links in an incognito/private browser window. If you can read your README and see your post while logged out, the grader can too.

---

<div align="center">

**Back:** [← 03 — Screenshot evidence guide](03-screenshot-evidence-guide.md) · **Home:** [README](../README.md)

</div>
