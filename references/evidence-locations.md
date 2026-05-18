# Evidence Locations Reference

Canonical artifact paths and registry locations across operating systems. Use this when populating Section 3 ("Where the Evidence Lives") of the case guide.

## Table of contents
- [Windows](#windows)
- [Linux](#linux)
- [macOS](#macos)
- [iOS](#ios)
- [Android](#android)
- [Network captures (pcap)](#network-captures-pcap)
- [Email artifacts](#email-artifacts)
- [Cloud / SaaS notes](#cloud--saas-notes)

---

## Windows

### Filesystem metadata (NTFS)
- `$MFT` — Master File Table. Metadata for every file. Located at the root of the volume. Tools: MFTECmd, analyzeMFT.py, FTK.
- `$LogFile` — NTFS transaction log. Recent file operations.
- `$UsnJrnl:$J` — Update Sequence Number journal. Change records for files (created/modified/deleted). Often catches things the user "deleted" the same session.
- `$Recycle.Bin\<SID>\` — Deleted files. `$I*` files hold metadata (original path, deletion time); `$R*` files hold the actual content.

### Registry hives
On disk:
- `C:\Windows\System32\config\SYSTEM` — system config, services, USB history, network info
- `C:\Windows\System32\config\SOFTWARE` — installed apps, Run keys, Windows version
- `C:\Windows\System32\config\SECURITY` — security policy
- `C:\Windows\System32\config\SAM` — local accounts, hashed passwords (with SYSTEM key for decryption)
- `C:\Users\<user>\NTUSER.DAT` — per-user settings, RecentDocs, TypedURLs, RunMRU
- `C:\Users\<user>\AppData\Local\Microsoft\Windows\UsrClass.dat` — ShellBags (folder-view metadata, evidence of folder access even if folder deleted)

High-value registry keys:
- `SYSTEM\CurrentControlSet\Enum\USBSTOR` — USB device connection history (vendor, product, serial)
- `SYSTEM\CurrentControlSet\Control\DeviceClasses\{53f56307-...}` — disk class device GUIDs, ties USB devices to drive letters
- `SOFTWARE\Microsoft\Windows\CurrentVersion\Run` and `RunOnce` — autostart programs (persistence)
- `SOFTWARE\Microsoft\Windows NT\CurrentVersion\ProfileList` — user SIDs to profile path
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs` — recently opened files by extension
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist` — GUI program execution history (ROT13 encoded)
- `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths` — paths typed in Explorer address bar

Tools: RegRipper, Eric Zimmerman's Registry Explorer / RECmd.

### Execution evidence
- `C:\Windows\Prefetch\*.pf` — Prefetch. Up to 1024 entries (Win 8+). Last run time, run count, files loaded. **Disabled by default on SSDs in some configs**, so absence isn't proof of nothing.
- `C:\Windows\AppCompat\Programs\Amcache.hve` — Amcache. Executed program metadata + SHA-1 hashes.
- `SYSTEM\CurrentControlSet\Control\Session Manager\AppCompatCache` — ShimCache (also called AppCompatCache). Executed binaries with timestamps. Note: timestamps are file last-modified, not execution time — students get this wrong constantly.
- `SYSTEM\CurrentControlSet\Services\bam\State\UserSettings\<SID>` — BAM (Background Activity Moderator). Executable run times.
- `C:\Windows\System32\sru\SRUDB.dat` — System Resource Usage Monitor. Network usage and energy consumption per process. Tool: srum-dump.

### User activity
- `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\` — LNK files for recently opened files. Tool: LECmd.
- `C:\Users\<user>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations\` and `CustomDestinations\` — Jump Lists. Tool: JLECmd.
- ShellBags (in `UsrClass.dat`) — folder access history. Tool: SBECmd.

### Event logs (Windows Vista+)
Path: `C:\Windows\System32\winevt\Logs\*.evtx`

High-value channels:
- `Security.evtx` — logons (4624 success, 4625 fail, 4634 logoff), privilege use, object access
- `System.evtx` — service install/start, time changes
- `Application.evtx` — app crashes, MSI installs
- `Microsoft-Windows-PowerShell/Operational.evtx` — PS script blocks (event 4104)
- `Microsoft-Windows-Sysmon/Operational.evtx` — Sysmon, if installed (process create, network connections, file create)
- `Microsoft-Windows-TerminalServices-LocalSessionManager/Operational.evtx` — RDP sessions

Tools: EvtxECmd, Chainsaw, Hayabusa.

### Memory artifacts (on disk)
- `C:\hiberfil.sys` — hibernation file. Compressed RAM snapshot. Convert with hibr2bin/Volatility.
- `C:\pagefile.sys` — page file. Fragments of memory paged to disk. Strings + bulk_extractor are useful here.
- `C:\swapfile.sys` — Modern app swap file (Win 8+).

### Browsers
| Browser | History/cache path |
|---|---|
| Chrome | `%LOCALAPPDATA%\Google\Chrome\User Data\Default\` (History, Cookies, Login Data, Bookmarks — all SQLite) |
| Edge (Chromium) | `%LOCALAPPDATA%\Microsoft\Edge\User Data\Default\` |
| Firefox | `%APPDATA%\Mozilla\Firefox\Profiles\<random>.default-release\` (places.sqlite, formhistory.sqlite) |
| IE / legacy Edge | `%LOCALAPPDATA%\Microsoft\Windows\WebCache\WebCacheV*.dat` (ESE database) |

Tools: Hindsight (Chrome/Edge), DB Browser for SQLite, NirSoft browser tools, ESEDatabaseView for IE.

### Volume Shadow Copies
- Snapshots managed by VSS. Often contain older versions of files an attacker tried to delete.
- Mount with `vshadowmount` (libvshadow) or PowerShell. Each snapshot is a point-in-time view of the volume.

### Other useful spots
- `C:\Windows\System32\Tasks\*` — scheduled tasks (XML).
- `C:\Windows\Inf\setupapi.dev.log` — driver install log, useful for USB-first-connected timestamps.
- `C:\ProgramData\Microsoft\Windows Defender\Support\` — Defender detection logs.

---

## Linux

### Logs (`/var/log/`)
- `auth.log` (Debian) / `secure` (RHEL) — authentication events (sudo, ssh, logins)
- `syslog` / `messages` — general system messages
- `kern.log` — kernel
- `dpkg.log` / `apt/history.log` (Debian) or `dnf.log` / `yum.log` (RHEL) — package installs
- `wtmp`, `btmp`, `lastlog` — login records (binary, parse with `last`, `lastb`, `lastlog`)
- `audit/audit.log` — auditd events (if enabled)
- `journalctl` (systemd) — `/var/log/journal/<machine-id>/*.journal`. Parse with `journalctl --file` or `journalctl -D /path`.

### User artifacts
- `~/.bash_history`, `~/.zsh_history` — shell history. Note: not always written if user used `kill -9` on the shell or set `HISTFILE=/dev/null`.
- `~/.lesshst`, `~/.viminfo` — less and vim history
- `~/.ssh/known_hosts`, `authorized_keys`, `id_*` — SSH keys and known servers
- `~/.local/share/recently-used.xbel` — recent files (XDG, GNOME)
- `~/.config/`, `~/.cache/` — app configs and caches

### Persistence locations (where attackers hide)
- `/etc/cron*`, `/var/spool/cron/crontabs/*` — cron jobs
- `/etc/systemd/system/*.service`, `~/.config/systemd/user/*` — systemd unit files
- `/etc/init.d/`, `/etc/rc*.d/` — SysV init (older systems)
- `/etc/profile`, `/etc/bash.bashrc`, `~/.bashrc`, `~/.profile` — shell init
- `/etc/ld.so.preload` — LD_PRELOAD for global library injection
- `/etc/passwd`, `/etc/shadow`, `/etc/sudoers`, `/etc/group` — users, hashes, sudo grants

### Filesystem
- `/tmp`, `/var/tmp`, `/dev/shm` — common attacker drop zones
- `/proc/<pid>/exe` — symlink to running process binary (useful for live forensics)
- `/proc/<pid>/maps`, `/proc/<pid>/cmdline` — memory map and command line
- File timestamps: `stat` shows atime/mtime/ctime; `debugfs` for ext4 inode forensics

### Tools
TSK (`fls`, `icat`, `mmls`), Plaso/log2timeline, LiME (live memory acquisition), AVML, Volatility (Linux profiles).

---

## macOS

### Logs
- `/var/log/` — system logs (asl, install, system)
- `~/Library/Logs/`, `/Library/Logs/` — app and per-user logs
- Unified Log: `log show --predicate '...' --last 24h` (or extract `.logarchive` for offline analysis with `log show --archive`)

### User artifacts
- `~/Library/Preferences/*.plist` — app preferences (binary plist; convert with `plutil -convert xml1`)
- `~/Library/Application Support/<app>/` — app data
- FSEvents: `/.fseventsd/` (root of volume) — file system change log. Tool: FSEventsParser.
- `~/Library/Preferences/com.apple.recentitems.plist` — recently opened files/apps/servers
- `~/Library/Application Support/Knowledge/knowledgeC.db` — Knowledge Base (app usage, focus, locations)
- `~/Library/Containers/com.apple.Safari/Data/Library/Safari/History.db` — Safari history

### macOS specifics
- `com.apple.quarantine` extended attribute on downloaded files (Gatekeeper)
- Spotlight metadata: `.Spotlight-V100/`
- Time Machine backups: `/Volumes/<TM-disk>/Backups.backupdb/`
- `~/.bash_sessions/` — terminal session logs (if shell is bash)

### Tools
mac_apt, APOLLO (cross-platform Apple pattern of life), Mac Forensics by SUMURI, Plaso macOS parsers.

---

## iOS

iOS forensics typically works from:
1. iTunes/Finder backup (`~/Library/Application Support/MobileSync/Backup/<UDID>/` on macOS)
2. Full file system extraction (checkm8 / GrayKey / Cellebrite advanced) — only on specific hardware/iOS combos
3. Logical extraction via AFC (Apple File Conduit)

### Backup structure
- `Manifest.db` — SQLite mapping fileID hash → original domain/path. Without this you can't make sense of the backup.
- `Manifest.plist` — backup metadata (encrypted? device info)
- Files stored as 40-char SHA-1 hashes in two-char prefix subdirectories

### Key SQLite databases (post-decryption)
- `sms.db` — iMessage and SMS
- `CallHistory.storedata` (Core Data, modern) or `call_history.db` (older) — calls
- `AddressBook.sqlitedb` — contacts
- `Photos.sqlite` — photo library metadata
- `History.db` (Safari)
- `knowledgeC.db` — pattern of life (app usage, device usage, focus)
- `interactionC.db` — Siri suggestions / contact interactions
- `biome/` — newer behavioral data store

### Tools
ALEAPP (iOS) — wait, ALEAPP is Android. **iLEAPP** for iOS. Other tools: Cellebrite, Magnet AXIOM, MOBILedit, iBackup Viewer (consumer-grade).

---

## Android

### Acquisition methods (from least to most invasive)
1. ADB backup (`adb backup`) — limited, depends on apps allowing backup
2. Logical extraction (Cellebrite logical, Magnet AXIOM)
3. File system extraction via custom recovery (TWRP) — requires unlocked bootloader
4. Physical extraction / chip-off — destructive or near-destructive

### Key paths on the device (require root or extraction)
- `/data/data/<app.package.name>/` — per-app private data (DBs, prefs, cache)
- `/data/system/` — accounts.db, packages.xml, locksettings.db
- `/sdcard/` (or `/storage/emulated/0/`) — user-accessible storage (downloads, DCIM, WhatsApp media)
- `/data/log/`, `/data/anr/` — system logs, ANR (app-not-responding) traces

### Common app artifacts
- WhatsApp: `msgstore.db` (encrypted with key), media in `WhatsApp/Media/`
- Telegram: `cache4.db` and media
- Signal: encrypted SQLCipher DB; key in keystore
- Browser apps: similar SQLite layout to Chrome desktop

### Tools
ALEAPP (Android Logs Events And Properties Parser), Andriller, Magnet AXIOM, Cellebrite UFED, ADEL.

---

## Network captures (pcap)

When the evidence is a packet capture, focus on:

| Layer / artifact | What to look for | Tool |
|---|---|---|
| DNS queries | Beaconing patterns, DGA domains, exfil via DNS TXT | Wireshark, Zeek `dns.log` |
| HTTP | User-Agents, URIs, posted data, downloaded files | Wireshark "Export Objects > HTTP", NetworkMiner |
| TLS/SSL | SNI, JA3/JA3S fingerprints (without keys, you can still profile) | Zeek `ssl.log`, Wireshark TLS dissector |
| SMB | File transfers, named pipes (`\PSEXESVC` = lateral movement signal) | Wireshark, Zeek |
| Email | SMTP/POP3/IMAP plaintext (rare now), credentials | Wireshark "Follow Stream" |
| Files | Reassemble carved files | NetworkMiner, foremost on pcap |

Useful Wireshark display filters:
- `tcp.flags.syn==1 and tcp.flags.ack==0` — connection attempts
- `dns.qry.name contains "domain.com"`
- `http.request.method == "POST"`
- `frame contains "password"` (sloppy but sometimes works)

---

## Email artifacts

- **Outlook PST/OST**: Personal Storage Table / Offline Storage Table. Path: `%LOCALAPPDATA%\Microsoft\Outlook\` or `Documents\Outlook Files\`. Tools: libpff, Aid4Mail, MailXaminer.
- **MBOX** (Thunderbird, Apple Mail, many Linux clients): plain text, easy to parse.
- **Headers**: always pull full headers — `Received:` chain shows the actual hop path, useful for spoof detection.
- **Webmail**: artifacts live in browser cache, not on disk as messages. Look at IndexedDB and SQLite for Gmail/Outlook web.

---

## Cloud / SaaS notes

Traditional disk imaging doesn't apply. Relevant artifacts:

- Provider-side logs (Microsoft 365 Unified Audit Log, Google Workspace Admin Logs, AWS CloudTrail, Azure Activity Log)
- Endpoint sync caches (OneDrive client cache, Dropbox metadata DB, Google Drive File Stream cache)
- API exports / takeout exports
- Browser-side artifacts (the user accessed the cloud through a browser → see browsers section)

If a case is cloud-heavy, the methodology shifts to **log analysis and timeline correlation** rather than artifact carving. Timeline tools (Plaso, Timesketch) are even more central.
