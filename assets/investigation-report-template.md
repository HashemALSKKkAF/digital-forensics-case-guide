# Forensic Investigation Report

**Case Number**: _____________________________
**Report Date**: _____________________________
**Examiner**: _____________________________
**Report Version**: _____________________________
**Classification**: (e.g., *Confidential — Privileged & Confidential / Internal / Public*)

---

## 1. Executive Summary

*One paragraph. Plain language. State the scope of the examination and the principal conclusion. A non-technical reader should be able to read only this section and understand what you found.*

```


```

---

## 2. Authority and Scope

### 2.1 Authority
*What authorized this examination? E.g., "Internal investigation request from Legal dated YYYY-MM-DD"; "Search warrant 24-CR-XXXX issued by Court of XXX on YYYY-MM-DD"; "Signed consent of system owner."*

### 2.2 Questions in scope
*Bullet list of the specific questions the examiner was asked to answer.*

- 
- 
- 

### 2.3 Out of scope
*Be explicit about what was NOT examined or analyzed. Limits prevent over-reading the report later.*

- 
- 

---

## 3. Evidence Examined

For each item, link to the corresponding Chain of Custody form and Evidence Log entry.

### Item 001
| Field | Value |
|---|---|
| Description | |
| Source | |
| Acquisition date | |
| Acquisition method | |
| Image file | (path / filename) |
| Hash (SHA-256) at acquisition | |
| Hash verified before analysis | (yes/no, date) |

*Repeat for each item.*

---

## 4. Examination Environment

| Field | Value |
|---|---|
| Workstation | (model / OS / hostname) |
| Workstation hash baseline | (if applicable) |
| Forensic suite(s) | (Autopsy 4.x / FTK 7.x / etc.) |
| Other tools | (with versions — Volatility 3.x.x, Plaso 2024.xx, RegRipper 3.0, etc.) |
| Working storage location | |
| Network isolation | (yes/no — describe) |

---

## 5. Methodology

*Describe the procedural steps taken. A peer examiner should be able to follow this and reproduce the work. Reference specific tool commands where appropriate.*

### 5.1 Acquisition
*How each item was acquired and verified.*

### 5.2 Examination
*How artifacts were extracted from the acquired images.*

### 5.3 Analysis
*How extracted artifacts were correlated and interpreted.*

### 5.4 Tool versions and configuration
*Already in section 4, but reiterate any tool-specific configurations (custom plugins, ruleset versions, etc.).*

---

## 6. Findings

*Organize findings by question (preferred) or by artifact category. For each finding, separate the OBSERVATION (what the data shows) from the INTERPRETATION (what it means).*

### Finding 6.1 — *(short title)*

**Observation:**
*What was observed. Cite the specific artifact, location, and the data values. Include hash of the source artifact if relevant.*

**Source(s):**
- Item ___, location: `path/to/artifact`
- (additional sources)

**Tool(s) used:**
- 

**Interpretation:**
*What the observation indicates. Use appropriately hedged language ("consistent with", "indicates", "suggests"). State alternative explanations if any are plausible.*

**Limitations:**
*What this finding does NOT establish.*

---

### Finding 6.2 — *(short title)*

**Observation:**

**Source(s):**

**Tool(s) used:**

**Interpretation:**

**Limitations:**

---

*(Repeat for each finding.)*

---

## 7. Timeline

*A consolidated chronology of relevant events derived from the findings. This is often the most useful section for a non-technical reader.*

| Date / Time (UTC) | Event | Source artifact | Item # | Confidence |
|---|---|---|---|---|
| | | | | |
| | | | | |
| | | | | |

Confidence levels: *High / Medium / Low* — or use the verbal hedging from the findings.

---

## 8. Conclusions

*Tie the findings back to the questions in section 2. Answer each question or state that the available evidence does not allow an answer. Do not reach beyond what the data supports.*

```


```

---

## 9. Recommendations

*Optional. If the requesting party has asked for next steps, list them here — additional collections, monitoring, follow-up interviews, etc.*

- 
- 

---

## 10. Glossary

*Define every technical term used in the report. The non-technical reader will return to this section repeatedly.*

| Term | Definition |
|---|---|
| MFT | Master File Table — the NTFS data structure that records metadata for every file on the volume. |
| Prefetch | A Windows feature that records executable run history to optimize startup; produces forensic value as evidence of program execution. |
| | |
| | |

---

## 11. Appendices

- **Appendix A**: Hash listings (full, with paths)
- **Appendix B**: Tool output (raw timelines, registry exports, etc.)
- **Appendix C**: Screenshots (each captioned with item, source, date)
- **Appendix D**: Chain of Custody forms (one per item)
- **Appendix E**: Evidence Log

---

## Examiner Certification

*I certify that the contents of this report are an accurate representation of the examination conducted, that the procedures described were followed, and that I am qualified to perform digital forensic examinations.*

Examiner: _______________________________
Signature: _______________________________
Date: _______________________________

---

*This report is the work product of the examiner identified above. Distribution should be limited to those with a legitimate need to know consistent with the classification level above.*
