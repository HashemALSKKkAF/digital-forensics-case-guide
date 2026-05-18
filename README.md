# Digital Forensics Case Guide

A [Claude Skill](https://docs.claude.com/en/docs/agents-and-tools/agent-skills) that walks a digital forensics student through investigating a case end-to-end — from intake to a final Q&A report — the way a real examiner would, but with a teacher's voice.

Built for the case where a teacher hands you an assignment ("here are some images, here are the questions, find the evidence"), this skill turns it into an 8-step interactive workflow: intake, evidence mapping, self-directed investigation with named tools, and a final report that maps your screenshots back to each teacher question.

## What this skill does

When triggered, the skill runs an **interactive 8-step facilitator**:

1. **Asks the case name** (or the file list, if the case is unnamed).
2. **Asks which images/folders the teacher targeted** — narrows scope to evidence-rich files.
3. **Looks for / asks for the teacher's question file** — refuses to proceed without it, because the final report is structured around the questions.
4. **Asks for any additional instructions or context** the teacher gave.
5. **Produces the evidence map** — a per-device table of evidence locations, paths, and how each ties to a teacher question, saved as a markdown file.
6. **Names the specific tool** for extracting each evidence (Autopsy, FTK, Volatility, RegRipper, etc.).
7. **Hands the investigation back to the student** — open each location with the named tool, screenshot, name the files using the evidence names from the table, upload back.
8. **Produces the final Q&A report** — maps each teacher question to the student's evidence (or, if missing, to public sources with citations).

Two markdown files are produced along the way:

- `<case-slug>-evidence-map.md` (after Step 5)
- `<case-slug>-final-report.md` (after Step 8)

## Who this is for

Digital forensics students working through case-based coursework, especially when the teacher provides files (disk images, memory dumps, mobile backups, pcaps) and a question rubric. Assumes basic Linux / OS / networking literacy but **zero prior knowledge of the specific case**.

The skill also works for:

- Famous public cases (Silk Road, BTK, Enron, Capital One, Stuxnet, etc.) — uses web research to ground the evidence map in what was actually found.
- Hypothetical / CTF-style scenarios — applies forensic principles without web research.

## Triggers

The skill activates automatically on phrasings like:

- "investigate this case"
- "look for / find / recover / extract / examine evidence"
- "digital forensics," "forensics," "forensic investigation"
- "my teacher gave me this case"
- "walk me through this dump"
- Any forensic tool name (Autopsy, FTK, Volatility, Wireshark, etc.)
- Artifact types (disk image, memory dump, pcap, registry hive, etc.)

It does **not** need you to say the word "forensics" explicitly — phrasings like "help me investigate" or "where would I look for evidence of X" are enough.

## Install

### Claude.ai (web/desktop)

1. Download the latest release (`.skill` file) from the [Releases](../../releases) tab, or zip this repo's `digital-forensics-case-guide/` folder yourself.
2. In Claude.ai → **Settings → Capabilities → Skills**, click **Upload**.
3. Select the `.skill` file.
4. Toggle the skill on.
5. Start a new chat and trigger it (see Triggers above).

### Claude Code (CLI)

Either install via the skills CLI:

```bash
npx skills add https://github.com/HashemALSKKkAF/digital-forensics-case-guide
```

Or copy manually:

```bash
git clone https://github.com/HashemALSKKkAF/digital-forensics-case-guide.git
mkdir -p ~/.claude/skills/
cp -r digital-forensics-case-guide ~/.claude/skills/
```

Then in Claude Code:

```
/skills reload
```

Verify with `/skills` — `digital-forensics-case-guide` should be listed.

## Structure

```
digital-forensics-case-guide/
├── SKILL.md                              # Workflow, triggers, tone
├── references/
│   ├── evidence-locations.md             # Canonical artifact paths (Win/Linux/macOS/iOS/Android)
│   ├── forensic-tools.md                 # Tool catalog by phase
│   ├── practice-datasets.md              # Where to find real practice images
│   └── documentation-standards.md        # NIST SP 800-86, ISO 27037, ACPO, SWGDE
├── assets/
│   ├── chain-of-custody-template.md
│   ├── evidence-log-template.md
│   └── investigation-report-template.md
├── LICENSE
└── README.md
```

## Customization

The skill is plain markdown — open and edit any file. Common tuning points:

- **`SKILL.md`** — the 8-step workflow, trigger keywords, tone guidance. Edit if your course emphasizes a different methodology or you want to tighten/loosen the intake.
- **`references/evidence-locations.md`** — add OS-specific artifact paths your course emphasizes.
- **`references/forensic-tools.md`** — add or replace tools if your program standardizes on specific ones (some schools are EnCase-heavy, others Autopsy-only).
- **`references/practice-datasets.md`** — add the dataset repository your class uses.

After editing, run `/skills reload` in Claude Code to pick up changes mid-session.

## Limitations

- **Famous-case branch is research-dependent.** Quality of the evidence map for famous cases depends on what public reporting exists; for cases with thin or contested coverage, the skill will say so rather than fabricate details.
- **Teacher questions are mandatory.** The final report (Step 8) is structured around them. If you don't have a question file, the skill will help you derive plausible questions from the case context, but the report quality depends on question quality.
- **Photo naming must be exact.** Screenshots uploaded in Step 7 must match the snake_case evidence names from the Step 5 table, or the skill will ask you to clarify the mapping.

## License

MIT — see [LICENSE](LICENSE).

## Contributing

PRs welcome. If you find the skill misbehaving on a real assignment — wrong tone, missing artifact path, hallucinated detail about a famous case — open an issue with the case context and the conversation log.
