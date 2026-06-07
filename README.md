# AI-Agent-Skills
A personal library of reusable agent skills across GPT, Claude, Perplexity & Gemini

This repository is a curated collection of custom skills I've built in my journey of learning AI. The goal was to extend the capabilities 
of AI agents and automate research, structure workflows, and standardizing how I get 
high-quality, consistent outputs across platforms. Each skill is a self-contained module that 
an agent applies automatically whenever the relevant task comes up. There are crossover modules and the project is still work in progress.

---

## What Are Agent Skills?

Skills are reusable instruction sets that teach an AI agent how to handle a specific task once, 
so it remembers the workflow forever. Instead of re-explaining a process every time, you upload 
a skill and the agent applies the right format, steps, and quality checks automatically. The 
skills in this library are largely portable across Claude, Perplexity, GPT, and Gemini.

---

## Skill Library

### Research & Extraction

| Skill | Description |

| [Web Extractor](./web-extractor) | Pulls structured, evidence-backed information from websites, articles, and online sources into clean, usable outputs. Separates facts from inferences and ranks sources by reliability. |
| [Text-Video-Audio Web Extractor](./text-video-audio-web-extractor) | Extends extraction to video and audio sources, including YouTube transcripts and multimodal content, with source-integrity checks. |
| [Low Token Research](./low-token-research) | Produces concise, dense research outputs without wasting context. Built for fast synthesis, compressed summaries, and limited-context workflows. |
| [Skill Extractor](./skill-extractor) | Reads an existing conversation, document, or workflow and extracts the reusable skill logic hidden inside it — turns ad-hoc instructions into a structured, portable skill file. |

### Writing & Document Creation

| Skill | Description |

| [Idea Essay Prompt](./idea-essay-prompt) | Turns rough ideas, notes, or screenshots into structured essays and reports. Focuses on thesis development, section logic, source grounding, and counterarguments. |
| [Course Reading Prompt](./course-reading-prompt) | Converts academic readings into structured summaries, key terms, core arguments, and exam-ready study notes. |
| [Prompt Document Generator](./prompt-document-generator) | Converts rough workflow ideas into polished, reusable prompt documents with defined inputs, steps, output format, constraints, and quality checks. |

### Productivity & Workflow

| Skill | Description |

| [Job Search Tracker](./job-search-tracker) | Tracks job and internship applications as a structured pipeline — company, role, deadline, URL, resume version, visa constraints, value score, and next action. |
| [Quick Clarifier](./quick-clarifier) | Identifies the minimum clarifications needed before executing a task. Reduces friction without making incorrect assumptions. |
| [About Me Portfolio](./about-me-portfolio) | Stores personal context and generates consistent professional profile content — bios, LinkedIn summaries, GitHub descriptions, and resume sections. |
| [Skill Editor](./skill-editor) | Takes an existing skill file and applies targeted edits — removing contradictions, adding missing sections, stress-testing logic, and updating outputs without rewriting the whole skill. |

### Product & Personal

| Skill | Description |

| [Product Finder Workflow](./product-finder-workflow) | Finds products online from vague descriptions, project needs, or partial memories. Handles niche items, multi-product searches, bills of materials, and alternatives. Rankings include price, fit, availability, and seller trust. |
| [Meal Planner Workflow](./meal-planner-workflow) | Creates practical meal plans accounting for available ingredients, substitutions, shopping lists, flavor theory, and out-of-stock alternatives. Can be used with an inventory system in restaurants or at home.|

---

## How to Use

Each skill lives in its own folder containing the skill file and a short description.

**Perplexity:** Navigate to `perplexity.ai/computer/skills`, click **+ Create skill → Upload a 
skill**, and attach the `.md` or `.zip` file.

**Claude / GPT / Gemini:** Paste the skill content into the platform's custom instructions, 
system prompt, or project context.
