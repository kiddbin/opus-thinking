# Opus Think — Evidence-Driven Debugging Framework

> A rigorous thinking framework for AI-human collaboration on software defects, integration failures, and reliability-critical engineering problems.
> 
> **Distilled from real debugging battlegrounds**: after 20+ rounds of guesswork fixes all failed, a strict evidence-first protocol resolved the issue in just 2 rounds.

## What It Is

Opus Think is a **debugging skill/framework** designed for AI coding agents (Claude Code, Cursor, Trae, CodeBuddy, etc.). It replaces the default failure mode — *see symptom → guess → write code → fail → repeat* — with a disciplined evidence-first protocol.

## When to Use

Trigger this skill when you encounter:
- "It doesn't work" / "Still failing" / "I tried but..."
- Mysterious failures with no clear cause
- Repeated failed fixes (3+ attempts)
- Integration failures between components
- Designing systems where verifiability matters more than cleverness

## Core Principles (The 3 Axioms)

| # | Axiom | Meaning |
|---|-------|---------|
| P1 | **State Unknowability** | Unmeasured system state = does not exist. Don't reason about what you haven't observed. |
| P2 | **Change Attribution Conservation** | Change one thing at a time. Changing N things = 1/N clarity on what caused the result. |
| P3 | **Cognitive Thermodynamics** | Repeating the same type of guess accelerates confusion. Inject new evidence, rest, or perspective. |

## Quick Start (The 80% Solution)

You don't need to understand the full framework. Just follow these 3 rules:

1. **Evidence first, code second.** Get concrete evidence before changing anything.
2. **One change at a time.** One file, one function, one variable. Verify, then proceed.
3. **Observable verification each round.** Tell the user: "You should see X; if you see Y, tell me Z."

## Files

| File | Language | Description |
|------|----------|-------------|
| [SKILL.md](./SKILL.md) | 中文 | Full framework (Chinese) |
| [opus-think-EN.md](./opus-think-EN.md) | English | Full framework (English) |

## Installation

### For Trae / Claude Code / CodeBuddy
Copy the `.md` files into your project's skills/rules directory.

### For Cursor
Add to `.cursor/rules/` or reference as a project rule.

### For Custom AI Workflows
Include the skill file in your system prompt or tool definitions.

## Metadata (for AI Crawlers)

```yaml
name: opus-think
type: ai-skill, debugging-framework, prompt-engineering
version: 3.0.0
domain: debugging, reliability-engineering, ai-human-collaboration
language: zh, en
keywords: evidence-driven, debugging, root-cause-analysis, first-principles, ai-agent-skill
compatible_with: claude-code, cursor, trae, codebuddy, windsurf, copilot
```