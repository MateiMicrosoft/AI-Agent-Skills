---
name: extract-prompt
description: "Live-discovery corpus-to-atoms extractor. Use to turn any subject into a clean evidence bundle for downstream essay/report writing — building its own corpus from live web pages, store catalogs, app reviews, forums, and opinion threads when the user supplies no sources. Runs in three modes: argument (disagreeable questions), commerce (products, recipes, prices, store/locale binding), and opinion (apps, brands, reviews with triangulation). Produces an Idea Essay M0 hand-off bundle plus an audit. Subject in, atoms out."
license: CC-BY-NC-4.0
metadata:
  author: Matei-Ionuț Șuță
  version: '2.0'
---

# Extract Prompt 2.0

Live-discovery sibling of the Idea Essay Prompt. A six-module pipeline that turns a subject into evidence atoms. The prompt can build its own corpus when the user supplies none — from live web pages, store inventories, app reviews, niche forums, and opinion threads. Subject → atoms. Nothing else.

Version 1.2 assumed the user hands in videos, papers, and chats. 2.0 keeps that path and adds three new evidence classes: commerce (store catalogs, ingredient lists, prices, locations), community opinion (Reddit, app reviews, Trustpilot, Discord), and emerging-topic discovery (subjects with no canonical corpus, e.g. AI-designed cat toys). The hand-off contract to Idea Essay M0 is unchanged.

## When to Use This Skill

Use this skill when you need to transform a subject — with or without attached sources — into a tight, sourced, mode-aware evidence bundle ready for an essay, report, presentation, or operational brief. It is the inspection/extraction front-end: it gathers and atomizes evidence and stops. Essay drafting, rendering, and final audit belong to the downstream Idea Essay Prompt.

## What 2.0 adds over 1.2

| Capability (new in 2.0) | Where it lives |
| --- | --- |
| Live source discovery from a bare subject (no attachments required) | E1 — Corpus assembly |
| Commerce mode: store catalogs, SKUs, ingredient lists, geo-availability | E1 role tag commerce; E2 atom types |
| Opinion mining: forums, app reviews, social threads with sentiment tagging | E1 role tag opinion; E2 sentiment + consensus flag |
| Locale binding: city, country, store-chain, currency, language | E0 — locale slot |
| Evidence triangulation: an opinion atom must be backed by ≥2 independent threads | E2 — triangulation rule |
| Freshness floor: tier-2 and below sources must carry a date and decay weight | E1 — freshness; E2 — decay |
| Refusal-to-fetch on subjects requiring expert credentialed sources | E0 — gate |

## Three discovery modes (set at E0)

The prompt runs in one of three modes depending on what the subject is asking for. Mode is set at E0 and binds every later module's source-tier rules and atom-type vocabulary.

| Mode | Triggered by | Tier-1 sources | What atoms look like |
| --- | --- | --- | --- |
| Argument (default, same as 1.2) | A disagreeable question about a phenomenon or idea | Peer-reviewed papers, official data, first-hand transcripts | Claim, mechanism, contradiction, decision |
| Commerce | Subject names a product, recipe, ingredient, store, or location | Retailer's own catalog page, official product page, manufacturer spec, government food database | SKU, ingredient, price (with currency + date), availability, substitute |
| Opinion | Subject names an app, brand, service, community, or experience | Aggregated review platforms with verified-purchase filters; official changelogs; long-form first-hand accounts | Sentiment, consensus, dissent, recurring complaint, recurring praise |

## Pipeline at a glance

Same six modules. E0 now also picks the mode and the locale. E1 can build a corpus from scratch via live fetches. E2 carries mode-specific atom types. E3–E5 are unchanged in shape but tightened so that opinion and commerce evidence cannot impersonate primary fact.

| # | Module | What it does (2.0) |
| --- | --- | --- |
| E0 | Subject framing + mode + locale | Name the disagreeable question (argument mode) or the operational question (commerce / opinion mode). Pick mode. Bind locale. Name the anti-question. |
| E1 | Corpus assembly (live-capable) | If sources are attached, ingest. If not, generate a source plan and fetch — with role, tier, weight, freshness, locale, atom cap. |
| E2 | Atom extraction (mode-aware) | Evidence only. Per-source caps. Density floor. Mode-specific atom types. Opinion atoms require triangulation. |
| E3 | Cluster + connect | Group by mechanism / contradiction / scale (argument) or by decision-rule / substitution / failure-mode (commerce, opinion). |
| E4 | Combat pass | Argue against the framing question. In commerce / opinion, the argument is operational: which option wins, under which constraint. |
| E5 | Hand-off | Compress to Idea Essay M0 bundle. Sufficiency strong / weak. Cap 90 atoms. Refuse hand-off if two of three axes are weak. |

## The modules

Each module is one transformation. Input, process, output, failure mode is the contract. A module that cannot name what it produces or how it fails does not belong in the pipeline.

### E0 — Subject framing + mode + locale

Name the question. Pick the mode. Bind the place.

- **INPUT:** User's subject in any language. Optional attachments. Optional locale hints (city, country, store, brand, app).
- **PROCESS:**
  1. State the subject in one sentence the user would actually use.
  2. Pick mode: argument, commerce, or opinion (rules above). Record the trigger phrase that decided it.
  3. Bind locale: country, city, language, currency, store/brand/app if named. If absent and the mode is commerce or opinion, ask once — do not infer.
  4. Convert the subject into a disagreeable question (argument) or an operational question (commerce / opinion) — one a reasonable reader could answer the opposite way or operate on differently.
  5. State the anti-question: the framing the prompt will explicitly argue against, or the operational path it will explicitly reject.
  6. Lift every side-comment into a one-line binding rule.
  7. Detect language; carry to E5.
  8. Refusal gate: if the subject requires credentialed expertise (medical diagnosis, legal advice, financial advice on a specific instrument), refuse the mode and downgrade to argument with an explicit hedge.
- **OUTPUT:** A single question, a single anti-question, a mode tag, a locale block, a rules file, a language tag.
- **FAILS WHEN:** Restates the topic as a question (what is X?). Picks commerce mode without binding a store or country. Picks opinion mode without naming the app, brand, or community.

### E1 — Corpus assembly (live-capable)

Build the corpus if the user did not. Tag role, tier, weight, freshness, locale.

- **INPUT:** The E0 question, mode, and locale. Optional attached sources. Optional Source-Truth Pack.
- **PROCESS:**
  1. If sources are attached, normalize each: title, source type, length or page count, publication or recording date. Skip to step 3.
  2. If no sources are attached, build a source plan:
     - Argument mode: 2–4 primary (papers, official data, first-hand), 3–6 secondary (established outlets), 0–4 tertiary (opinion, forum) as background.
     - Commerce mode: the retailer's own catalog page (primary), product/manufacturer spec (primary), 1–2 official food / safety / regulatory databases (primary), 1–3 secondary recipe or culinary references, locale-binding required.
     - Opinion mode: 1–2 first-party changelogs or official statements (primary), 1–2 review aggregators with verified-purchase or long-form filter (secondary), 3–8 forum threads or app reviews (tertiary).
  3. Tag role first: evidence (extract atoms), instruction (binding rule), reference (on-demand), meta (ignore), commerce (catalog / inventory; extract as evidence with commerce-atom vocabulary), opinion (community sentiment; extract as evidence with triangulation rule).
  4. Tag trust tier: primary, secondary, tertiary.
  5. Mark binding weight: full / half / quarter, based on how directly it speaks to E0.
  6. Tag freshness: every secondary or tertiary source must carry a date. Apply decay: a source older than 24 months drops one weight tier in commerce or opinion mode; older than 60 months drops two.
  7. Compute per-source atom cap: full = 60, half = 30, quarter = 15.
  8. Flag duplicates, in-ledger sources, and locale mismatches (e.g. a US Lidl catalog used for a Bucharest query is locale-mismatched and demoted one tier).
  9. Emit rules file for instruction-tagged items.
- **OUTPUT:** Annotated manifest: source, role, type, tier, weight, freshness, locale, atom_cap, date, duplicate-of, in-ledger. Plus the rules file.
- **FAILS WHEN:** Treats a 3-hour video and a 2-page note as equal weight. Treats an opinion thread as primary because it is long. Uses a non-locale catalog page in commerce mode without demoting. Skips freshness tag on a tertiary source in opinion mode.

### E2 — Atom extraction (mode-aware)

Evidence only. Per-source caps. Density floor. Mode-specific atom types. Triangulation for opinion.

- **INPUT:** The E1 manifest, filtered to evidence, commerce, and opinion roles.
- **PROCESS:**
  1. Skip non-evidence sources. Pull atoms from evidence, commerce, and opinion roles only.
  2. From each source, pull atoms in the mode-specific vocabulary:
     - Argument: claim, mechanism, example, contradiction, unfinished question, decision.
     - Commerce: SKU, ingredient, price (currency + date + store), availability, pack-size, substitute, dietary tag, allergen, certification.
     - Opinion: sentiment (positive / negative / mixed), consensus claim, dissent claim, recurring complaint, recurring praise, edge case.
  3. One row per atom. ≤ 25 words. Compress; do not paste verbatim.
  4. Record provenance: source ID + timestamp (audio / video) or page (text) or thread permalink + post date (forum / review).
  5. Tag each atom: known (tier-1 fact), inferred (two tier-2 or one tier-1), speculative (tier-3 or contradicted).
  6. Triangulation rule (opinion mode): an opinion atom may only be tagged consensus if it appears in at least two independent threads or platforms. Otherwise it is tagged single-voice and demoted to speculative.
  7. Locale rule (commerce mode): any price, availability, or store-specific atom must carry the locale block from E0; an atom missing locale is rejected, not softened.
  8. Respect per-source atom_cap mechanically.
  9. Apply density floor: ≥ 1 atom per 5 min video, 1 atom per page text, 1 atom per 500 words transcript, 1 atom per 10 reviews in opinion mode, 1 atom per catalog page in commerce mode. Below floor → flag thin, demote one weight tier, recompute cap, re-extract. Above 2× floor → flag rich, cap +50 percent.
- **OUTPUT:** Atom list: id, source pointer, atom text (≤ 25 words), mode-tag, known / inferred / speculative, consensus / single-voice (opinion), locale (commerce), density flag, final atom_cap honored.
- **FAILS WHEN:** Treats a single Reddit thread's enthusiasm as consensus. Records a price without store + date + currency. Extracts atoms from instruction or meta. Skips the density floor; a 200-review app yields 4 atoms and passes silently.

### E3 — Cluster + connect

By mechanism (argument), by decision-rule (commerce), by recurrence (opinion). Never by topic.

- **INPUT:** The atom list from E2.
- **PROCESS:**
  1. Group atoms by the right axis for the mode:
     - Argument: mechanism, contradiction, scale.
     - Commerce: decision-rule (what swaps for what), substitution chain, constraint (budget, dietary, allergen, availability), failure-mode (out of stock, wrong locale).
     - Opinion: recurring complaint, recurring praise, edge-case failure, version-drift (when the product changed).
  2. Refuse to cluster by topic. Topic buckets are reference categories, not arguments.
  3. Name each cluster in ≤ 5 words. The name must contain a verb or a comparative or a decision phrase — not just a noun.
  4. Surface the three strongest cross-source connections.
  5. Flag single-source clusters.
- **OUTPUT:** Cluster map: cluster name, member atoms, cross-source connections, single-source flag.
- **FAILS WHEN:** Clusters become topic buckets (fruits, batteries, UX). Names lack a verb or comparative. Single-source clusters pass without flag.

### E4 — Combat pass

Argue with the question. In commerce / opinion, argue an operational decision.

- **INPUT:** Cluster map from E3 + E0 question, anti-question, mode, locale.
- **PROCESS:**
  1. For each cluster, write one paragraph that argues — against the framing question, against the anti-question, or against another cluster. The paragraph takes a position.
  2. In commerce mode, the argued position names a concrete decision: which recipe wins, which substitute beats the original, which constraint dominates. The paragraph must name a binding constraint (price, locale, dietary, allergen) — not just preference.
  3. In opinion mode, the argued position names which recurring claim survives triangulation and which collapses on inspection. Reject the implicit consensus reading.
  4. Reject balanced pro-con framing. A pro-con list is a survey, not an argument.
  5. Mark every empirical claim with supporting atom IDs. Claims without atom support are deleted, not softened.
  6. Require one counterpressure paragraph: the strongest case the combat pass itself is wrong.
  7. Cap each paragraph at 120 words.
- **OUTPUT:** A combative draft per cluster, each linked to atom IDs, plus one counterpressure paragraph.
- **FAILS WHEN:** Outputs a balanced pro-con list. Names a winner in commerce mode without a binding constraint. Calls a single-voice opinion thread a consensus.

### E5 — Hand-off

Compress to the Idea Essay M0 bundle. Score sufficiency. Refuse if weak.

- **INPUT:** All upstream output: question, manifest, atom list, cluster map, combative draft, locale, mode.
- **PROCESS:**
  1. Build the bundle: thesis seed (one sentence from the strongest combat paragraph; in commerce / opinion modes, the seed names the operational claim), evidence pool (atoms tagged known or inferred; opinion atoms must be consensus-tagged), scope (one sentence on what the essay must cover and what it must not — including locale boundary), suggested mode for the essay (scientific, article, presentation, or operational brief).
  2. Score sufficiency on three axes — thesis seed, evidence pool, scope — using strong / weak.
  3. If two of three are weak, refuse hand-off. Return to E1 with a note on the missing tier or locale.
  4. Cap bundle at 90 atoms. Above 90, force a second E2 compression pass.
  5. Opinion-mode guardrail: if more than 60 percent of the evidence pool is tier-3, mark the bundle as opinion-heavy and downstream M2 must hedge its thesis accordingly.
  6. Commerce-mode guardrail: every priced or available-at atom must still carry locale + date at hand-off. Strip any that lost the tag in compression and re-extract.
  7. Emit the bundle in Idea Essay M0 input format. Nothing else.
  8. Record hand-off: timestamp, bundle ID, scores, atom count, mode, locale.
- **OUTPUT:** An Idea Essay M0 input bundle with a sufficiency score and a one-line hand-off record.
- **FAILS WHEN:** Generous sufficiency because the corpus was large. Hands off an opinion-heavy bundle without the flag. Hands off a commerce bundle whose prices lost their date or locale in compression.

## Sufficiency score — what counts as strong (2.0)

1.2's three axes still hold. 2.0 adds mode-specific minimums so the score does not drift toward "large enough" in commerce or opinion mode, where corpora are easy to inflate.

| Axis | Strong when | Weak when |
| --- | --- | --- |
| Thesis seed | One combat paragraph names a position (argument) or a binding operational decision (commerce / opinion) a reasonable reader could hold the opposite of. | No paragraph stakes a position; all paragraphs survey. |
| Evidence pool — argument | ≥ 25 atoms tagged known or inferred, from ≥ 3 sources, with ≥ 1 tier-1. | < 25 atoms, single-source domination (one source > 60 percent), or no tier-1. |
| Evidence pool — commerce | ≥ 20 atoms; ≥ 1 retailer catalog (primary) for the bound locale; every priced atom carries currency + date + store. | No retailer-of-record source for the bound locale; or any priced atom missing currency / date / store. |
| Evidence pool — opinion | ≥ 20 atoms; ≥ 3 independent threads or platforms; consensus atoms triangulated across ≥ 2 of them; ≥ 1 first-party source (changelog, official statement). | All atoms from one platform; no first-party source; consensus atoms not triangulated. |
| Scope | One sentence on what the essay must cover; one sentence on what it must not; locale stated explicitly if mode is commerce or opinion. Written before drafting. | Scope is implicit, post-hoc, or omits locale in commerce / opinion mode. |

## New cross-module failure modes (2.0 only)

| Failure | How it looks | Which modules let it through |
| --- | --- | --- |
| Locale leak | A US-Lidl catalog answers a Bucharest-Lidl question; prices and SKUs do not exist in the bound locale. | E0 (no locale bind), E1 (no locale demotion), E2 (no locale tag on atom) |
| Opinion-as-fact | A single Reddit thread's enthusiasm appears in the bundle as a consensus claim. | E1 (no opinion role), E2 (no triangulation), E5 (no opinion-heavy flag) |
| Staleness drift | A 2019 review of a 2024 app dominates the evidence pool; the product has since changed. | E1 (no freshness tag), E2 (no decay applied) |
| Catalog inflation | A retailer's catalog page yields 200 SKU atoms and crowds out every other source. | E1 (full weight on a catalog), E2 (no per-source cap on commerce sources) |
| Mode confusion | A commerce question is run in argument mode; the bundle has no priced atoms and no locale. | E0 (mode not chosen or wrongly chosen) |

## Master prompt (paste-ready)

Run as the system instruction. Pass subject and (optional) source list as a user message. Pass side-comments as binding rules. The output of E5 is the user message passed to the Idea Essay Prompt.

```
You are a corpus-to-atoms extractor with live-discovery capability.
Run the input through six modules in order. Do not skip a module.

E0 SUBJECT FRAMING + MODE + LOCALE.
    State the subject in one sentence the user would actually use.
    Pick MODE: argument | commerce | opinion. Record the trigger phrase.
    Bind LOCALE: country, city, language, currency, store/brand/app. If missing
    and mode is commerce or opinion, ask once. Do not infer.
    Convert subject to a disagreeable question (argument) or an operational
    question (commerce/opinion). State the anti-question.
    Lift every side-comment into a binding rule. Detect language; carry to E5.
    REFUSAL GATE: if credentialed-expertise required (medical, legal, financial
    instrument advice), downgrade to argument mode with an explicit hedge.

E1 CORPUS ASSEMBLY (live-capable).
    If sources are attached, normalize them. If not, build a source plan:
      argument: 2-4 primary, 3-6 secondary, 0-4 tertiary.
      commerce: retailer catalog (primary) for bound locale,
                manufacturer spec, food/regulatory database,
                1-3 secondary culinary refs.
      opinion: 1-2 first-party (changelog, statement),
                1-2 review aggregators, 3-8 forum threads / app reviews.
    TAG ROLE FIRST: evidence | instruction | reference | meta | commerce | opinion.
    Tag trust tier: primary | secondary | tertiary.
    Tag binding weight: full | half | quarter.
    Tag freshness (date required for tier-2 and below). Apply decay:
        older than 24 months -> demote one weight tier (commerce/opinion).
        older than 60 months -> demote two.
    Compute atom_cap: full=60, half=30, quarter=15.
    Flag duplicates, in-ledger, and LOCALE MISMATCHES (demote one tier).
    Emit rules file for instruction-tagged items.

E2 ATOM EXTRACTION (mode-aware).
    Pull atoms from evidence, commerce, and opinion roles only.
    Atom vocabulary by mode:
      argument: claim | mechanism | example | contradiction | question | decision.
      commerce: sku | ingredient | price | availability | pack-size |
               substitute | dietary | allergen | certification.
      opinion: sentiment | consensus | dissent | complaint | praise | edge-case.
    One row per atom, <= 25 words. Record provenance (timestamp, page, or
    thread permalink + post date).
    Tag known | inferred | speculative.
    TRIANGULATION (opinion): consensus tag requires >= 2 independent threads.
    LOCALE (commerce): every priced or availability atom carries currency + date
    + store from E0. Missing locale -> reject the atom, do not soften.
    Respect per-source atom_cap mechanically.
    Density floor: 1 atom / 5 min video, 1 / page, 1 / 500 words,
    1 / 10 reviews (opinion), 1 / catalog page (commerce).
    Below -> thin, demote, recompute, re-extract. Above 2x -> rich, cap +50%.

E3 CLUSTER + CONNECT.
    Group by mechanism / contradiction / scale (argument), decision-rule /
    substitution / constraint / failure-mode (commerce), or recurring complaint /
    praise / edge-case / version-drift (opinion).
    Refuse topic buckets. Cluster name must contain a verb, comparative,
    or decision phrase. Surface 3 strongest cross-source connections.
    Flag single-source clusters.

E4 COMBAT PASS.
    For each cluster, draft one paragraph that argues, not surveys.
    Commerce: argue a concrete decision bound by a constraint.
    Opinion: argue which recurring claim survives triangulation.
    Mark every empirical claim with supporting atom IDs. Cap 120 words.
    Require one counterpressure paragraph.

E5 HAND-OFF.
    Build the Idea Essay M0 bundle: thesis seed, evidence pool, scope,
    suggested mode (scientific | article | presentation | operational brief).
    Score sufficiency strong / weak per axis (argument | commerce | opinion rule).
    Refuse handoff if 2 of 3 are weak — return to E1.
    Cap bundle at 90 atoms.
    Opinion-heavy guardrail: if > 60% of atoms are tier-3, flag opinion-heavy.
    Commerce guardrail: every priced atom keeps currency + date + store + locale.
    Emit bundle in M0 input format. Record handoff.

REQUIRED INPUTS:
  subject:         one sentence.
  sources:         optional. If empty, run live-discovery per mode.
  locale:          optional unless mode is commerce or opinion.
  side_comments:   optional list of binding rules.
  past_ledger:     optional Source-Truth Pack reference.
  output_language: ISO code or 'match_input'.

EMITTED OUTPUT:
  bundle.json   -- the Idea Essay M0 input bundle.
  audit.json    -- manifest, atom list, cluster map, combat draft,
                sufficiency score, mode, locale, handoff record.
```

## Three worked examples

How E0 sets mode and locale on the three kinds of subjects the user raised. Each example ends at the hand-off bundle — the Idea Essay Prompt picks it up from there.

### Example 1 — AI-designed cat toys

Emerging subject, sparse corpus. Argument mode with opinion supplements.

- **E0:** Subject: AI-designed cat toys. Question: Does AI-led toy design produce safer or more engaging cat toys than human-led design, or does it mostly automate novelty without raising the floor on safety? Anti-question: AI-designed cat toys are a marketing label with no measurable effect on safety or engagement. Mode: argument. Locale: not required (subject is global).
- **E1 plan:** Primary: pet-product safety regulators (CPSC, EU REACH summaries on phthalates and small parts) and any peer-reviewed feline-enrichment studies. Secondary: established pet-trade publications, veterinary-journal commentary, manufacturer R&D notes. Opinion: 4–6 long Reddit / r/CatAdvice threads and 2–3 review aggregators (Chewy verified-purchase reviews) — triangulation required.
- **E2 atom types:** Argument atoms (claim, mechanism, contradiction) from primary and secondary. Opinion atoms (recurring complaint about choking-risk, recurring praise for unpredictable motion patterns) from tertiary, only with consensus tag if triangulated.
- **E4 stance:** AI-led design raises novelty (motion patterns, scent permutations) but does not move the safety floor; safety is set by material and small-parts regulation that pre-dates the design step. Counterpressure: a single brand has used AI to model fracture risk on chew toys; if that diffuses, the claim weakens.
- **E5:** Bundle: thesis seed as above, evidence pool ≥ 25 atoms with at least one primary safety source, scope explicitly excludes dog and small-mammal toys. Opinion-heavy flag fires if tier-3 atoms exceed 60 percent — likely on this subject — and M2 must hedge.

### Example 2 — Sweet dish from Lidl, Bucharest

Commerce mode. Locale-bound. Operational question.

- **E0:** Subject: What sweet dish can I make using fruits and ingredients from Lidl in Bucharest? Operational question: Given current Lidl-Bucharest inventory and an under-RON-40 budget, which sweet dish maximizes seasonality and minimizes ingredient count? Anti-question: The best sweet dish requires ingredients Lidl-Bucharest does not stock; substitute elsewhere. Mode: commerce. Locale: Romania / Bucharest / Lidl / RON / Romanian.
- **E1 plan:** Primary: Lidl Romania catalog (lidl.ro) — current weekly flyer (Brosura) and online assortment, with date. Manufacturer specs for own-brand items if cited. Secondary: 2–3 Romanian culinary references for traditional dishes (e.g. plăcintă cu mere, papanaşi, tartă cu fructe). Opinion: optional 1–2 Lidl-Bucharest review threads on fruit freshness.
- **E2 atom types:** Ingredient atoms (apples, plums, telemea, smântână) each carrying SKU, RON price, weekly-flyer date, store-of-record. Substitute atoms (Lidl smântână 20 percent vs. crème fraîche). Allergen atoms (gluten, dairy). Locale-missing atoms are rejected, not softened.
- **E4 stance:** Under RON 40 and current flyer dates, a four-ingredient apple-and-cinnamon tart wins on seasonality and ingredient count; papanaşi loses on the brânză-de-vaci constraint, which Lidl stocks inconsistently. Counterpressure: if the flyer rotates plums into season, the tart base swaps.
- **E5:** Bundle: thesis seed as above, evidence pool with every priced atom carrying currency + date + store, scope binds to Lidl-Bucharest and excludes other chains. Suggested essay mode: operational brief.

### Example 3 — Forum / opinion mining for an app

Opinion mode. Triangulation enforced.

- **E0:** Subject: Is the latest version of [App X] worse than the previous one? Operational question: Across at least three independent communities, which recurring complaint about the latest [App X] release survives triangulation, and which collapses on inspection? Anti-question: The version-regression narrative is single-platform noise. Mode: opinion. Locale: language and platform bound (e.g. English / iOS / r/AppX / App Store).
- **E1 plan:** First-party: the changelog or release notes for the version under review. Secondary: 1–2 long-form review pieces dated after the release. Tertiary: 3–8 forum threads (Reddit, Discord, dedicated forum) and the App Store / Play Store review feeds, all dated after the release.
- **E2 atom types:** Complaint atoms (specific feature regressed) and praise atoms (specific feature improved). Consensus tag requires ≥ 2 independent threads or platforms — a single subreddit does not count. Atoms older than the current release are rejected; freshness is binding here.
- **E4 stance:** The recurring complaint about [feature Y] survives across three platforms and matches a documented changelog removal; the recurring complaint about [feature Z] collapses to a single thread amplified by replies and does not appear in App Store reviews. Counterpressure: the App Store reviews skew toward stability complaints, not feature complaints, which may understate feature regressions.
- **E5:** Bundle: thesis seed names which complaint survives, evidence pool flagged opinion-heavy (likely > 60 percent tier-3), scope binds platforms and excludes pre-release atoms. Suggested essay mode: article with hedged thesis.

## Operating discipline

Extended for live discovery, commerce, and opinion.

**Do (green):**

- Set mode at E0 before fetching anything. Mode binds atom vocabulary.
- Bind locale before any commerce or opinion fetch. Ask once if missing; do not infer.
- Tag freshness on every tier-2 and tier-3 source. Apply decay mechanically.
- Triangulate opinion atoms across at least two independent platforms before tagging consensus.
- Keep currency, date, and store on every priced or availability atom from end to end.
- Hand off a smaller, sharper bundle. 90 atoms is a ceiling, not a target.

**Be careful (yellow):**

- Long forum threads look like rich evidence; without triangulation they are still single-voice.
- Retailer catalogs inflate atom counts fast; the per-source cap is mechanical, not advisory.
- An English-language review of a locale-specific product is locale-mismatched until proven otherwise.
- Aggregator scores compress away dissent. Pull the dissent atoms back in by hand.
- Cross-references break under page reflow; verify after the Idea Essay renders.

**Avoid (red):**

- Inventing studies, statistics, prices, dates, store names, or quoted reviews.
- Running commerce mode without a bound locale.
- Tagging a single Reddit or App Store thread as consensus.
- Letting a stale review of an old version dominate an opinion bundle for the latest version.
- Mixing roles: an instruction or side-comment must not become an atom.
- Handing off > 90 atoms because the downstream prompt "can handle it" — its M4 cannot.

## Contract with the Idea Essay Prompt

The hand-off format is unchanged. The Idea Essay Prompt's M0 reads a bundle with thesis seed, evidence pool, scope, and suggested mode; 2.0 adds mode (argument / commerce / opinion), locale, and optional opinion-heavy and commerce-locale-bound flags inside the same envelope. When the Idea Essay Prompt sees those flags, M2 hedges its thesis, M4 keeps locale and date on every commerce claim, and M7's stress test requires that no opinion claim travels as a known fact.

Decision rule: if a step transforms one kind of artifact into another, it earns its own prompt. 2.0 transforms heterogeneous sources — now including live-fetched ones — into a clean atom bundle and stops there. Essay drafting, rendering, and final audit remain the Idea Essay Prompt's job.
