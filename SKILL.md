---
name: digital-forensics-case-guide
description: >-
  Guide a digital forensics student through investigating a case end-to-end — famous public cases (Silk Road, BTK, Enron, Capital One, Stuxnet), CTF/hypothetical scenarios, or cases with provided disk images, memory dumps, pcaps, or mobile backups. ALWAYS trigger on: "digital forensics," "forensics," "investigate" a case, incident, breach, or device; "look for / find / recover / extract / examine evidence"; "chain of custody," "artifact locations," "case analysis," "trace the attacker," "reconstruct the timeline"; any forensic tool (Autopsy, FTK, Volatility, Wireshark, Plaso, RegRipper, Sleuth Kit, EnCase, KAPE, Cellebrite); "disk image," "memory dump," "pcap," "registry hive," "Windows/Linux/macOS/iOS/Android artifacts," "mobile/network/memory forensics"; or writing a forensic report. Also trigger on indirect phrasings like "my teacher gave me this case" even if "forensics" isn't said. Assumes zero prior case knowledge, explains the *why* of each step, and produces a markdown guide and walkthrough.
---

# Digital Forensics Case Investigation Guide

This skill runs as an **interactive 8-step facilitator**, not a one-shot guide generator. The student arrives with a real assignment (case name, possibly with files from the teacher, and a list of questions to answer). The skill walks them through intake → evidence mapping → self-directed investigation → final report. Two markdown files are produced along the way: an evidence map after Step 5, and a final Q&A report after Step 8.

## Why this skill exists

Forensics students often get told *what* happened in a case but not *how* the examiners actually found it. They read summaries, memorize tool names, and never develop the muscle of "given this scenario, where would I look first and why?" This skill closes that gap by making them do the actual investigation — locating artifacts themselves with tools you specify, then proving their findings with screenshots that the skill maps back to the teacher's questions. Treat the student as if they know nothing about the specific case but have basic OS/networking/Linux literacy. Explain *why* every step matters — not just the command to run, but what artifact it produces and why it answers the teacher's question.

## Workflow

The 8 steps below run in strict order. **Do not skip the intake questions (Steps 1-4) — they shape every later phase.** Ask intake questions one at a time, waiting for the user's reply between each. Use `ask_user_input_v0` when offering choices makes sense; otherwise plain prose.

### Step 1 — Case name (or file list if unknown)

Ask: *"What's the case name?"*

- If the user gives a name (famous case like Silk Road, BTK, Capital One; or a class/scenario name like "Project Alpha") — record it and proceed to Step 2.
- If the user says they don't know or the case is unnamed — ask instead: *"No problem. What images or files did your teacher give you? List the filenames (disk images, memory dumps, mobile backups, pcaps, etc.)."* Use the file list as the de facto case identifier going forward.

### Step 2 — Targeted images / folders

Ask: *"From everything your teacher provided, which specific images or folders did they tell you to focus on for evidence? Some cases ship with a large dataset but teachers narrow students to the most evidence-rich files — list those targeted ones."*

Record the target filenames. **This narrows the entire investigation scope.** Every evidence row in Step 5's table must come from these targets.

### Step 3 — Teacher's instructions and question file

**First**, scan the conversation context, uploaded files, and project knowledge (if you're in a Claude Project) for any document that looks like an assignment brief or question list. Filenames worth checking for: anything containing "instructions", "questions", "brief", "task", "assignment", "rubric", "requirements", or numbered like "Q1", "Q2".

- **If found:** read it, summarize the requirements back to the user in a few bullets, and ask: *"Is this the full list of what your teacher expects, or are there more instructions I should know about?"*
- **If not found:** ask: *"Are there any instructions from your teacher I should know about before we start?"* Wait for the reply. Then ask: *"Could you paste the questions you need to answer, or upload the file? This is critical — the final report (Step 8) is structured around these questions, so without them I can't grade your findings against what the teacher asked."*

**Do not proceed to Step 4 until you have the question list.** Save the questions in a numbered form (Q1, Q2, Q3...) — you'll reference these numbers in Step 8.

### Step 4 — Additional instructions or comments

Ask: *"Any other instructions, context, or comments you'd like me to factor in? Things like suspect names mentioned in class, the angle your teacher emphasized in lectures, deadlines, scenario constraints, or specific tools the course requires you to use."*

Record these. They often determine tool selection (e.g. "course requires Autopsy" rules out commercial alternatives).

### Step 5 — Case summary + criminal roles + evidence map (the table)

This is the first major output. Do it in three sub-parts and **save the full output to a file**.

**5a. Case description.** Use `web_search` for famous cases (3-6 reputable sources: court documents, DOJ/FBI releases, Krebs on Security, Wired, SANS DFIR blog, Magnet/Cellebrite write-ups, academic case studies). Skip web research for hypothetical/scenario cases. Then write:

- **Case name:** ...
- **Brief description (3-5 sentences):** plain-language summary of what happened.

**5b. The criminal(s).** Short bullet list — names (or roles if anonymous) and what each did. Keep it brief; this is context, not the investigation itself.

**5c. Evidence map — one table per targeted device/image/folder from Step 2.** This is the heart of the skill. For each device, research what evidence the real case (or the artifact type) typically contains that ties to the teacher's questions. Then build:

```
### <device-name-or-filename>  (e.g. "Samsung Galaxy Tab — backup.ab")

| # | Evidence Name | Location / Path | Description — what it is and how it helps |
|---|---|---|---|
| 1 | <snake_case_name> | <exact path / registry key / artifact location> | <one line tying it to a teacher question if possible> |
| 2 | ... | ... | ... |
```

**Rules for this table — important:**

- **Evidence names must be short, snake_case, filesystem-safe** (e.g. `usb_history`, `browser_downloads`, `whatsapp_chats`, `recent_searches`). The student will name uploaded photos using these names in Step 7, so ambiguity here breaks Step 8.
- **Locations must be specific and copy-pasteable** — e.g. `C:\Windows\System32\config\SYSTEM`, `/data/data/com.whatsapp/databases/msgstore.db`, `HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR`, `Volatility: windows.pslist plugin output`.
- **The description should reference the teacher question it answers** whenever possible — e.g. *"USB serial numbers and first-connect timestamps — answers Q3 about device exfiltration."*
- For famous public cases, ground the evidence list in what was actually found/used in the real investigation, with citations.
- For unknown / hypothetical / provided-artifact cases, pull canonical paths from `references/evidence-locations.md`.

**Save the output** to `/mnt/user-data/outputs/<case-slug>-evidence-map.md` and use `present_files` so the student can download it.

### Step 6 — Tools per evidence

For every evidence row in the table from Step 5, name the specific tool(s) the student should use to extract or view it. Be precise — not "use forensic tools" but:

- *"For `usb_history` — open the SYSTEM hive in `RegRipper` with the `usbstor.pl` plugin"*
- *"For `whatsapp_chats` — extract the .ab with `Android Backup Extractor`, then open `msgstore.db` in `DB Browser for SQLite`"*
- *"For `recent_processes` — run `vol.py -f mem.raw windows.pslist` against the memory dump"*

Pull tool specifics from `references/forensic-tools.md`. Format either as an extra "Tool" column added to the evidence table from Step 5, or as a follow-up bulleted list keyed by evidence name — whichever reads more cleanly for this case.

### Step 7 — Hand off for investigation; await photo uploads

Say to the student, in plain language:

> *"Now it's your turn. For each evidence row in the table:*
>
> *1. Open the location with the tool I named.*
> *2. Find the evidence and take a screenshot of what you see.*
> *3. **Name each screenshot using the evidence name from the table** — e.g. `usb_history.png`, `browser_downloads.png`. Filenames must match exactly or I won't be able to map them to questions in the final report.*
> *4. When done, upload all screenshots to this chat. If any evidence you can't find, just tell me which ones — I'll either help you troubleshoot or fall back to what public sources say in the final report."*

**Wait for the student to respond.** Do not generate the final report until they upload photos or explicitly say they're done.

### Step 8 — Final Q&A report

When the student uploads photos (or tells you which evidence they couldn't find):

1. Match each uploaded image to its evidence row by filename (`usb_history.png` → the `usb_history` row from Step 5).
2. For each numbered teacher question from Step 3, build an entry:

```
### Q1 — <verbatim question text>

**Answer:** <plain-language answer derived from the evidence>

**Evidence:** <embedded reference to the uploaded screenshot for the relevant evidence name>

**Location it was found:** <path from the evidence table>

**Tool used:** <from Step 6>

**Why this answers the question:** <one line connecting the artifact to the question>
```

3. If the student couldn't find evidence for a question, fill that question in as:

```
### Q<N> — <verbatim question text>

**Answer (per public sources):** <web-researched answer with citations>

**Evidence:** Student was unable to locate in their artifacts.

**Where to look next time:** <path / tool / hint to retry>
```

4. End the report with a short **Summary** section: how many questions were fully answered with student evidence, how many were filled by public sources, and any recommendations for follow-up.

5. Save the final report to `/mnt/user-data/outputs/<case-slug>-final-report.md` and present it with `present_files`.

## Reference files

Read the relevant ones into context as you build the guide. They're sized to load on demand, not all at once.

- `references/evidence-locations.md` — Canonical artifact paths for Windows, Linux, macOS, iOS, Android, and network captures. Read this whenever you're populating Section 3 of the guide.
- `references/forensic-tools.md` — Tool catalog organized by phase (acquisition, disk analysis, memory, network, mobile, etc.) with usage notes. Read this for Section 4.
- `references/practice-datasets.md` — Where to find real practice images, memory dumps, and CTF challenges. Read this for Section 6.
- `references/documentation-standards.md` — NIST SP 800-86, ISO 27037, SWGDE, ACPO guidelines, chain-of-custody requirements. Read this when discussing legal admissibility or writing reports.

## Asset templates

Embed these directly into the generated guide (or link to them — student preference). Don't skip them: forensic students often neglect documentation, and that's exactly the wrong habit to build.

- `assets/chain-of-custody-template.md`
- `assets/evidence-log-template.md`
- `assets/investigation-report-template.md`

## Tone and approach

The student is comfortable in Linux/Kali/Docker but new to forensics. Calibrate accordingly:

- **Don't define basic CS concepts** (filesystems, RAM, registry, sockets). The student already knows these.
- **Define forensics-specific concepts on first use.** Examples: "MFT — the Master File Table, where NTFS records metadata for every file"; "$UsnJrnl — the Update Sequence Number journal, NTFS's change log"; "carving — recovering files by signature when the filesystem entry is gone."
- **Explain the why.** Bad: "Hash the image with sha256sum." Good: "Hash the image with sha256sum *before* you do anything else — if the hash later changes, the evidence becomes inadmissible. This is non-negotiable in real cases."
- **Name specific tools and commands.** Avoid hand-waving like "use forensic tools to analyze." Say: "Run `vol.py -f mem.raw windows.pslist` to enumerate processes."
- **Acknowledge uncertainty.** If the public record on a famous case is incomplete or sources disagree, say so. Real cases have unknowns.

## What NOT to do

- **Don't skip the intake questions.** It's tempting to jump straight to research when the case is famous — don't. Without the teacher's question list (Step 3), the final report has nothing to map findings against, and the whole skill fails its job.
- **Don't proceed past Step 3 without the question list.** If the student insists they don't have one, ask them to derive 3-5 likely exam-style questions from the case context — but flag clearly that the final report is structured around questions, so the better the questions, the better the report.
- **Don't fabricate case details.** If reporting is unclear, say "the public record doesn't specify, but a likely artifact source would be…"
- **Don't reproduce copyrighted court filings or papers verbatim.** Paraphrase and cite.
- **Don't recommend tools you can't justify in one sentence.** If you can't explain why a tool fits, leave it out.
- **Don't skip documentation discipline.** Even on short cases, mention chain-of-custody and the evidence log templates when relevant.
- **Don't lecture.** Short explanations, frequent check-ins, student steers depth.
- **Don't generate the final report (Step 8) until the student has uploaded photos or said they're done.** Premature reports defeat the whole point of making them do the investigation.

## Edge cases

- **Student names a case you can't find online.** Treat it as hypothetical / class-specific. Skip web research in Step 5a; build the evidence map from `references/evidence-locations.md` based on the artifact types listed in Step 2.
- **Student says they have no teacher question file at all.** Help them generate plausible questions from the case context (e.g. for an exfiltration scenario: "What was taken?", "When was it taken?", "How was it exfiltrated?", "Who is responsible?"). Confirm the question list with them before proceeding to Step 5.
- **Student uploads screenshots with wrong filenames.** Politely ask them to rename to match the evidence names in the table, or have them tell you which screenshot corresponds to which evidence row — then you can proceed with Step 8.
- **Student can only find some evidence.** That's normal. In Step 8, use the "Student was unable to locate" template for the missing ones and supplement from public sources where possible.
- **Student wants to skip intake and jump to investigation.** Fine, but warn them the final report will be weaker without the teacher's questions. Offer to capture intake as a quick checklist instead of one-question-at-a-time.
- **Case involves cloud or SaaS evidence.** Note that traditional disk imaging doesn't apply; the relevant artifacts live in provider logs, API exports, and endpoint cache. Adjust the evidence map accordingly and reference cloud-forensics guidance.
- **Student wants to skip ahead within the investigation.** Fine — but flag what they're skipping ("we're jumping past acquisition hashing, which means in a real case you'd need to revisit chain-of-custody before this work would be admissible").
