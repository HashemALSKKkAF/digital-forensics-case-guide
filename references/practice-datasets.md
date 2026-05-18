# Practice Datasets Reference

Where to actually do the work. Use this for Section 6 ("Practice This Yourself") of the case guide. Match the dataset to what the case is teaching — don't just dump every link.

## Open evidence repositories

### NIST CFReDS (Computer Forensic Reference Datasets)
- **URL**: https://cfreds.nist.gov/
- **What's there**: Disk images, memory dumps, mobile images, network captures. Includes "Hacking Case", "Mobile Device Image", "Memory Images", and more, each with a scenario and questions.
- **Why it matters**: NIST-curated, used in academia, well-documented expected findings. Great for first-time students.

### Digital Corpora (Garfinkel et al.)
- **URL**: https://digitalcorpora.org/
- **Highlights**:
  - **M57-Patents** scenario — multi-week corporate IP theft scenario, multiple machines, network captures
  - **NPS Test Disk Images** — synthetic disks with varied filesystems
  - **govdocs1** — million-document corpus for testing
- **Why it matters**: realistic, multi-machine scenarios. M57 is particularly good for learning timeline correlation across hosts.

### The Honeynet Project Challenges
- **URL**: https://www.honeynet.org/challenges/
- **What's there**: A long-running series of forensic challenges with sample data and write-ups by winners.
- **Why it matters**: real attacker behavior, not synthetic. Solutions are published, so students can compare their findings.

---

## Active CTF / challenge platforms

### CyberDefenders
- **URL**: https://cyberdefenders.org/
- **What's there**: Blue-team / DFIR challenges. Many have downloadable artifacts (memory dumps, disk images, pcaps, eventlogs). "Hawkeye", "TomAndJerry", "Insider", "RedLine" etc.
- **Free vs paid**: most challenges are free; some are gated behind a subscription.
- **Why it matters**: the closest free analog to commercial training. Modern Windows artifacts and current attacker TTPs.

### HackTheBox Sherlocks
- **URL**: https://app.hackthebox.com/sherlocks
- **What's there**: DFIR-specific scenarios with downloadable artifacts and a question-and-answer flag system.
- **Why it matters**: structured progression, current threat scenarios.

### TryHackMe DFIR rooms
- **URL**: https://tryhackme.com/
- **Recommended rooms**: "Volatility", "Autopsy", "Disk Analysis & Autopsy", "Investigating Windows", "Splunk: Boss of the SOC", "MasterMindXX" series.
- **Why it matters**: guided, beginner-friendly, low cost.

### LetsDefend
- **URL**: https://letsdefend.io/
- **What's there**: SOC analyst simulator with investigation challenges; some include forensic artifacts.

### Magnet Forensics CTFs / Weekly CTF
- **URL**: https://www.magnetforensics.com/blog/category/forensic-challenges-and-ctf/
- **What's there**: Magnet runs an annual CTF with public archives, plus the "Magnet Weekly" series. iOS/Android/Windows scenarios.

### Belkasoft CTFs
- **URL**: https://belkasoft.com/ctf
- **What's there**: Periodic CTFs with sample images. Past challenges remain available.

---

## Memory-specific datasets

### Volatility Foundation samples
- **URL**: https://github.com/volatilityfoundation/volatility/wiki/Memory-Samples
- **What's there**: A curated list of public memory images for testing — Cridex, Stuxnet variant, Zeus, etc.
- **Why it matters**: classic malware-in-memory cases. Pair with the Volatility plugin documentation to learn `malfind`, `pslist`, etc.

### MemLabs (stuxnet999)
- **URL**: https://github.com/stuxnet999/MemLabs
- **What's there**: Six progressively harder memory forensics CTF challenges with walkthroughs.

---

## Mobile-specific datasets

### NIST CFReDS Mobile sets
- Several Android and iOS images on cfreds.nist.gov — see "Mobile Device Image".

### Josh Hickman's iOS / Android test images
- **URL**: https://thebinaryhick.blog/ (search "test image")
- **What's there**: Periodically released, well-populated reference images of recent iOS and Android versions. Used as defaults by iLEAPP/ALEAPP test suites.
- **Why it matters**: realistic data populated by the author, matched to current OS versions.

---

## Network / pcap datasets

### Malware-Traffic-Analysis.net (Brad Duncan)
- **URL**: https://www.malware-traffic-analysis.net/
- **What's there**: Hundreds of pcap exercises with infection scenarios. Each has a brief, alerts, the pcap, and answers.
- **Why it matters**: this is where many SOC analysts learn pcap analysis. Outstanding free resource.

### CIC Datasets (University of New Brunswick)
- **URL**: https://www.unb.ca/cic/datasets/
- **What's there**: Labeled datasets for IDS, DDoS, ransomware traffic etc. More research-oriented.

### Wireshark sample captures
- **URL**: https://wiki.wireshark.org/SampleCaptures
- **What's there**: Protocol-by-protocol sample captures. Good for learning how a specific protocol "looks" before you analyze a real case.

---

## Disk image / corporate-scenario datasets

### DFIR Madness (case study)
- **URL**: https://dfirmadness.com/the-stolen-szechuan-sauce/
- **What's there**: "The Stolen Szechuan Sauce" — full corporate intrusion scenario with disk images, memory, pcap, logs across multiple hosts.
- **Why it matters**: cohesive multi-host case, realistic intrusion chain. Excellent capstone-style exercise.

### Ali Hadi — "Case 001 / Case 002"
- **URL**: https://www.ashemery.com/dfir.html
- **What's there**: Free Windows-based forensic challenges with memory + disk images.

### 13Cubed scenarios
- **URL**: https://www.13cubed.com/
- **What's there**: Richard Davis posts walkthrough videos with downloadable images.

---

## Reading and case-study repositories

These don't give you images, but they're how you build context for famous cases.

### About DFIR
- **URL**: https://aboutdfir.com/
- **What's there**: A massive curated resource list — challenges, tools, books, papers, blogs.

### SANS DFIR resources
- **URL**: https://www.sans.org/blog/category/digital-forensics-and-incident-response/
- **What's there**: Blog posts, white papers, case studies. Many SANS poster PDFs (the "Hunt Evil" poster, the "Windows Forensic Analysis" poster) are free and worth printing.

### Court documents (PACER, CourtListener)
- **PACER**: https://pacer.uscourts.gov/ (US federal, pay-per-page)
- **CourtListener (RECAP)**: https://www.courtlistener.com/ (free, often mirrors PACER docs)
- **Why it matters**: indictments and forensic affidavits in famous cases describe exactly what artifacts were collected and how.

---

## How to pick a dataset for a case

| Case type | Pick from |
|---|---|
| Windows endpoint compromise | DFIR Madness Szechuan Sauce, Ali Hadi cases, CyberDefenders "Hawkeye" |
| Memory forensics with malware | MemLabs, Volatility Foundation samples |
| Multi-host corporate scenario | M57-Patents, DFIR Madness |
| Network exfil / C2 | Malware-Traffic-Analysis.net, Honeynet challenges |
| iOS investigation | Josh Hickman iOS test image, CFReDS mobile |
| Android | Hickman Android, ALEAPP test images |
| Live triage practice | Run KAPE on your own VM (no public dataset needed; build your own) |

When you recommend a dataset, give the link AND one specific exercise the student should attempt — not just "go play with this." Example: *"Download the Szechuan Sauce 'DC01' image. First task: identify the initial point of compromise by correlating the EVTX security log with the user's recent files. Don't peek at the writeup until you've got an answer."*
