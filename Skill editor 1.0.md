---
name: prompt-editor
description: "Behavior-preserving editor for shipped prompt-skills, PDF prompt packs, and agent instructions. Use to compress, expand, refactor, retarget, extend, lint, migrate, or merge an existing prompt without silently changing its contract (inputs, outputs, refusal logic, schema semantics, few-shot order, parseability). Takes an existing prompt plus an edit instruction and produces a new version with a structured audit object and a symmetric diff manifest. Uses Extract Prompt 2.0 as its diagnostic engine: the prompt under edit is the corpus, its modules are evidence, its rules are atoms, and the edit instruction is the disagreeable question. Absorbs Prompt Compressor 1.1."
license: CC-BY-NC-4.0
metadata:
  author: Matei-Ionuț Șuță
  version: '1.0'
---

# Prompt Editor 1.0 — behavior-preserving prompt editing

A general-purpose editor for shipped prompt-skills, PDF prompt packs, and agent instructions. It does not start from a blank page. It takes an existing prompt and a user edit instruction and produces a smaller, larger, refactored, retargeted, merged, or migrated version — without silently changing the prompt's contract: inputs, outputs, refusal logic, schema semantics, few-shot order, and parseability.

Prompt Editor 1.0 absorbs Prompt Compressor 1.1. Compression becomes one of eight edit operations. The diagnostic engine is Extract Prompt 2.0: the prompt under edit is treated as a corpus, its modules as evidence sources, its rules as atoms, and the user's edit instruction as the disagreeable question. Extract 2.0 produces the audit map; the Editor produces the rewrite.

## When to Use This Skill

Use this skill whenever a user wants to change an existing prompt, skill, or agent instruction — make it shorter, longer, restructured, retargeted to a new audience or model, extended with a new capability, merged with another prompt, migrated to a different contract format (XML/JSON/Markdown, OpenAI/Claude/MCP tools), or simply audited (lint) — while guaranteeing that the prompt's executable behavior survives unless the user explicitly names which part of the contract may change.

## Why this exists (engineering note)

Three problems showed up in production with Compressor 1.1:

- Operators wanted to shorten prompts; they ended up needing to retarget, extend, merge, and migrate them. The compressor said no.
- Edits were being applied to prompts no one had audited first. The compressor's audit fires at the end. The damage starts earlier — when an editor cannot tell which span is load-bearing.
- Two sibling prompts (Extract 1.2, Idea Essay 3.4) already encode a clean corpus → atoms → cluster → argument pipeline. Reusing them as the editor's diagnostic engine is cheaper than rebuilding analysis inside the editor.

The fix is a separation of concerns. Extract 2.0 inspects. Editor edits. Compressor's lock-mapping and structured-audit object survive — they become the editor's Behavior-Preservation Pass and apply to every edit operation, not only compression.

## Core rule

Editing is not authoring. An author can drop the original. An editor must preserve the executable behavior of the original prompt unless the user's edit instruction explicitly names which part of the contract changes.

The editor prefers behavior preservation over the requested edit. No output ships without a structured audit object and a diff manifest. If the audit fails or the diff cannot be reconciled with declared locks, return a refusal or a rollback note rather than the edit.

## The eight edit operations

Every user instruction maps to one or more of these eight operations. Mode is set at intake (E0 in Extract 2.0 terminology). Multi-operation edits run in the order declared.

| Op | What it does | Compressor 1.1 equivalent |
| --- | --- | --- |
| compress | Reduce tokens; preserve contract. | Full pipeline (C0–C10) |
| expand | Add missing modules, examples, or rules the audit shows are absent or thin. | Not supported |
| refactor | Restructure (split a module, merge two modules, rename a rule, lift globals) without changing behavior. | Partial: C5, C7, C8 |
| retarget | Change audience (human → AI), language, or task_type. Behavior preserved; surface form changes. | Partial: audience flag |
| extend | Add new capability (new mode, new tool, new refusal trigger). Adds atoms; never silently removes. | Not supported |
| lint | Audit-only. No rewrite. Report failures, weak axes, missing locks, dead rules. | Not exposed standalone |
| migrate | Move the prompt between contracts (e.g. Idea Essay M0 input → Anthropic XML; OpenAI tools → Claude tools). | Not supported |
| merge | Combine two prompts into one, resolving rule collisions via declared precedence. | Not supported |

## How Extract 2.0 plugs into the editor

The editor's first half is Extract 2.0 with one binding change: the corpus is always exactly one source — the prompt under edit — and the framing question is always the user's edit instruction. Modules E0–E5 produce the inspection bundle. The editor's second half (X0–X4 below) consumes that bundle and emits the rewrite.

```
USER EDIT INSTRUCTION + ORIGINAL PROMPT
        |
        v
[ EXTRACT 2.0 — diagnostic engine ]
  E0 frame edit as a disagreeable question; pick op; bind locale/audience
  E1 treat the prompt as a single 'full-weight evidence' source; tag every
     module as a sub-source (role: rule | refusal | schema | example |
     format | prose | filler)
  E2 atomize: each rule, refusal trigger, schema field, example becomes an atom
     with provenance (module + line range)
  E3 cluster atoms by behavior (decision-rule, refusal, schema-shape,
     style, dead-code)
  E4 combat pass: which clusters resist the edit, which welcome it
  E5 hand-off: emit inspection bundle (atoms, locks, free spans, dead spans,
     sufficiency score)
        |
        v
[ EDITOR — rewrite engine ]
  X0 intent classification (which op, with what parameters)
  X1 edit plan (atom-level: keep / rewrite / delete / add / move)
  X2 apply edit; produce a structured diff against the inspection bundle
  X3 behavior-preservation pass (Compressor 1.1's lock-mapping +
     audit, generalized to all ops)
  X4 refuse, roll back, or ship
```

## Module map

| # | Module | Job | Output |
| --- | --- | --- | --- |
| E0 | Intent framing (Extract 2.0) | Frame edit as a disagreeable / operational question. Pick op. Bind audience, language, locale, task_type. | edit_intent |
| E1 | Prompt-as-corpus assembly | Tag every module/section by role: rule, refusal, schema, format, example_primary, example_edge, example_shape, prose, filler. | manifest |
| E2 | Atom extraction | One atom per rule, refusal trigger, schema field, example, format directive. Carry provenance. | atoms[] |
| E3 | Cluster + connect | Group atoms by behavior axis. Surface dead atoms (referenced nowhere) and load-bearing atoms (referenced ≥2x). | clusters[] |
| E4 | Combat pass | For each cluster, decide whether the edit applies, must skip, or must rewrite. | combat_map |
| E5 | Hand-off | Emit the inspection bundle to X0. Sufficiency strong/weak per axis. Refuse if 2 of 3 weak. | inspection_bundle |
| X0 | Intent classification | Confirm the op declared at E0 matches what the diff will actually do. Reject silent op-drift. | op_lock |
| X1 | Edit plan | Atom-level plan: keep / rewrite / delete / add / move. One row per atom touched. | edit_plan |
| X2 | Apply + diff | Produce the new prompt and a structured diff manifest against the inspection bundle. | diff, new_prompt |
| X3 | Behavior-preservation pass | Compressor 1.1's audit, generalized: locks, negations, schema, refusal triggers, examples, format, unicode, plus the new op-specific checks. | audit |
| X4 | Ship or refuse | Ship on full pass. Refuse with reason and recommended retry on any hard failure. | result |

## E0 — Intent framing

Same shape as Extract 2.0 E0. The disagreeable question is the user's edit instruction restated so a reasonable reviewer could disagree with the proposed change. The anti-question is the surface reading of the instruction the editor will explicitly reject.

```json
{
  "op": "compress | expand | refactor | retarget | extend | lint | migrate | merge",
  "edit_question": "<disagreeable restatement of the user instruction>",
  "anti_question": "<the surface reading the editor refuses>",
  "audience": "human_readable | ai_only",
  "language": "<ISO code or 'match_input'>",
  "locale": "<optional; required for retarget / migrate>",
  "task_type": "general | structured_output | refusal_safety | few_shot | retrieval | tool_use | code_agent",
  "locks": ["<exact span or semantic rule that must survive>"],
  "ratio": 0.50,
  "target": "<module-name or 'global'>"
}
```

`ratio` is compress-only. `target` is refactor / extend only.

**Fails when:** op is missing; edit_question is a restatement of the user instruction without inversion (any reviewer would agree); retarget or migrate declared without a locale or contract; compress declared with ratio < 0.10 or > 0.90.

## E1 — Prompt-as-corpus assembly

The original prompt is normalized into a manifest. Every section, callout, code block, and example becomes a tagged sub-source. The role taxonomy reuses Compressor 1.1's Inventory classes plus three additions Extract 2.0 needs for opinion / commerce / live-discovery prompts.

| Role | Definition | Default editability |
| --- | --- | --- |
| rule | Directive the runtime model must follow. | Low |
| refusal | Condition that blocks or redirects unsafe/invalid tasks. | Lowest |
| schema | Output fields, types, enums, JSON or tool-call examples. | Lowest |
| format | Output format instructions, ordering, headings, templates. | Low |
| example_primary | Main happy-path example. | Medium-low |
| example_edge | Refusal, error, boundary, non-obvious behavior example. | Low |
| example_shape | Duplicate example included mainly to show shape. | Medium |
| prose | Rationale or explanation. | High |
| filler | Politeness, hedging, transition phrases, repeated padding. | Highest |
| mode_binding (new) | Names which run-mode the prompt is in (e.g. Extract 2.0 argument/commerce/opinion). Behavior-bearing. | Low |
| locale_binding (new) | Names country, currency, store, language. Behavior-bearing in commerce / opinion. | Low |
| source_plan (new) | Names which live sources the prompt may fetch (Extract 2.0 E1). | Low |

## E2 — Atom extraction

One atom per rule, refusal trigger, schema field, example, and format directive. Atoms carry provenance (module name + line range) so the diff manifest can name exactly which source span maps to which rewritten span. Two new tags ride alongside Extract 2.0's known/inferred/speculative:

- **load_bearing** — referenced by ≥2 other atoms (e.g. a schema field used in three examples). Editing a load-bearing atom forces cascade edits, which the editor must enumerate before X2 runs.
- **dead** — defined but referenced nowhere. Candidates for deletion under compress and refactor, but never deleted silently under expand, retarget, or migrate.

## E3 — Cluster + connect

Cluster atoms by behavior axis, not by topic. Names must contain a verb or a comparative or a decision phrase.

| Axis | Cluster names look like | Why this axis |
| --- | --- | --- |
| Decision-rule | refuses-when-source-empty, requires-locale-on-priced-atom | These are the rules the runtime model executes. |
| Refusal | blocks-credentialed-advice, blocks-undated-prices | Refusal logic is the highest-tier lock. |
| Schema-shape | emits-M0-bundle, emits-claim-ledger-row | Schema-shape clusters bind to downstream parsers. |
| Style / tone | argues-not-surveys, strips-AI-worship-verbs | Style is editable but bound to operator preferences. |
| Dead-code | defines-but-never-references-X | Surfaced for deletion; never deleted silently. |

## E4 — Combat pass

For each cluster, decide one of three positions against the edit instruction: **applies** (the edit should change this cluster), **skips** (the cluster is out of scope), or **rewrites** (the cluster survives but in a new form). Reject balanced framing: every cluster takes a position. The combat paragraph is capped at 80 words.

## E5 — Hand-off to X0

Emit the inspection bundle: atoms, clusters with combat decisions, the lock map (input locks paired with the atoms that satisfy them), the dead-atom list, and a sufficiency score on three axes — edit_question, atom_pool, lock_coverage. If two of three are weak, refuse hand-off. Cap the bundle at 120 atoms (the editor handles up to 90 for compress, up to 120 for refactor and migrate where surface-mass is higher).

## The editor half (X0–X4)

Where Extract 2.0 stops and the rewrite begins.

### X0 — Intent classification

Confirm the op declared at E0 matches what the diff will actually do.

- **INPUT:** The inspection bundle from E5, the edit_intent from E0.
- **PROCESS:**
  1. For each cluster's combat decision, classify the implied op:
     - applies + atoms shrink → compress
     - applies + atoms added → expand or extend
     - applies + atoms move/rename → refactor
     - applies + audience or language changes → retarget
     - applies + downstream contract changes → migrate
     - applies on no atoms; only audit findings → lint
     - applies across two source prompts → merge
  2. Compare to the op declared at E0. If they disagree → return to E0 with a note.
  3. Lock the op. Downstream modules may not silently switch.
- **OUTPUT:** `op_lock: { op, secondary_ops[], parameters }`
- **FAILS WHEN:** Declared op is compress but the diff adds atoms. Declared op is refactor but the diff changes a schema field. Silent op-drift between E0 and X2.

### X1 — Edit plan

Atom-level plan. One row per atom touched.

- **INPUT:** Inspection bundle + op_lock.
- **PROCESS:** Produce a table of edits: keep, rewrite, delete, add, move. Every locked atom maps to keep or rewrite — never delete. Every load-bearing atom edit enumerates its cascade. Every dead atom is marked delete-candidate, not delete, unless op is compress or refactor.
- **OUTPUT:** `edit_plan[]: { atom_id, action, new_text?, cascade_ids[] }`
- **FAILS WHEN:** A locked atom is marked delete. A load-bearing edit has empty cascade. A dead atom is deleted under expand, extend, retarget, or migrate.

### X2 — Apply + diff

Produce the new prompt and a structured diff manifest.

- **INPUT:** Edit plan + original prompt.
- **PROCESS:**
  1. Apply edits in the order: deletes → rewrites → adds → moves. (Moves last so addresses stay valid.)
  2. Emit a diff manifest: every atom_id paired with before-text, after-text, action, provenance.
  3. The new prompt is re-parsed through Extract 2.0 E1–E2 to produce a post-edit manifest.
  4. Diff manifest + post-edit manifest go to X3.
- **OUTPUT:** new_prompt, diff, post_manifest.
- **FAILS WHEN:** Post-edit manifest cannot be parsed. An atom in the diff has no counterpart in either manifest. A locked atom's after-text is empty.

### X3 — Behavior-preservation pass

Compressor 1.1's audit, generalized to all ops, plus three new op-specific checks.

- **INPUT:** Diff + post_manifest + op_lock + the original input locks.
- **PROCESS:** Run the original Compressor 1.1 audit set (locks_mapped, negations_preserved, schema_fields_preserved, template_fields_declared, refusal_triggers_preserved, legend_present_if_symbols_used, unicode_integrity, example_order_preserved, format_instructions_preserved) plus:
  - **op_consistency** — the diff's actions match op_lock.
  - **cascade_resolved** — every load-bearing edit's cascade ids appear as edits.
  - **dead_only_deletion** — deletions under non-compress/refactor ops are limited to dead atoms.
  - **locale_carried** — under migrate or retarget, every priced or availability atom still carries currency + date + store (Extract 2.0 commerce rule).
  - **triangulation_carried** — under migrate or retarget, opinion consensus atoms still cite ≥2 independent threads.
- **OUTPUT:** `audit: { passed, checks{}, failures[], warnings[] }`
- **FAILS WHEN:** Any hard check is false. Op-drift detected. Cascade unresolved. Locale or triangulation dropped under a non-compress op.

### X4 — Ship or refuse

Ship on full pass. Refuse with structured reason on any hard failure.

- **INPUT:** Audit.
- **PROCESS:** If audit.passed is true and no warning is in the hard list → ship the new prompt plus the audit and diff. Otherwise emit a refusal that names blocking_items, suggests the minimum change that would unblock, and offers a downgrade (e.g. compress at ratio 0.55 instead of 0.30; migrate as refactor + retarget).
- **OUTPUT:**
  - Shipped: `{ new_prompt, diff, audit, inspection_bundle }`
  - Refused: `{ refused: true, reason, blocking_items[], suggestion, recommended_op }`
- **FAILS WHEN:** Ships a prompt whose audit has any false hard check. Returns a non-structured refusal.

## The diff manifest

Compressor 1.1 emitted a `dropped[]` list. Editing needs more. The diff manifest is symmetric: every change is named in both directions, so the audit can verify that no atom moved without a record.

```json
{
  "op": "refactor",
  "summary": { "kept": 47, "rewritten": 12, "deleted": 3, "added": 6, "moved": 2 },
  "edits": [
    {
      "atom_id": "E1.R3",
      "action": "rewrite",
      "role": "rule",
      "before": "Tag every URL as tier-secondary by default.",
      "after": "Tag URLs by primary/secondary/tertiary rule; never default.",
      "locked": true,
      "cascade_ids": ["E1.R7", "E2.R1"],
      "reason": "removes silent default; matches new locale rule"
    },
    {
      "atom_id": "E1.R12",
      "action": "delete",
      "role": "prose",
      "before": "Note that this is generally good practice.",
      "dead": true,
      "reason": "referenced nowhere; filler"
    }
  ]
}
```

## Op-aware behavior budgets

Compressor 1.1's task-aware compression budgets generalize: every op has a different tolerance for which spans it may touch.

| Op | Protect most | Edit most |
| --- | --- | --- |
| compress | Schema, refusal, negations, examples (esp. edge). | Filler, prose, repeated shape-only examples. |
| expand | Existing rules, existing examples, existing schema. | Gaps surfaced by lint: missing edge example, missing refusal, missing mode binding. |
| refactor | All atoms (no behavior change allowed); only addresses move. | Section boundaries, rule numbering, global lifting, template extraction. |
| retarget | Schema, refusal triggers, example order. | Audience surface (human vs ai_only), language, task_type-specific phrasing. |
| extend | Every existing atom unchanged unless the new capability requires a documented edit. | New mode bindings, new rules, new refusal triggers, new examples. |
| lint | Everything. Lint never edits. | Nothing. Output is the audit only. |
| migrate | Semantic contract (what the prompt promises downstream). | Surface format (XML ↔ Markdown ↔ JSON; OpenAI tools ↔ Claude tools ↔ MCP). |
| merge | Each source prompt's locked atoms. | Collisions resolved by declared precedence; duplicate rules collapsed. |

## Worked example — editing the Extract Prompt

The team has Extract Prompt 1.2 in production and the new Extract Prompt 2.0 in draft. Instead of replacing 1.2 wholesale, the operator asks the editor to extend 1.2 with the live-discovery, commerce, and opinion capabilities, preserving the existing argument-mode behavior. This is the exact workflow this skill exists for.

**Operator instruction:** Take Extract Prompt 1.2 and add the ability to fetch sources when none are provided, support commerce queries (store inventories, prices, locale binding) and opinion mining (forums, reviews with triangulation). Do not change argument-mode behavior. Keep the M0 hand-off contract.

### E0 — Intent framing

```json
{
  "op": "extend",
  "edit_question": "Can Extract 1.2 absorb live discovery, commerce, and opinion modes without changing argument-mode behavior or the M0 hand-off?",
  "anti_question": "Rewrite Extract 1.2 from scratch as Extract 2.0.",
  "audience": "human_readable",
  "language": "en",
  "task_type": "structured_output",
  "locks": [
    "argument-mode behavior unchanged",
    "M0 input bundle contract unchanged",
    "role taxonomy: evidence | instruction | reference | meta survives",
    "density floor and per-source atom_cap mechanics survive"
  ],
  "target": "global"
}
```

### E1–E2 — Manifest and atoms (excerpt)

Every module of Extract 1.2 becomes a sub-source; every rule becomes an atom. We surface only the load-bearing atoms here; the full manifest is ~80 atoms.

| Atom | Mod | Role | Text (≤25 words) | Load-bearing? |
| --- | --- | --- | --- | --- |
| E0.R1 | E0 | rule | State the subject in one sentence the user would actually use. | yes (referenced by E5) |
| E0.R2 | E0 | rule | Convert the subject into a disagreeable question. | yes (referenced by E4, E5) |
| E1.R2 | E1 | rule | Tag role first: evidence \| instruction \| reference \| meta. | yes — locked |
| E1.R5 | E1 | rule | Compute per-source atom_cap: full=60, half=30, quarter=15. | yes — locked |
| E2.R5 | E2 | rule | Tag each atom: known \| inferred \| speculative. | yes (referenced by E5) |
| E2.R7 | E2 | rule | Density floor: ≥1 atom per 5 min video / 1 page / 500 words. | yes — locked |
| E5.S1 | E5 | schema | M0 bundle: thesis seed, evidence pool, scope, suggested mode. | yes — locked |
| E5.R3 | E5 | rule | If two of three sufficiency axes are weak, refuse hand-off. | yes — locked |

### E3 — Clusters and combat decisions

| Cluster | Decision vs. edit instruction | Why |
| --- | --- | --- |
| frames-question-as-disagreement | applies (extend) | Generalizes to operational questions for commerce / opinion modes. |
| assembles-corpus-by-role-and-tier | applies (extend) | Add new roles: commerce, opinion. Existing four roles stay. |
| caps-atoms-by-weight | skips | Mechanic survives unchanged; locked. |
| enforces-density-floor | rewrites | Add floor rows for reviews/page (opinion) and catalog pages (commerce). |
| clusters-by-mechanism-not-topic | rewrites | Add decision-rule / substitution axes for commerce; recurrence axes for opinion. |
| emits-M0-bundle | skips | Contract is locked. Add envelope flags only (mode, locale). |
| refuses-handoff-on-two-weak-axes | skips | Locked. Sufficiency rules extend per mode but the hand-off refusal logic does not move. |

### E5 → X0 — Op confirmation

Six clusters applied or rewrote; two skipped. No cluster deleted atoms. Op-lock confirms extend with secondary op refactor (cluster names and role taxonomy widened). No silent compress or migrate.

### X1 — Edit plan (excerpt)

| Atom | Action | After-text (excerpt) | Cascade |
| --- | --- | --- | --- |
| E0.R2 | rewrite | Convert subject into a disagreeable question (argument) or an operational question (commerce / opinion). | E0.R3, E4.R1 |
| E0.R* | add | Pick mode: argument \| commerce \| opinion. Bind locale if commerce/opinion. | E1.R2, E2.R*, E5.R* |
| E1.R2 | rewrite | Tag role: evidence \| instruction \| reference \| meta \| commerce \| opinion. | E2.R1 |
| E1.R* | add | Tag freshness on tier-2 and tier-3; apply decay 24/60 months in commerce/opinion. | E2.R* |
| E2.R5 | rewrite | Same tags + commerce-atom and opinion-atom vocabularies; triangulation rule for opinion consensus. | E3.R*, E5.R* |
| E2.R7 | rewrite | Density floor extended: + 1 atom / 10 reviews (opinion), 1 / catalog page (commerce). | — |
| E5.S1 | keep | M0 bundle contract unchanged. Envelope gains optional mode, locale, opinion-heavy flags. | — |
| E5.R3 | keep | Hand-off refusal logic unchanged. | — |

### X3 — Behavior-preservation pass result

```json
{
  "passed": true,
  "op": "extend",
  "checks": {
    "locks_mapped": true,
    "negations_preserved": true,
    "schema_fields_preserved": true,
    "refusal_triggers_preserved": true,
    "example_order_preserved": true,
    "format_instructions_preserved": true,
    "unicode_integrity": true,
    "op_consistency": true,
    "cascade_resolved": true,
    "dead_only_deletion": true,
    "locale_carried": true,
    "triangulation_carried": true
  },
  "warnings": [
    "E3 cluster axis widened; downstream consumers of cluster names must accept new verbs"
  ],
  "summary": { "kept": 62, "rewritten": 11, "deleted": 0, "added": 14, "moved": 0 }
}
```

### X4 — Ship

Audit passes. Editor ships Extract 2.0 as the extended Extract 1.2. The diff manifest is the human-reviewable record: 0 deletions, 4 locks honored, M0 hand-off contract intact, behavior preserved on argument-mode subjects. The cat-toys essay and the Lidl-Bucharest commerce query work on the new prompt; every argument-mode subject that worked on 1.2 still works on 2.0.

## Stress-test suite (Editor extensions)

Compressor 1.1's nine tests survive. Five new tests cover the editing surface.

| Test | Input risk | Expected result |
| --- | --- | --- |
| Op-drift | User asks compress; clusters reveal the diff adds atoms. | Editor returns to E0, asks whether op should be extend. |
| Cascade gap | Load-bearing atom rewritten; one of three referencing atoms is not touched. | X3 fails cascade_resolved; rolls back. |
| Silent dead delete | Op is retarget; dead atom is deleted in the diff. | X3 fails dead_only_deletion; rolls back the deletion only. |
| Locale strip | Op is migrate; commerce atom's currency tag dropped in surface change. | X3 fails locale_carried; refuses ship. |
| Merge collision | Two source prompts both define refuse-when-source-empty with different consequences. | X1 forces declared precedence; otherwise X4 refuses with a structured collision report. |
| Re-parse failure | After X2, Extract 2.0 E1 cannot tag the new prompt's modules. | X3 fails; the rewrite was not semantically valid. |

## Operating discipline

**Do (green):**

- Run Extract 2.0 against the prompt before the user's edit instruction is applied. Lint first, edit second.
- Map every input lock to at least one atom in the inspection bundle. Unmappable locks are intake failures, not warnings.
- Confirm op_lock at X0. If the diff drifts from the declared op, return to E0 — do not silently re-label.
- Emit the diff manifest in both directions: every changed atom names its before and after.
- Refuse small. Recommend the minimum unblocking change rather than the maximum.

**Be careful (yellow):**

- A cluster that rewrites in three sibling modules looks like a refactor; if the rewrite changes a runtime decision, it is an extend.
- Dead atoms are not always filler. A defined-but-unreferenced refusal trigger may be a regression hint, not slack.
- Symbol substitution under retarget to ai_only must keep the legend, the negation discipline, and the example order.
- Merge collisions resolve by declared precedence; if precedence is not declared, refuse and ask.
- Migrating tool-call schemas is not a surface change; tool-name and parameter-name edits cascade into every example.

**Avoid (red):**

- Editing without an inspection bundle. Without Extract 2.0's pass, the editor is guessing.
- Letting compress add atoms because the rewrite reads better.
- Letting extend rewrite a locked rule because the new mode "wants it."
- Shipping a refactor whose post-edit Extract 2.0 manifest cannot be parsed.
- Returning a non-structured refusal. Refusal is a feature; prose is not.

## Stop conditions

Inherited from Compressor 1.1 and extended for the editor's surface.

**Hard stops:**

- Stop and refuse if the inspection-bundle sufficiency score has two of three weak axes.
- Stop and refuse if op_lock at X0 disagrees with the op declared at E0 and the operator does not reconcile.
- Stop and roll back if a negation is dropped or inverted in any op.
- Stop and roll back if a locked atom is marked delete or its after-text is empty.
- Stop and roll back if a load-bearing edit's cascade is incomplete.
- Stop and roll back if a dead-only-deletion check fails under a non-compress / non-refactor op.
- Stop and roll back if a migrated commerce or opinion atom loses currency, date, store, or triangulation.
- Do not retry blindly after audit failure. Inspect failures[], diff.edits[], and the inspection bundle first.

## Contract with the rest of the pack

Prompt Editor 1.0 has a defined edge with each sibling prompt:

| Sibling | Editor's relationship to it |
| --- | --- |
| Extract Prompt 2.0 | The diagnostic engine. Editor reuses E0–E5 unchanged; treats the prompt under edit as a single full-weight evidence source whose modules are sub-sources. Editor never re-implements extraction. |
| Idea Essay Prompt 3.4 | Out of scope. Editor edits prompts, not essays. If the operator asks to edit a rendered PDF, the editor refuses and points at the Idea Essay's M8/M9 rendering modules. |
| Prompt Compressor 1.1 | Absorbed. Compressor's C0–C10 modules survive as the editor's behavior-preservation pass for compress ops; their audit set is the floor for every op. Calling the compressor directly is now a thin wrapper around editor(op=compress). |

Decision rule (carried from Extract 2.0): if a step transforms one kind of artifact into another, it earns its own prompt. The editor transforms a prompt and an edit instruction into a new prompt plus a structured diff and audit. Inspection (corpus → atoms) lives in Extract 2.0. Rendering (essay → PDF) lives in Idea Essay 3.4. Editing lives here.

## Team checklist (delivery gate)

Before a prompt edit ships to production, the on-call engineer signs off on:

- Inspection bundle generated by Extract 2.0 attached to the change.
- op_lock matches the operator's declared intent; no silent op-drift between E0 and X2.
- Every input lock has a mapped atom in the diff manifest (status: present, lifted, symbolized, or rewritten).
- X3 audit passes with no false hard check. Warnings, if any, are reviewed and accepted in writing.
- Regression suite run: the prompt's previous worked example still produces the previous output shape under the post-edit prompt.
- Diff manifest reviewed by a second engineer for non-compress ops.
- Refusal path tested at least once during development (intentionally bad ratio, bad merge collision, or bad migrate contract).
