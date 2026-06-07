---
name: prompt-compressor
description: "Compress long prompts and PDF prompt-skills into shorter, parse-safe, behavior-preserving versions without losing the behavior contract (inputs, outputs, refusal logic, schemas). Use when asked to shrink, compress, or token-reduce a prompt or skill at a target ratio. Runs a nine-module pipeline (C0–C8): intake, span inventory, filler removal, symbolic substitution, structural conversion, directive lifting, template extraction, structured audit, and refusal. Compression is not summarization: it removes redundancy, never information. Every output ships with an audit object; if the target ratio would break a locked rule, it returns a structured refusal instead. Based on LLMLingua research and Anthropic context-engineering guidance."
license: CC-BY-NC-4.0
metadata:
  author: Matei-Ionuț Șuță
  version: '1.0'
---

# Prompt Compressor 1.0 — Shrink Skill PDFs Without Breaking Them

A reusable skill that compresses existing prompts and PDF prompt-skills into shorter, parse-safe, behavior-preserving versions. It draws on LLMLingua research, Anthropic context-engineering guidance, and community token-saving patterns.

## When to Use This Skill

Use this skill when a user hands you a long prompt or a PDF skill and wants it made shorter while keeping the same behavior on the same inputs. It works by ranking every span by load-bearing weight, deleting filler, swapping verbose phrases for symbols and short keys, lifting repeated directives into a single header, extracting recurring output shapes into named templates, and refusing the job when no safe compression exists at the requested ratio.

Compression is not summarization. A summary loses information; a compressed prompt loses redundancy. The behavior contract — inputs, outputs, refusal logic, schemas — must survive byte-for-byte in meaning, even if its surface form changes.

## Inputs and outputs at a glance

| Input | What you supply | Output | What you get |
| --- | --- | --- | --- |
| Source | Prompt text or PDF skill | Target | Token reduction ratio you asked for |
| Ratio | e.g. 0.50 (cut to half) | Compressed | Shorter prompt with same behavior |
| Audience | human_readable or ai_only | Audit | Before/after tokens, dropped sections, parse-check |
| Locks | Rules that must not change | Refusal | Structured object if target unsafe |

## Why this exists

Token budgets are real. OpenAI documents that 1 token ≈ 4 characters ≈ ¾ of a word. A 10-page PDF skill can easily exceed 3,000 tokens once parsed, and most of that weight is prose explanation the model does not need at inference time. Microsoft LLMLingua shows automated pruning achieves up to 20× compression on RAG prompts with minimal performance loss. This skill is the manual, audited equivalent for your own prompt library.

## Module map

Nine modules, executed in order. Each declares INPUT, PROCESS, OUTPUT, and FAILS WHEN. The pattern mirrors Extract Prompt 1.2 so the two skills compose cleanly: Extract surfaces structured facts, Compressor squeezes the prompts that ask for those facts.

| # | Module | Job |
| --- | --- | --- |
| C0 | Intake | Receive source, ratio, audience, locks. Validate. |
| C1 | Inventory | Classify every span: rule, schema, example, prose, filler. |
| C2 | Filler removal | Drop politeness, hedging, articles where unambiguous. |
| C3 | Symbolic substitution | Swap common phrases for symbols and short keys. |
| C4 | Structural conversion | Prose → bullets → key:value. Keep one canonical example. |
| C5 | Directive lifting | De-duplicate repeated rules into a single header. |
| C6 | Template extraction | Recurring output shapes → named template referenced inline. |
| C7 | Audit | Before/after token count, dropped log, parse-safety check. |
| C8 | Refusal | Structured object if target ratio would break a locked rule. |

### Hard rule

- Behavior preservation beats ratio. If C8 fires, ship the refusal — never deliver a prompt that quietly drops a locked rule.
- Every output must include the C7 audit object. No audit, no ship.

## C0 — Intake

Receive the job and confirm it is well-formed before any compression happens.

- **INPUT:** source (str|pdf), ratio (0.10–0.90), audience (human_readable|ai_only), locks (list of strings).
- **PROCESS:** Parse PDF if needed. Tokenize source. Confirm ratio is achievable in principle (skill should not request < 0.10).
- **OUTPUT:** intake object: `{tokens_in, ratio, audience, locks, source_kind}`.
- **FAILS WHEN:** ratio outside 0.10–0.90; locks list references text not present in source; source is binary non-PDF.

### Audience matters

`human_readable` keeps the prompt skim-able and editable by future-you. `ai_only` allows aggressive symbolic substitution — math operators, abbreviated keys, no narrative connective tissue. Outputs use math operators like → (leads to), ⇒ (implies), ⇔ (iff), ∈ (in), ≥, ≤ as semantic shortcuts plus 1–2 letter keys. Use `ai_only` only when the prompt will never be edited by a human again.

### Intake example

```json
{
  "source_kind": "pdf",
  "tokens_in": 3142,
  "ratio": 0.50,
  "audience": "human_readable",
  "locks": [
    "refuse if source is empty",
    "schema: {topic, findings, sources}",
    "never invent facts"
  ]
}
```

If any lock string is missing from the source, intake fails fast. This is cheap insurance against the most common bug in prompt compression: someone declared a lock that was already paraphrased away in an earlier edit.

## C1 — Inventory

Before deleting anything, classify every span. You cannot compress what you have not categorized.

| Class | Definition | Compressibility |
| --- | --- | --- |
| Rule | A directive the model must follow (must, never, refuse if) | Low — protect |
| Schema | Output shape: JSON keys, field types, enums | Low — protect |
| Example | Concrete input→output demonstration | Medium — keep one canonical |
| Prose | Explanation of why a rule exists | High — drop or shorten |
| Filler | Politeness, hedging, repetition, transitional sentences | Maximum — delete |

This mirrors Anthropic context-engineering guidance: "minimal precise instructions, say less mean more," and "curate diverse canonical examples, not exhaustive edge-case lists." Inventory is the audit trail that lets C7 prove what was kept and what was cut.

**Inventory output shape:**

- spans: list of `{id, class, tokens, locked, text_preview}`
- `locked = true` if the span is referenced in intake.locks OR is a schema field.
- Class label is the contract: later modules may only touch a span by its class.

## C2 — Filler removal

Apply a fixed deletion list. No model judgment, no creativity. The same input always yields the same output.

| Pattern | Action | Example before → after |
| --- | --- | --- |
| Politeness | Delete | "Please carefully consider" → ∅ |
| Hedging | Delete | "It may sometimes be the case that" → ∅ |
| Filler adverbs | Delete | "very", "really", "quite", "simply" → ∅ |
| Articles (ai_only) | Delete when unambiguous | "the user provides the prompt" → "user provides prompt" |
| Repeated phrasing | First instance only | Three "make sure to" lines → one |

A widely-shared coding-agent experiment ("caveman talk") reports ~38% token reduction on coding tasks by dropping articles, fillers, and politeness alone. The trade is legibility: `human_readable` mode keeps articles; `ai_only` drops them.

**Never delete:**

- Anything inside a locked span (C1).
- Negations: "not", "never", "must not" — flipping these breaks behavior.
- Numeric qualifiers: "at least 3", "exactly one", "up to N".
- Refusal triggers: "refuse if", "unsafe when", "reject when".

## C3 — Symbolic substitution

Replace verbose phrases with single-character symbols and short keys. Each substitution must be declared once at the top of the compressed prompt so the model sees the legend before the usage.

### Standard substitution table

| Phrase | Sym | Phrase | Sym |
| --- | --- | --- | --- |
| leads to / produces | → | if and only if | ⇔ |
| greater than or equal | ≥ | implies | ⇒ |
| less than or equal | ≤ | for each | ∀ |
| element of / is a | ∈ | there exists | ∃ |
| subset of / part of | ⊆ | not / negation | ¬ |
| union of / combine | ∪ | and | ∧ |
| approximately equal | ≈ | or | ∨ |

### Short-key conventions

Used in `ai_only` mode to abbreviate field names. Always declare the legend before first use.

```
LEGEND
  s = source
  q = query
  k = key
  v = value
  R = rule (must follow)
  S = schema (output shape)
  X = example (canonical)
  F = fail trigger (refuse if)
BODY
  R1: s ∈ {pdf, str}
  R2: ¬empty(s) ⇒ proceed
  F1: empty(s) ⇒ return refusal
```

## C4 — Structural conversion

Convert prose into the smallest structural form that preserves meaning. Three levels, applied progressively until the ratio target is hit.

| Level | Transformation | Token savings |
| --- | --- | --- |
| L1 | Prose paragraphs → bullets (one idea per line) | 10–25% |
| L2 | Bullets → key:value lines (drop verb phrases) | 20–40% |
| L3 | Examples: many → one canonical + "see X1 for shape" | 30–60% |

**Before and after:**

```
BEFORE (prose, 41 tokens)
When the user provides a source document, you should first
carefully read through the entire document and then identify
the key facts that are relevant to the user's question.

AFTER L2 (key:value, 16 tokens — 61% reduction)
R1: read(source) → facts
R2: facts ⊆ relevant_to(question)
```

C4 is the structured-prompting leg of the standard compression taxonomy (semantic summarization, structured/JSON prompting, relevance filtering, instruction referencing, template abstraction).

## C5 — Directive lifting

Repeated rules cost tokens every time they appear. Lift them once into a GLOBAL header and reference by ID afterwards. This is the "instruction referencing" technique applied to your own prompt.

```
BEFORE (rule repeated 3 times, ~60 tokens)
In Module A: always cite the source URL.
In Module B: cite the source URL for every claim.
In Module C: each claim must include a source URL citation.

AFTER (one GLOBAL line, ~12 tokens)
GLOBAL G1: every claim ⇒ source_url
Module A: …obey G1
Module B: …obey G1
Module C: …obey G1
```

**De-duplication checklist:**

- Identical sentences across modules → lift to GLOBAL.
- Near-identical sentences (>80% overlap) → unify wording, then lift.
- Module-specific rules → keep local, do not over-lift.
- If GLOBAL exceeds ~15 lines, you are over-lifting; some rules belong local.

## C6 — Template extraction

Recurring output shapes — same JSON keys repeated for each item, same Markdown skeleton repeated per section — should be declared once as a named template and referenced.

```
DECLARE (once)
TPL-01 = {id, claim, source_url, confidence ∈ [0,1]}

USE (inline, many times)
For each finding emit TPL-01.
For each counter-claim emit TPL-01 with negated=true.
```

This is the explicit form of the "template abstraction" technique. Combined with C5 directive lifting, it converts what looks like a long prompt with many examples into a short prompt with a legend, two named templates, and a handful of typed references. The behavior is identical; the token count is a fraction. The LongLLMLingua paper reports a 17.1% performance improvement at 4× compression when the structure is made explicit this way.

**Templates compose:** Templates can reference other templates. This is how a 30-page source skill collapses into a 6-page compressed skill without losing structural fidelity.

```
DECLARE
TPL-01 = {id, claim, source_url, confidence ∈ [0,1]}
TPL-02 = {finding: TPL-01, counter_findings: [TPL-01]}
TPL-03 = {topic, evidence: [TPL-02], conclusion}

USE
Emit one TPL-03 per topic.
```

## C7 — Audit

Every shipment includes a structured audit object. No audit, no ship.

```json
{
  "tokens_in": 3142,
  "tokens_out": 1488,
  "ratio": 0.474,
  "target": 0.50,
  "hit_target": true,
  "dropped": [
    {"span_id": 17, "class": "prose", "reason": "redundant rationale"},
    {"span_id": 22, "class": "filler", "reason": "politeness"},
    {"span_id": 31, "class": "example", "reason": "non-canonical duplicate"}
  ],
  "lifted_globals": ["G1: every claim ⇒ source_url"],
  "templates": ["TPL-01: finding shape"],
  "parse_check": "PASS — JSON valid, all locks present, no negations flipped",
  "warnings": []
}
```

**Parse-safety checks:**

| Check | Pass condition |
| --- | --- |
| Locks present | Every string in intake.locks appears verbatim in output. |
| Negations intact | Count of "not / never / must not / refuse if" unchanged. |
| Schema fields | All declared keys still appear in the schema block. |
| Refusal triggers | Every "refuse if" / "reject when" clause still present. |
| Legend coverage | Every symbol used in body is declared in legend. |

## C8 — Refusal

When the requested ratio would force deletion of a locked span, do not negotiate. Return a structured refusal so the caller knows exactly what blocked the job.

```json
{
  "refused": true,
  "reason": "target_ratio_breaks_lock",
  "target": 0.30,
  "max_safe_ratio": 0.55,
  "blocking_locks": [
    "refusal trigger: refuse if source is empty",
    "schema field: confidence ∈ [0,1]"
  ],
  "suggestion": "Retry with ratio ≥ 0.55, or remove locks if no longer needed."
}
```

**Refusal is a feature:**

- A silent over-compression that drops a rule is worse than no compression at all.
- The caller can either relax the ratio or relax the locks. Either is fine; both must be explicit.
- Always include max_safe_ratio so the caller has a concrete next step.

## Worked example — compressing a prompt fragment

A 7-line fragment from a hypothetical research-summary skill, followed by its compressed form. The audit object shows what survived.

```
Source (96 tokens)
You are a research assistant. When the user provides a topic,
please carefully read through the relevant documents and then
summarize the key findings. Always make sure to cite your
sources with URLs. Never invent facts. If the source is empty,
you must refuse the request. Output should be in JSON format
with fields: topic, findings (a list), and sources (a list of
URLs).

Compressed (38 tokens, ratio 0.40)
LEGEND  s=source  R=rule  F=fail  S=schema
GLOBAL  G1: every claim ⇒ source_url   G2: ¬invent
R1: s ⇒ summarize(findings)
F1: empty(s) ⇒ refuse
S:  {topic, findings:[], sources:[url]}
```

```json
{
  "tokens_in": 96, "tokens_out": 38, "ratio": 0.396,
  "target": 0.40, "hit_target": true,
  "dropped": [
    {"class": "filler", "reason": "'please carefully', 'make sure to'"},
    {"class": "prose", "reason": "role narration"}
  ],
  "lifted_globals": ["G1: cite", "G2: no invention"],
  "parse_check": "PASS — refusal trigger preserved, schema intact"
}
```

## How to call this skill

Treat it like any module-based prompt: give it a clear job object, then read the output and audit.

```
INPUT
  source:   <paste prompt or attach PDF>
  ratio:    0.50
  audience: human_readable
  locks: [
    "refuse if source is empty",
    "schema: {topic, findings, sources}"
  ]

EXPECTED OUTPUT
  compressed_prompt: <string>
  audit: <C7 object>
  refusal: null | <C8 object>
```

### Pairs well with

| Companion skill | Use together when |
| --- | --- |
| Extract Prompt 1.2 | You need to compress a long source document AND extract facts from it. |
| Source-Truth Pack 1.1 | Compressed prompt must still emit a full evidence ledger. |
| Skill Engineering | You are about to ship a new skill and want a tight v1 instead of a verbose v0. |
| LLM Control Primitives | Compressed prompt uses forced tokens, banned tokens, or cached headers. |

## Stop conditions

- If audit shows `hit_target=false`, do not retry blindly — inspect `dropped[]` and locks first.
- If `parse_check` reports a flipped negation, treat as a hard fail and roll back.
- If you cannot hit the target without breaking a lock, return the C8 refusal and let the caller decide.

Ship the audit, refuse when locked, never quietly drop a rule.

## Sources

Primary research backing each technique:

- LLMLingua (Microsoft, GitHub) — small LM identifies and removes non-essential tokens; up to 20× compression with minimal loss. [github.com/microsoft/LLMLingua](https://github.com/microsoft/LLMLingua)
- LongLLMLingua (arXiv 2310.06839) — question-aware coarse-to-fine compression; 17.1% performance gain at 4× compression. [arxiv.org/abs/2310.06839](https://arxiv.org/abs/2310.06839)
- Anthropic — effective context engineering — minimal precise instructions; canonical examples over exhaustive lists; compaction patterns. [anthropic.com/engineering](https://www.anthropic.com/engineering)
- MachineLearningMastery — prompt compression techniques — semantic summarization, structured prompting, relevance filtering, instruction referencing, template abstraction. [machinelearningmastery.com](https://machinelearningmastery.com/)
- Towards Data Science — RAG cost reduction via prompt compression — entropy-based pruning; low-perplexity tokens are removable. [towardsdatascience.com](https://towardsdatascience.com/)
- OpenAI — tokens and how to count them — 1 token ≈ 4 chars ≈ ¾ word; budget math. [help.openai.com](https://help.openai.com/)
- Microsoft LLMLingua-2 (arXiv 2403.12968) — BERT-based, task-agnostic, 3–6× faster than v1. [arxiv.org/abs/2403.12968](https://arxiv.org/abs/2403.12968)

**Reading order:** If you read only three of these, start with the Anthropic context-engineering essay for the philosophy, then MachineLearningMastery for the five concrete techniques, then the LLMLingua repository to see what fully-automated compression looks like at scale.
