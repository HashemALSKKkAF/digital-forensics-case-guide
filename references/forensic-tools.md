# Forensic Tools Reference

Tool catalog organized by investigation phase. Use this when populating Section 4 ("Tools Used") of the case guide. Bias toward tools the student can actually run (free/open source); commercial tools are listed because students need to know them by name.

## Table of contents
- [Acquisition (imaging)](#acquisition-imaging)
- [Hashing & integrity](#hashing--integrity)
- [Disk / filesystem analysis](#disk--filesystem-analysis)
- [Memory analysis](#memory-analysis)
- [Network / pcap analysis](#network--pcap-analysis)
- [Mobile device analysis](#mobile-device-analysis)
- [Browser artifacts](#browser-artifacts)
- [Registry analysis (Windows)](#registry-analysis-windows)
- [Event log analysis (Windows)](#event-log-analysis-windows)
- [Timeline construction](#timeline-construction)
- [File carving](#file-carving)
- [Triage / live response](#triage--live-response)
- [Specialty / niche](#specialty--niche)

---

## Acquisition (imaging)

| Tool | Open source | Notes |
|---|---|---|
| **FTK Imager** | Free (closed-source) | Industry standard for Windows. Creates E01/raw, supports memory capture. |
| **Guymager** | OS | Linux GUI imager. Fast, multi-threaded, produces E01/AFF/dd. |
| **dd / dcfldd / dc3dd** | OS | CLI imaging. `dc3dd` adds hashing and progress output (DoD Cyber Crime Center fork). |
| **EWF tools** (`ewfacquire`) | OS | libewf-based. Creates E01 from CLI on Linux. |
| **KAPE** (Kroll Artifact Parser and Extractor) | Free | Targeted live triage — pulls specific artifacts from a running system fast. Critical for IR. |
| **Velociraptor** | OS | Live response and remote acquisition at scale. VQL-based queries. |
| **AVML** (Microsoft) | OS | Linux memory acquisition. |
| **LiME** | OS | Linux memory acquisition (LKM). |
| **MAGNET RAM Capture** | Free | Free Windows memory dumper. |

**Key principle**: always use a hardware or software write blocker on disk source media. Never image a suspect drive read/write — first thing the defense will ask is "did you alter the evidence."

---

## Hashing & integrity

- `md5sum`, `sha1sum`, `sha256sum` — built into every Linux distro
- `Get-FileHash` (PowerShell)
- `hashdeep` / `md5deep` — recursive hashing of directory trees
- `ssdeep` — fuzzy hashing (similarity, not equality — useful for finding "near-duplicate" files an attacker modified slightly)

**Why hash?** Two reasons: (1) verify the image didn't change between acquisition and analysis, (2) compare files against known-good or known-bad hash sets (NSRL, VirusTotal).

NSRL (National Software Reference Library): https://www.nist.gov/itl/ssd/software-quality-group/national-software-reference-library-nsrl — hash database of "known" files. Filtering NSRL hashes from a large image dramatically reduces what you need to look at.

---

## Disk / filesystem analysis

| Tool | License | Strengths |
|---|---|---|
| **Autopsy** + **Sleuth Kit** | OS | Free, full-featured, Windows-friendly GUI on top of TSK. Good first tool for students. |
| **The Sleuth Kit (TSK)** CLI | OS | `mmls` (partition layout), `fls` (file list), `icat` (inode dump), `istat` (inode metadata). |
| **FTK** (AccessData) | Commercial | Heavy enterprise tool. Indexed search across huge datasets. |
| **EnCase** (OpenText) | Commercial | The legal gold standard in many jurisdictions. EnScript scripting language. |
| **X-Ways Forensics** | Commercial | Power-user favorite. Extremely fast, low-level hex view, scripting. |
| **Magnet AXIOM** | Commercial | Strong for mobile + cloud + computer combined. Good UI for non-experts. |
| **OSForensics** (PassMark) | Commercial | Mid-priced, popular in academia. |

Common Sleuth Kit workflow on a raw image:
```bash
mmls disk.dd                          # show partitions
fls -r -m / -o 2048 disk.dd > body    # walk filesystem starting at offset 2048
mactime -b body > timeline.csv        # convert to MAC timeline
icat -o 2048 disk.dd 12345 > file     # extract by inode
```

---

## Memory analysis

- **Volatility 3** — open source, the standard. `vol -f mem.raw windows.pslist`, `windows.netscan`, `windows.malfind`, `windows.cmdline`. Linux/macOS profiles also supported.
- **Volatility 2** — older, but still has plugins not yet in v3.
- **Rekall** — Volatility fork (now largely dormant but historically important).
- **MemProcFS** — mounts a memory image as a virtual filesystem. Browse processes as folders. Massive UX win.
- **Redline** (FireEye/Mandiant) — free GUI memory analysis for Windows. Less powerful than Volatility but gentler learning curve.
- **WinPmem / OSXPMem / LinPmem** — acquisition counterparts.

Common Volatility plugins to mention:
- `pslist` / `pstree` — running processes
- `cmdline` — process command lines
- `netscan` — network connections
- `malfind` — injected/hidden code
- `dlllist` — loaded DLLs per process
- `handles` — open handles (files, registry keys, mutexes)
- `filescan` / `dumpfiles` — files in memory
- `hivescan` / `hivelist` / `hashdump` — registry hives in memory + extract account hashes

---

## Network / pcap analysis

- **Wireshark** — packet inspection GUI. Display filters and "Follow stream" are the bread and butter.
- **tshark** — Wireshark's CLI. Scriptable. `tshark -r file.pcap -Y 'http.request' -T fields -e ip.src -e http.host`
- **NetworkMiner** — free; reassembles files, extracts credentials, builds host inventory automatically.
- **Zeek** (formerly Bro) — produces structured logs from pcap (`conn.log`, `dns.log`, `http.log`, `ssl.log`, `files.log`). Great for large captures.
- **Suricata** — IDS that can also process pcap with rules; outputs eve.json.
- **tcpdump** — minimal CLI capture/filter.
- **Arkime** (formerly Moloch) — large-scale pcap indexing for SOCs.

For files exfiltrated over the wire: NetworkMiner reassembles automatically, or use Wireshark "File > Export Objects > HTTP/SMB/FTP/etc."

---

## Mobile device analysis

| Tool | iOS | Android | Notes |
|---|---|---|---|
| **Cellebrite UFED** | ✓ | ✓ | Industry standard. Closed source, expensive, used by LE. |
| **Magnet AXIOM** | ✓ | ✓ | Friendly UI, integrates with computer/cloud cases. |
| **MOBILedit Forensic** | ✓ | ✓ | Mid-tier commercial. |
| **iLEAPP** | ✓ | — | OS, Python. Parses iOS backups/extractions. The free standard. |
| **ALEAPP** | — | ✓ | Android counterpart to iLEAPP. |
| **Andriller** | — | ✓ | Open source Android. |
| **APOLLO** | ✓ | — | Cross-platform "pattern of life" parser (KnowledgeC, biome). |

Working with iOS backups directly:
- Decrypt with the password (or brute-force with `iOSbackup`/Hashcat mode 14800)
- `Manifest.db` first — it tells you what every hashed filename in the backup actually is
- Parse with iLEAPP for a structured report

---

## Browser artifacts

- **Hindsight** — Chromium history (Chrome, Edge, Brave, Opera). One command, structured output.
- **DB Browser for SQLite** — open the History/Cookies/Login Data DBs directly.
- **NirSoft suite** — `BrowsingHistoryView`, `ChromeHistoryView`, `ChromeCacheView`, `IECacheView`, `MZHistoryView`. Tiny Windows tools, GUI.
- **ESEDatabaseView** — for legacy IE/Edge `WebCacheV01.dat`.

Cookies from Chrome are encrypted on Windows with DPAPI — to decrypt programmatically you need the user's master key + Windows login password (or live extraction with the user's session active).

---

## Registry analysis (Windows)

- **RegRipper** — Perl, plugin-based. The classic. `rip.pl -r SYSTEM -p usbstor`.
- **Eric Zimmerman's Registry Explorer / RECmd** — modern GUI + CLI. Bookmarks for high-value keys built in.
- **YARP** — Python registry parser library, useful for scripting.
- **AccessData Registry Viewer** — comes with FTK.

For ShellBags specifically: **SBECmd** (Eric Zimmerman) parses both `UsrClass.dat` and `NTUSER.DAT` ShellBags into a CSV.

For UserAssist (GUI program execution): RegRipper plugin or RegistryExplorer's bookmark.

---

## Event log analysis (Windows)

Modern `.evtx` (Vista+) parsers:
- **EvtxECmd** (Eric Zimmerman) — fast CLI parser to CSV/JSON.
- **Chainsaw** — runs Sigma rules against EVTX. Detects known attacker patterns out of the box.
- **Hayabusa** — similar to Chainsaw, Rust-based, very fast.
- **EventLogExplorer** — GUI viewer.
- Native `Get-WinEvent` PowerShell.

Old `.evt` (XP/2003): `evtxexport`, or convert via Windows itself.

For high-signal hunting: feed EVTX through **Sigma rules** via Chainsaw/Hayabusa rather than reading every event. There are thousands per hour on a normal box.

---

## Timeline construction

- **Plaso / log2timeline** — the standard. Parses ~200 artifact types into a unified "super timeline" (CSV or Elastic). `log2timeline.py case.plaso disk.E01` then `psort.py -o l2tcsv -w timeline.csv case.plaso`.
- **Timesketch** — web frontend for plaso output. Multiple analysts can collaborate, tag, filter.
- **MFTECmd** + **EvtxECmd** + **AmcacheParser** + ... → CSV → consolidate in Excel / KAPE's RECmd.
- **Mactime** (TSK) — narrower: just MAC times from a body file.

The student should learn Plaso early. It's painful at first but it's how real timelines get built.

---

## File carving

- **PhotoRec** — recovers files from raw blocks by signature. Targets photos, docs, archives, many file types.
- **Foremost** — older, signature-based.
- **Scalpel** — Foremost fork, faster.
- **bulk_extractor** — different philosophy: scans for *features* (emails, URLs, credit cards, BTC addresses) across raw bytes. Doesn't carve files but extracts evidence patterns. Excellent on pagefile/swap/unallocated.

Carving recovers *content* but loses *filesystem context* — no path, no timestamps. Use it after MFT-aware methods, not before.

---

## Triage / live response

- **KAPE** — targets define what to collect, modules define what to parse. Industry favorite.
- **Velociraptor** — agent-based, scales to thousands of hosts.
- **GRR Rapid Response** — Google's open-source alternative.
- **Sysinternals** (`Autoruns`, `Process Explorer`, `Procmon`, `Sigcheck`, `Strings`) — handheld live analysis. Run from a sterile USB on a powered-on suspect machine when full imaging isn't feasible.
- **CyLR** — minimal live collector.

---

## Specialty / niche

- **Strings**: `strings -e l file` (16-bit LE for Windows binaries), `strings -a` (full file). Then grep.
- **FLOSS** (Mandiant) — extracts strings *including* obfuscated/stack strings from malware.
- **Ghidra** / **IDA Free** / **radare2** / **Cutter** — reverse engineering, when malware analysis enters the picture.
- **YARA** — signature matching across files. Useful with curated rule sets (e.g., `awesome-yara`).
- **Capa** — capability detection in binaries (Mandiant).
- **OSFMount** — mount images on Windows for read-only browsing.
- **xmount** / **ewfmount** — mount E01/Ex01 on Linux.
- **VBoxManage** / **qemu-img** — convert image formats (VMDK ↔ raw ↔ E01 with detours).
