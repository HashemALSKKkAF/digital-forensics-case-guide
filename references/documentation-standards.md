# Documentation Standards Reference

What "proper documentation" actually means in digital forensics. Use this when explaining Section 7 of the case guide, when the student asks about admissibility, or when they wonder why the chain-of-custody form has so many fields.

## Why this matters

A finding is only useful if it survives challenge. In court, in a tribunal, in an internal HR proceeding — anywhere with consequences — opposing counsel will probe two things first:

1. **Authenticity** — is this evidence what you say it is?
2. **Integrity** — has it been altered since collection?

Sloppy documentation defeats both. A perfectly correct technical finding can be excluded because the examiner can't show, on paper, that the evidence was handled rigorously. This is why students who focus only on the "fun" parts of forensics (carving, decoding, RE) often hit a wall in their first real engagement.

## Key standards to know by name

### NIST SP 800-86 — Guide to Integrating Forensic Techniques into Incident Response
- **URL**: https://csrc.nist.gov/publications/detail/sp/800-86/final
- The widely-cited US baseline. Defines a four-phase model: Collection → Examination → Analysis → Reporting. Mention this when explaining methodology — it's where the phases come from.

### NIST SP 800-101 (Rev. 1) — Guidelines on Mobile Device Forensics
- Mobile-specific extension of 800-86.

### ISO/IEC 27037 — Identification, collection, acquisition and preservation of digital evidence
- The international counterpart. Defines roles ("DEFR" — Digital Evidence First Responder; "DES" — Digital Evidence Specialist) and procedural requirements.

### ISO/IEC 27041, 27042, 27043
- 27041: assurance for investigative methods
- 27042: analysis and interpretation
- 27043: incident investigation principles

### ACPO Good Practice Guide for Digital Evidence (UK)
- Old-school but foundational. Four principles, repeatedly cited:
  1. No action should change data on a device that may be used in court.
  2. If you must access original data, you must be competent and able to explain why.
  3. An audit trail must allow another examiner to reproduce your work.
  4. The case officer is responsible for ensuring law and these principles are followed.

### SWGDE (Scientific Working Group on Digital Evidence)
- **URL**: https://www.swgde.org/
- US-led group publishing best practices for specific domains (mobile, video, audio, image authentication, etc.). Free PDFs.

Students should be able to name at least NIST 800-86, ISO 27037, and the ACPO principles when asked about standards.

---

## The four phases (NIST 800-86)

Use these as the methodology backbone in every guide.

### 1. Collection
- Identify potential sources of evidence
- Apply write blockers (hardware preferred, software acceptable when documented)
- Acquire (image disks, capture memory, preserve logs)
- Hash everything **immediately** — before any analysis
- Document acquisition: time, examiner, tool/version, source device, hash values, location of resulting image file

### 2. Examination
- Reduce: filter known-good (NSRL), zero in on relevant date ranges, scope to specific users
- Extract: parse artifacts (registry, EVTX, browser DBs, etc.) into reviewable form
- Maintain a working hypothesis log — what you're looking for and why

### 3. Analysis
- Correlate findings across artifacts (timeline construction is core here)
- Test alternative explanations — could this be a sync conflict instead of malicious deletion?
- Distinguish *artifact* (what the data shows) from *interpretation* (what it means)

### 4. Reporting
- Audience-appropriate (executive vs. technical reader vs. court)
- Reproducible (another examiner could follow your steps)
- Cite tools, versions, and timestamps for every finding

---

## Chain of custody — what it actually proves

Chain of custody is a **continuous record** of who had the evidence, when, and what they did with it. It must:

- Begin at the moment of collection (or earlier, if seizure preceded collection)
- Have no unexplained gaps
- Identify every person who handled the item
- Record every transfer (from whom, to whom, when, why)
- Record every action taken on the evidence (imaged, analyzed, returned)
- Be signed at each handoff

If there's a gap — even a few hours where nobody can say where the device was — opposing counsel will argue tampering was possible. You don't have to *prove* tampering; the doubt alone can be enough to exclude.

For digital evidence specifically, hashes are the technical complement to the paper trail. The chain says "I had it"; the hash says "and it's the same now as it was then."

---

## What goes into a forensic report

Audience: a technically-literate but non-specialist reader (a manager, a judge, an attorney). Sections, in order:

1. **Cover page** — case number, examiner, dates, classification
2. **Executive summary** — 1 paragraph, plain language, the conclusion
3. **Authority and scope** — what authorized you to examine, what you were asked to determine, what you were *not* asked
4. **Evidence received** — every item with serial numbers, hash values, and chain-of-custody references
5. **Methodology** — tools (with versions), techniques, what you did and in what order
6. **Findings** — the technical results, organized by question or by artifact
7. **Conclusion** — what the findings indicate, with appropriate hedging ("consistent with…" not "proves…" unless it actually does)
8. **Glossary** — define every technical term used
9. **Appendices** — full tool output, screenshots, hash lists, raw timelines

What you should **not** do in a forensic report:
- State conclusions beyond what the data supports
- Use "obviously", "clearly", "always" — these are red flags
- Speculate about motive or intent (that's for the jury/decision-maker)
- Hide or omit findings that complicate your conclusion — opposing experts will find them and your credibility dies

---

## Hedging language — the technical writer's vocabulary

Real reports use careful language because forensic findings rarely prove things definitively. Useful phrases:

- *"Consistent with [activity X]"* — the artifact pattern matches X but isn't unique to it
- *"Indicates"* — strong inference from clear evidence
- *"Suggests"* — supportable but with alternatives possible
- *"The artifact shows [observation]; this is typically produced by [cause], though [alternative] is also possible"*
- *"No evidence was identified of [Y]"* — useful, but does NOT mean Y didn't happen — only that you didn't find evidence of it

Students should learn to write findings as *observations* first and *interpretations* second. "The Prefetch entry for `payload.exe` shows a last execution time of 2024-03-15 14:22:01 UTC" is an observation. "The malware was executed at 14:22 on March 15" is an interpretation that depends on the observation plus context (Prefetch reliability, clock synchronization, attribution of the binary).

---

## Common documentation mistakes (what to flag if the student does these)

- Hashing the image *after* analysis instead of before
- Writing "I copied the files" without naming the tool, version, and source/destination paths
- A chain-of-custody form with one entry ("I have it") and a six-week gap before the next entry
- Mixing analyst opinions into the findings section
- No tool versions cited — every tool's behavior changes between versions, so reproducibility requires this
- Screenshots without hashes or context (which case? Which evidence item? When taken?)
- Reports that read like a transcript ("then I did X, then I did Y") instead of being organized by question or finding

---

## Where the templates live

Three asset files in this skill:
- `assets/chain-of-custody-template.md` — single-item chain of custody form
- `assets/evidence-log-template.md` — examiner's evidence log (multiple items in a case)
- `assets/investigation-report-template.md` — full report skeleton

Each is generic enough to adapt to any case. Students should fill them in for *every* case they investigate, even practice ones — building the muscle now is the whole point.
