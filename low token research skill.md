---
name: low-token-research
description: "Produce a rigorous, stress-tested research essay delivered as a downloadable, watermarked PDF. Use when the user asks for a report/essay on a subject (e.g. what humans cannot see/hear vs. what devices detect — multispectral, hyperspectral, infrared, UV, radar, sonar, ultrasound), a business/profit angle on such a topic, or a stress test of a previous answer of this type. Clean academic layout (Calibri/Carlito 12pt, double-spaced, numbered section titles, author + date only — no course/instructor header), low-opacity tiled watermark, every claim anchored to a measurable quantity or named source, at least three verbatim attributed quotes, a risk/falsification block, and exactly two Questions for Further Research. Token-light baseline (~3,000–5,000 tokens) that scales depth up when more source data is supplied."
license: CC-BY-NC-4.0
metadata:
  author: Matei-Ionuț Șuță
  version: '1.0'
  skill_id: Low-Token-research
---

# Low Token research

## Purpose

Produce a rigorous, stress-tested research essay, delivered as a downloadable, watermarked PDF.

## When to Use This Skill

Trigger when the user asks for any of:

- A report/essay on a subject. Examples of past usage: what humans cannot see/hear vs. what devices detect; multispectral, hyperspectral, infrared, UV, radar, sonar, ultrasound.
- A business/profit angle on any of the above.
- A "stress test" of a previous answer of this type.

Treat this as the structural template for all future topics: the goal is to take an initial idea and build on it slowly, deepening evidence with each pass.

## Runtime sizing

This skill is written to consume roughly 3,000–5,000 tokens when it runs on its own, so it can be composed alongside other skills within a normal context budget. Treat that range as the baseline cost of invoking the skill, not as a cap on the work: when the user supplies more source data, attachments, or sub-questions, scale the depth, number of sections, and length of analysis upward to use the additional input fully. More user data ⇒ more tokens spent and a longer essay; no extra data ⇒ stay near the baseline and do not pad.

## Core operating rules (consolidated)

### 1. Output format

- Deliver the deliverable as a downloadable PDF, attached in-conversation.
- Use a clean academic look for any essay/report-length output:
  - Calibri/Carlito font, 12 pt, double-spaced (leading ~24 pt).
  - 1-inch margins, letter size.
  - A simple author line and date only. This is INDEPENDENT work: do NOT add a course name, instructor/professor line, or class header of any kind.
  - Centered title (no bold, no underline).
  - Running header "LastName PageNumber" top-right of every page.
  - Works Cited on a new page, hanging indent, alphabetized.
- Numbered section titles are retained inside the body (e.g. "1. Stress-Tested Thesis") for navigability.

### 2. Watermark

- Apply the user's chosen watermark (default placeholder: "Matei-Ionuț Șuță") as a low-opacity (~0.07) tiled diagonal stamp across every page, plus an optional small visible footer line. Make the watermark string a parameter.

### 3. Stress-test procedure (apply to every draft AND to any prior answer)

1. Remove vague information; if a claim is too vague to defend, delete it OR replace it with an EXACT quoted passage cut from a scientific article, a physicist, or an authoritative source (verbatim, in quotation marks, attributed).
2. Remove "AI slop": looping restatements, ornamental complexity, sentences that re-describe the same idea only to sound deeper. One idea per paragraph.
3. Anchor every claim to a measurable quantity (nm, Hz/kHz, dollars, meters, degrees of visual angle, market size, CAGR) or to a named source.
4. Add a risk/falsification block: state what would make the claim false, and include a counterargument-and-rebuttal where an argument is made.
5. Add at least one NEW, web-inspired stress-test idea per major revision (e.g. NASA's "spectral fingerprint" check: is invisible data informative or merely decorative?).

### 4. Source-quote anchoring (canonical anchors to reuse)

- Visible-light limits: NASA "Visible Light" → 380 nm (violet) to 700 nm (red); "we can only see a small portion of this radiation."
- Hearing limits: NIH Neuroscience "The Audible Spectrum" → 20 Hz–20 kHz, adult upper limit ~15–17 kHz; bats to 200 kHz.
- Perceptual filling-in: Spillmann et al., Vision Research 46(25), 2006 → blind-spot completion of brightness, color, texture.
- Industrial ultrasound: FLIR → detects "2–130 kHz", leak as small as 0.007 L/min from 2.5 m; iCare → ultrasound analysis "typically 20 kHz"+.

Always quote verbatim, attribute inline, and list in Works Cited.

### 5. Standard question set (every report must answer the relevant ones for its topic)

- What frequencies/bands do humans detect vs. devices?
- What do cameras (light) vs. microphones (sound) actually detect? (Correct the common premise error that cameras detect sound.)
- How cheap is it to explore? Give a tiered cost ladder (tens of $ DIY → thousands → research-lab).
- Do we already do it, and is the data public? (astronomy + Earth: buildings, forests, caves, underwater.)
- Why do we NOT measure everything everywhere? (physics, cost, interpretation as an inverse problem, ethics/privacy.)
- How much does the brain fill in / edit before consciousness?
- Interesting perception phenomena per environment.
- What can't we measure yet, and what might we measure in the future?
- "Are we missing something?" → answer concretely, never mystically.

### 6. Endings

- Close every essay with exactly TWO "Questions for Further Research" that are specific, falsifiable, and forward-looking (privacy/ethics + a verification/trust question are strong defaults).

### 7. Scope & modularity

- When the user says "develop X as its own essay," produce a self-contained piece: thesis → evidence → model/analysis → stress test → future → two questions → Works Cited. Do not reference the parent document.
- When the user says "send only the new part," output ONLY that PDF.

## Contradiction ledger (resolved during consolidation)

| # | Conflicting instructions | Resolution |
| --- | --- | --- |
| 1 | "Calibri 12 + numbered titles" vs. an academic essay style that does not number section titles. | Adopt the clean academic look but keep numbered titles for navigability. |
| 2 | "a PDF I can download" vs. "attach the document here." | Always both generate AND attach the PDF in-conversation. |
| 3 | "make it more profound / paste exact source cut" vs. "remove AI slop / overly complex." | "Profound" = denser EVIDENCE, not denser prose. |
| 4 | Early outputs had no watermark vs. later "add watermark." | Watermark becomes a default-on, parameterized feature for all subsequent outputs. |
| 5 | Early free-form prose section headers vs. later numbered academic sections. | Fold free-form headers into numbered sections. |
| 6 | "Cameras detect sound" (recurring user premise) vs. physics. | Permanently correct: cameras detect light (EM radiation); microphones/sonar detect sound. |
| 7 | Implicit academic schooling header (author/instructor/course/date) vs. "this is independent work." | Use author + date only; no course/instructor line. |

## Discarded elements (and why)

- Per-sentence preview intros that duplicated list content: removed as AI slop.
- "In a meaningful sense / truly missing on" hedging language: removed; replaced with concrete, listed gaps.
- Any free-floating image/figure that did not carry a measurable signature: removed under the spectral-fingerprint test (decorative, not informative).
- Course-name and instructor/professor header lines: removed; this is independent work and must not look like a class submission.

## Delivery checklist (run before returning)

- [ ] Clean academic layout (author + date only, centered title, double-spacing).
- [ ] NO course or instructor/professor line anywhere.
- [ ] Calibri/Carlito 12 pt; numbered section titles present.
- [ ] Watermark applied (default "Matei-Ionuț Șuță", parameterized).
- [ ] Every claim anchored to a number or named source; ≥3 verbatim quotes.
- [ ] Stress-test done: slop removed, risk/counterargument block present, one new web-inspired stress idea added.
- [ ] Premise check (light vs. sound) stated.
- [ ] Exactly two Questions for Further Research.
- [ ] Works Cited page, hanging indent, alphabetized.
- [ ] PDF generated AND attached in-conversation; "only the new part" honored if asked.

## Expanded guidance, worked examples, and templates

### A. The "profound, not bloated" principle (worked example)

The single most common failure mode this skill guards against is mistaking verbal complexity for intellectual depth. Profundity is achieved by increasing the density of verifiable evidence, not by increasing the ornamental weight of the prose. Apply the following transformation to every paragraph.

**BEFORE (AI slop — vague, looping, decorative):**

> "In a very real and meaningful sense, the human brain is constantly and continuously working hard to construct a rich, seamless, and complete picture of the world around us, filling in countless gaps that we are not even aware of, in ways that are truly remarkable and profound."

**AFTER (stress-tested — one idea, sourced, measurable):**

> "The brain completes the receptor-free blind spot rather than reporting a gap. Spillmann et al. found that 'we perceive the brightness, color, and texture of the adjacent area as if they were actually there' (Vision Research 46.25, 2006)."

The AFTER version is shorter, contains a verbatim quote, names a source, and makes a single falsifiable claim. That is the target style for every paragraph.

### B. The cost-ladder pattern (reusable structure)

Whenever the user asks "how cheap/expensive is it," produce a tiered ladder so the reader can locate any budget on a single scale:

- Hobbyist (tens of dollars): IR-filter removal on a spare camera (~$10); a consumer laser rangefinder repurposed as crude LiDAR (~$70).
- Prosumer (hundreds): consumer thermal cameras, ultrasonic leak pens.
- Professional (thousands to ~$10k+): drone hyperspectral systems, professional ground-penetrating radar, acoustic imaging cameras.
- Research-lab (tens of thousands+): underwater LiDAR, synchronized multi-sensor fusion rigs.

State explicitly that no single device captures all bands; a "measure everything" system is a federation of instruments, and cost multiplies by band AND by location.

### C. The inverse-problem caveat (mandatory honesty)

Every sensing claim must acknowledge that detection is not interpretation. The same echo, emission, or reflectance can have multiple causes (a wall radar return may be pipe, void, cable, or layer boundary; an ultrasonic emission may be a leak, a bearing fault, arcing, or harmless turbulence). State that measurement must be cross-checked against plans, a second modality, or a trained human. This is what separates a credible report from a sensationalist one.

### D. Environment matrix (reuse when covering "where")

- Sky/space: JWST (infrared), Chandra (X-ray), VLA (radio), Hubble (UV–near IR); data released publicly; false-color compositing maps invisible bands to hues.
- Buildings: ground-penetrating radar + thermal → voids, pipes, moisture, heat loss.
- Forests/land: hyperspectral/multispectral drones → vegetation stress, species, fire risk, water content.
- Caves/underground: radar void detection in limestone; LiDAR geometry mapping.
- Underwater: sonar for long-range terrain (light dies fast); optical/laser LiDAR only in clear water; range collapses in turbidity.
- Audio everywhere: infrasound (<20 Hz) for slow motion/structural vibration; ultrasound (>20 kHz) for machine faults and leaks; bioacoustics for wildlife.

### E. Full essay skeleton (paste-ready ordering)

1. Author line and date only (no course, no instructor).
2. Centered title.
3. "1. Stress-Tested Thesis" — narrow, defensible, one paragraph.
4. Core numbered sections answering the Standard Question Set (Rule 5).
5. Source-Quote Anchors section — at least three verbatim, attributed quotes.
6. A perception-phenomena section (per environment, concrete not mystical).
7. "What we cannot measure yet" + "What we may measure in the future."
8. "Are we missing something?" — concrete, listed gaps.
9. (If requested) Business module with market size, ROI, model, and margin-killer stress test including a counterargument and rebuttal.
10. "Two Questions for Further Research" — exactly two; privacy/ethics + verification.
11. New page: "Works Cited," hanging indent, alphabetized.

### F. The web-inspired stress-test library (rotate one per revision)

- Spectral-fingerprint check: does invisible data yield a physical signature (like spectral absorption lines acting as atomic fingerprints) or is it merely pretty? Cut anything that fails.
- Falsifiability check: for each major claim, write the observation that would disprove it; if none exists, the claim is rhetoric, not evidence.
- Replication check: would an independent operator with the same instrument reach the same reading? If placement/calibration changes the answer, flag the uncertainty rather than hiding it.
- ROI-verification check (business): can savings be measured before AND after? If not, the value proposition is unsupported.
- Privacy check: would continuous measurement in this environment expose private behavior? If yes, mark what should remain unmeasured.

### G. Tone and style constraints

- Active voice; vary sentence length; no filler transitions ("it is important to note that").
- Never reproduce copyrighted text beyond brief quoted, attributed passages.
- Correct the user's premise where physics demands it (cameras = light, microphones/sonar = sound), but do so once and move on.
- Prefer concrete nouns and numbers to adjectives. Delete any adjective that does not change a decision the reader could make.

### H. Parameters (expose to the user)

- `watermark_text` (default "Matei-Ionuț Șuță")
- `include_business_module` (default: only if profit/market is requested)
- `standalone_mode` (default off; on when "make it its own essay")
- `send_only_new_part` (default off; on when explicitly requested)

### I. Example invocations → expected behavior

- "Make a report on what cameras and mics detect beyond human senses." → Full PDF, Standard Question Set, three+ quotes, two questions, watermark.
- "Stress test it." → Re-run the stress-test procedure (Rule 3), add one new stress idea, regenerate watermarked PDF, attach.
- "Add a business angle." → Append Business module with market size, ROI, service-first model, margin-killer stress test + rebuttal.
- "Develop the business part as its own essay, send only that." → standalone_mode + send_only_new_part; self-contained PDF, own Works Cited.

### J. Failure modes to avoid (negative examples)

- Returning prose in the chat when a PDF was requested.
- Using Times New Roman or omitting Calibri/Carlito.
- Adding a course or instructor/professor header (this is independent work).
- Dropping the numbered titles (they are explicitly retained).
- Forgetting the watermark on a regenerated file.
- Letting a claim stand without a number or source.
- Writing three questions, or zero, instead of exactly two.
- Reproducing the parent document when a standalone essay was requested.
