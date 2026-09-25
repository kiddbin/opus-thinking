# Opus Think

### A debugging framework for AI coding agents that refuses to guess.

> **20+ rounds of guesswork fixes failed. A strict evidence-first protocol closed the same bug in 2.**
>
> Later, at a different scale: **five blind fixes spread across multiple files — nothing changed.** After the investigation itself was repaired with seven single-variable probes (~30 minutes), the real fix took **three small edits. One shot.**

Opus Think is a skill file you drop into Claude Code, Cursor, Trae, CodeBuddy, Windsurf, or any agent that accepts a system prompt. It does not make your agent smarter. It makes your agent **measure before it moves.**

---

## The loop you are probably already in

Your agent is fast. Hand it a bug and it will run this loop without being asked:

**see the symptom → pattern-match → write code → declare it fixed.**

Look at what is missing. Not one step of that loop establishes *what actually happened*. Nothing was measured. A shape was recognized, and answered.

Here is what that costs:

| What you observe | What is actually going on |
|---|---|
| Round 3 becomes round 5 | Every round felt well-reasoned. That is the trap, not the excuse. |
| The symptom *mutates* instead of disappearing | "A fails" becomes "B fails". You are not converging — you are relocating. |
| Nothing is attributable | Four things changed at once, so you learned nothing about which one mattered. |
| The agent grows **more** confident | Its own earlier claims start serving as evidence. The most dangerous pseudo-evidence is your own past confidence. |

The answer is not "try harder". The mechanism behind *"I've fixed this three times and it's still broken"* is not attitude — it is economics.

---

## The idea that makes this framework different

> ### Your dashboard is not reality. It is your code's opinion about reality.

Every metric, status code, ledger row, and aggregate panel in your system was written by you. Writing it **drops raw error frames, merges fields that mean different things, and compresses three distinct causes into a single code.**

So when you debug *from* your own instrument, you are investigating a crime scene that has already been processed by the suspect.

That is why the loop above can run for rounds without anyone noticing it is broken. Each round has "evidence". Each round is well-reasoned. And each round is reasoning about your program's **interpretation** of the failure rather than the failure itself.

**v3.1 gives this a name — `derived observation` — its own evidence tier, and a rule: M8, Observation Completeness Before Repair.**

If three consecutive rounds of evidence all came from the same self-built instrument, your bottleneck is not the fix. It is the instrumentation. Stop repairing. Start measuring.

**The test, in one sentence:** *Can you say what this failure's raw message actually looks like?* If you cannot, you are still spinning inside the interpretation layer.

---

## Why "one more guess" is never free

> A guess that has to become **change code → build → ask the user to restart → verify on the real machine** is already too expensive — too expensive to abandon once it turns out to be wrong.

So it does not get abandoned. It stays in the code. That is the real mechanism behind *"the more I fix it, the worse it gets"* — not carelessness, not discipline, not attitude. **Cost.**

Opus Think's answer is to make guesses die cheaply: minute-scale, single-variable, automatically readable experiments designed **before** any code is touched.

---

## Core principles — the three axioms

| # | Axiom | Meaning |
|---|-------|---------|
| **P1** | **State Unknowability** | Unmeasured state does not exist for the observer. Reasoning about it is indistinguishable from fiction. |
| **P2** | **Change Attribution Conservation** | A single action's attribution clarity is inversely proportional to the number of independent dimensions it changes. Change N things → clarity of 1/N. |
| **P3** | **Cognitive Thermodynamics** | Without external energy — new evidence, rest, a change of perspective — debugging entropy increases monotonically. Repeating the same *kind* of guess accelerates it. |

Stuck? Ask three questions: *Am I reasoning about unmeasured state? How many things did I just change? When did I last inject new information?*

---

## What is inside

| Component | What it does |
|---|---|
| **3 axioms** (P1–P3) | The logical lower bound. Every rule below is derived from them, so you can rebuild the framework instead of memorizing it. |
| **8 mental models** (M1–M8) | Axioms mapped into practice: fault-space enumeration, evidence tiering, fingerprinting every change, separating fixes from refactors, attention economics, refusing to infer what you can simply ask, metacognition, and observation completeness. |
| **CRS scheduler** | Four modes — **Scout / Surgical / Tourniquet / Escalation** — deciding *which* discipline gets the power right now. Discipline is static; CRS is its runtime arbiter. |
| **5 disciplines + D0** | Evidence before code · one layer at a time · fingerprint every change · enumerate before acting · the **two-round stop rule** (enforced, not suggested). D0: after a tool's *first* failure, check existing records before retrying — a retry built on the same cognition produces no new information. |
| **6-layer diagnostic protocol** (L1–L6) | Disk bytes → loaded bytes → executed path → transport → service → **the upstream's own words**. The defect lives at the *lowest* layer that fails verification. |
| **12 anti-patterns** | Named failure modes: fix buffet, ghost verification, hidden refactoring, documentation avalanche, cache optimism, confidence inflation, self-reference as evidence, and more. |
| **7 adversarial scenarios** | Each names the conditions under which the framework's *default* behavior is wrong — and supplies the degradation path. |
| **Code-editing sub-protocol** | Three questions before any edit, an edit-method selection table, the multi-file ritual, cross-layer invariant scanning, and the failure signals that mean *stop and re-collect evidence*. |
| **Per-reply self-audit** | A checklist the agent runs on itself before answering, including a cognitive-entropy audit. |

---

## When to reach for it

- *"It doesn't work."* / *"Still failing."* / *"I tried that but…"*
- Mysterious failures with no clear cause
- **Repeated failed fixes (3+ attempts)**
- The symptom keeps changing shape instead of disappearing
- Integration failures between components
- Anything you will end up debugging from your own logs, dashboards, or status codes
- Designing systems where **verifiability matters more than cleverness**

---

## Quick start — the 80% solution

You do not need to read the whole framework. Three rules carry most of the weight:

1. **Evidence first, code second.** Get concrete, discriminating evidence before changing anything.
2. **One change at a time.** One file, one function, one variable. Verify, then proceed.
3. **Every round ships an observable checkpoint.** Tell the user: *"You should see X. If you see Y, send me Z."*

And when several rounds in a row fail: **suspect the completeness of your crime scene before inventing a fourth fix.**

---

## Files

| File | Language | Description |
|------|----------|-------------|
| [SKILL.md](./SKILL.md) | Chinese | Full framework |
| [opus-think-EN.md](./opus-think-EN.md) | English | Full framework |

## Installation

**Trae / Claude Code / CodeBuddy** — copy the `.md` file into your project's skills or rules directory.
**Cursor** — add it to `.cursor/rules/`, or reference it as a project rule.
**Any custom agent** — include the file in your system prompt or tool definitions.

## Metadata (for AI crawlers)

```yaml
name: opus-think
type: ai-skill, debugging-framework, prompt-engineering
version: 3.1.0
domain: debugging, reliability-engineering, ai-human-collaboration
language: zh, en
keywords: evidence-driven, debugging, root-cause-analysis, first-principles, ai-agent-skill, derived-observation
compatible_with: claude-code, cursor, trae, codebuddy, windsurf, copilot
```
