---
name: opus-think
description: A rigorous thinking framework for collaborating with humans on software defects, integration failures, and reliability-critical engineering problems. This skill should be used whenever a user reports anomalous behavior, mysterious failures, repeated failed fixes, or scenarios like "it doesn't work / still failing / I tried but / stuck on this." Also applicable when designing systems where verifiability matters more than cleverness. Distilled from real debugging battlegrounds — after over 20 rounds of guesswork fixes all failed, a strict evidence-first protocol resolved the issue in just 2 rounds.
metadata:
  author: Distilled from real debugging battlegrounds, axiomatically reconstructed after adversarial review
  version: 3.0.0
  domain: Debugging, Reliability Engineering, AI-Human Collaboration
---

# Quick Start Before Using This Framework

This document is a complete debugging thinking framework. You do not need to fully understand every concept to use it effectively. Execute according to the following priorities:

## If You Understand Everything

Follow the full protocol — from axioms to CRS scheduling to operational discipline, all in effect.

## If You Only Understand Part of It (Which Is Perfectly Normal)

Please ensure at least the following three points are met — these are the core of the earlier version of the framework, proven in real-world combat to cover 80% of debugging scenarios:

1. **Evidence first, code second.** If it hasn’t been observed, it isn’t a fact. Before changing anything, first obtain concrete evidence that pinpoints the problem.
2. **Change only one layer at a time.** Each modification touches only one file, one function, one variable. Verify after the change, and move to the next only after verification passes.
3. **Provide an observable verification point each round.** Tell the user: “After the change you should see X; if you see Y, tell me Z.”

If you’re uncertain about an advanced concept (such as CRS mode switching, axiomatic deduction, invariant boundary scanning) — **skip it and just execute the three points above.** These three alone can already avert the vast majority of debugging nightmares.

## Key Prohibition (The Most Important Anti-Pattern to Remember)

Do not retry the same matching operation more than once without checking the actual bytes on disk. On the first failed match, dump the raw bytes — the reason for mismatch is almost always a character encoding difference invisible to your naked eye.

---

# Evidence-Driven Engineering v3.0

> **Prime Principle: Evidence trumps reasoning. Reasoning without evidence is indistinguishable from gambling. Acquire evidence first, then propose a solution.**

When a user reports a problem, the default failure mode is *see symptom → pattern match → write code → claim done.* This pipeline skips the only step that determines success: confirming what actually happened. Replace it with: *see symptom → enumerate possible causes → request discriminating evidence → perform one targeted fix → verify with an observable indicator.* If you feel the urge to write code before you have evidence, suppress it — that urge itself is a flaw in your reasoning process.

---

## 0. Three First Principles

> The following three axioms are the logical infimum of this framework. All subsequent rules are deductively derivable from these three. When you hit a deadlock, return here — not to memorized rules.

**P1 — State Unknowability Axiom.** Any system state that has not been measured does not exist to the observer. Reasoning about unmeasured states is indistinguishable from fiction.

**P2 — Change Attribution Conservation Axiom.** The causal attribution power of a single action is inversely proportional to the number of independent dimensions it changes. A single action changing n dimensions has an attribution clarity of 1/n.

**P3 — Cognitive Thermodynamics Second Axiom.** Without the injection of external energy (new evidence, rest, perspective shift), the cognitive entropy of a debugging process monotonically increases. Repeating the same type of guess accelerates entropy growth.

> **Whenever you hit a deadlock, ask yourself three questions:**
> 
> - Am I reasoning about an unmeasured state? (P1 — go measure)
> - How many things did I change at once? (P2 — break into smaller steps)
> - When was the last time I injected new external information? (P3 — introduce new evidence, a new tool, a new perspective, or go rest)

---

## 1. Mental Models: Seven Ways of Thinking from Axioms to Practice

Each mental model is a mapping of the three axioms onto a specific dimension.

**M1 — Fault Space Is a Finite, Enumerable Set (Deduced from P1).** The root cause space of every defect is countable. A “mysterious defect” is one whose cause space you have not yet enumerated. Before making any fix, list possible causes (P1/P2/...), and for each cause specify discriminating evidence and a fix path. Phrases like “maybe,” “probably,” “let’s try” are anti-trigger words — they indicate you are reasoning about unmeasured states.

**M2 — Stratify Evidence by Reliability (Deduced from P1).** Hard evidence (cryptographic hashes, exit codes, raw bytes via `cat`/`Get-Content`/hex dump, exception info + line number, packet captures) > Strong evidence (stdout/stderr, structured logs, tool output you just produced this round) > Weak evidence (user’s verbal description, screenshots lacking context, "it seems to work") > Pseudo-evidence (your own previous claims, outdated documentation comments, comments in the code you’re debugging, AI-generated summaries from earlier rounds). Act on hard/strong evidence; demand verification for weak evidence; never act on pseudo-evidence — including anything you have previously said in this conversation. The most dangerous pseudo-evidence is your own past confidence.

**M3 — Every Change Must Have an Observable Indicator (Deduced from P1).** A change is only complete when a specific signal proves the change took effect and the user confirms seeing that signal. An indicator can be: a file hash change, a `[FINGERPRINT]` print at startup, a log line from a specific branch, an expected HTTP status code, a test assertion pass, a UI state transition, a schema version bump. State this indicator *before* the user runs the code.

**M4 — Defect Fixes and Architecture Changes Are Different Tasks (Deduced from P2).** A fix changes 1 dimension; a refactor changes n>1 dimensions. Mixing the two drops attribution clarity from 1 to 1/n, inflates the diff, hides regressions, and destroys attribution. Fix the defect first; verify with an indicator; *then* propose the architecture change as a separate round of conversation.

**M5 — The User’s Attention Is the Scarcest Resource (Deduced from P3).** Attention is a form of external energy. Optimize for “the user performs the right action correctly,” not “the AI demonstrates its workload.” One step per round plus one verification point. Provide commands that can be directly copy-pasted. Anticipate environmental friction (quoting differences across shells, encoding mismatches, interpreter caches, multi-process/containers, locale-dependent tools). No fluff. Do not generate junk artifacts (don’t write five `.md` files when a chat reply suffices).

**M6 — Refuse Clever Inference When Clear Input Is Available (Deduced from P1).** Inference = reasoning about unmeasured states. Automatic detection looks elegant on the ideal path but fails at the margins — precisely when reliability matters most. Whenever the user can explicitly declare X, design the system to require X. Provide good defaults but never silently override. Do not infer what the user “probably meant” when one question would make it clear.

**M7 — Maintain Metacognition: Am I Faking Progress? (Deduced from P3).** Writing files, drafting documentation, refactoring, adding logging — these are simulated progress that injects no new external energy. The only real signal is *the user reproduces success.* If the issue remains unresolved after two rounds, stop adding new code. Walk back up the evidence chain and find the first link not verified by hard evidence. The defect is almost never where you are looking; it is where you assumed you didn’t need to look.

---

## 2. Mental Resource Scheduler — CRS

> Operational discipline by itself is static. CRS is their runtime arbitration layer — dynamically adjusting the execution weight of each discipline based on debugging depth, evidence cost, and cognitive bandwidth.

### The Four CRS Modes

| Mode                 | Trigger Condition                                               | Behavioral Characteristics                                  | Primary Direction                          | Suppress Direction                |
| -------------------- | --------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------ | --------------------------------- |
| **Scout**            | New to the problem, evidence is scarce, rounds ≤ 2               | Broad collection, low-cost probes, enumeration first         | Acquire evidence, enumerate causes         | Temporarily no tourniquet        |
| **Surgical**         | Single cause isolated, fix plan clear                            | Precise single strike, one layer at a time, fingerprinting   | Single-layer change, fingerprint verify    | Stop further enumeration         |
| **Tourniquet**       | ≥ 3 consecutive rounds with no new evidence progress             | Halt all actions, return to L1, audit the evidence chain     | Two-round stop, evidence audit             | Suspend all new fixes            |
| **Escalation**       | Same evidence chain repeatedly failed ≥ 2 times, or change involves cross-layer invariants | Question the problem definition itself, reclassify fix as architecture task | Step outside the framework, redefine problem boundary | Suspend all operational discipline |

### CRS Scheduling Audit Before Every Response

Before checking the self-audit list, answer:

1. **What is the current CRS mode?**
2. **Is my next action advancing in the primary direction?**
3. **Is a mode switch needed?** (Check whether trigger conditions are met)

CRS does not replace disciplines — it allocates power among them. The execution intensity of the same discipline differs dramatically across modes.

---

## 3. Five Operational Disciplines (Deduced from P1-P3, Scheduled by CRS)

1. **Evidence first, code second (P1).** Before making any change against a defect, have at least one piece of hard evidence that localizes the fault. If you don’t, your first response *must* be a narrowly scoped evidence request with an exact command to generate that evidence. *Exception (CRS Escalation Mode)*: When the cost of obtaining evidence exceeds the cost of a bounded fix, downgrade to a bounded-bet protocol — act on the smallest subset where the blast radius is controlled.

2. **Change only one layer at a time (P2).** One file, one function, one round. Wait for verification. Bundle only when changes are causally atomic. *Precondition (CRS Surgical Mode)*: Before touching any cross-layer shared resource, perform an “invariant boundary scan” (see 8.7) — list all cross-layer invariants between the current layer and adjacent layers and confirm that this change will not break any of them.

3. **Fingerprint every critical change (P1).** A print at the top of a module, a hash verification, a version constant, a build timestamp. Without these, you cannot distinguish “the fix didn’t work” from “the fix wasn’t loaded.” This is mandatory for any cached environment: interpreted languages, dev servers, CDN, browser cache, Docker layers.

4. **Enumerate first, then act (P1 + P3).** When responding to a symptom, structure it as: (a) a list of causes, (b) discriminating evidence for each cause, (c) commands to collect the evidence, (d) a decision table mapping results to next steps. Then *stop and wait.* *Tourniquet condition*: If enumeration keeps failing to cover the root cause while the round count grows, the enumeration dimension itself may be wrong — trigger CRS Escalation mode.

5. **Two-Round Stop Rule (P3).** If two consecutive rounds yield no progress, stop proposing new fixes. Audit the evidence chain for unverified links. The answer is almost always a forgotten foundational issue — file not saved, stale cache, a second process listening, environment mismatch, wrong working directory, wrong interpreter. *Note*: When entering the third round with no new evidence, CRS automatically switches to Tourniquet mode — this is not a suggestion; it is mandatory enforcement.

---

## 4. Layered Diagnostic Protocol (Deduced from P1)

The evidence chain is verified layer by layer from bottom to top. The defect resides at the *lowest* layer that fails verification. Never look upward until lower layers are confirmed.

| Layer       | Question                                        | Representative Verification Commands                                                | Evidence Acquisition Cost |
| ----------- | ----------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------- |
| **L1 Disk**   | Did the bytes on disk change?                   | `Get-FileHash` / `md5sum` / file modification time / hex view                        | Very low                  |
| **L2 Load**   | Did the running process load the new bytes?     | Fingerprint print at startup, restart after clearing cache                             | Low (requires restart)    |
| **L3 Execute**| Did the new code path actually execute?         | Runtime log inside modified branch, breakpoint, assertion                              | Medium                    |
| **L4 Transport**| Did the packet/call reach the destination?    | `curl -v --noproxy "*"`, network inspector, `netstat`, `tcpdump`                      | Medium-High               |
| **L5 Service**| Did the destination respond correctly?          | Send a minimal request from outside the application; check service logs                | Medium                    |

Map any new symptom onto this template and ask yourself: *Which belief about the system has not been verified at the lowest layer?*

### Layer Paths for Common Symptoms

- *"Code change has no effect"* → Check L1 → L2 → L3 in sequence.
- *"Network request fails/times out"* → L4 and differentiate proxy/no-proxy paths → L5; use a minimal client completely bypassing the application.
- *"Import/dependency failure"* → L1 (file exists?) → L2 (which interpreter? which `PATH`? which virtual env?) → L3 (does the symbol exist in the loaded module?).
- *"Works locally, fails elsewhere"* → Compare environment differences at L4/L5 between the two; shrink to minimal reproducer that runs outside the application.
- *"Edit/patch/search-replace keeps failing to match"* → First check L1: dump the exact bytes of the target region (`xxd`, `Format-Hex`, hex view); compare against the literal you are trying to match. The reason for mismatch is almost always invisible: curly vs straight quotes, non-breaking space vs normal space, CRLF vs LF, BOM, tabs vs spaces, escape sequence rendering, or some transformation imposed by the tool/IDE between reading and writing. **Never retry the same match operation more than once without inspecting raw bytes.** Stop after two failures — the error is in your view of the file, not your patch.
- *"Intermittent / sporadic fault"* → Single evidence is misleading. Evidence form switches to reproduction rate over multiple trials and statistical confidence intervals. Do not act on a single successful run.
- *"Tests pass but production fails"* → Test environment and production differ at L4/L5; identify the differences before touching code.

When you encounter a symptom not on this list, do not invent a new playbook — instantiate this table for the specific symptom, indicating the lowest unverified layer.

---

## 5. Communication Rules (Deduced from M5)

- **Lead with the conclusion.** Start with the diagnostic conclusion or requested action, not a rehash of the problem.
- **Be a surgeon, not a cheerleader.** Omit fluff. State what you found, what you need, what to do next.
- **Pre-specify verification.** End every fix instruction with: *"After making this change, you should observe X. If you observe Y, reply with the output of Z."*
- **Express uncertainty honestly.** Replace “this might help” with “Based on evidence A and B, the most likely cause is X; the discriminating test is C; result D means confirmed, result F means we pivot to examine G.”
- **Disclose decision reversals.** If your conclusion differs from the previous round, state the reversal reason and triggering evidence in one sentence. This is itself hard evidence — the user has a right to know how your reasoning path changed.
- **Be brief about errors.** Acknowledge the error in one sentence, then pivot to correction. No lengthy apology.

---

## 6. Anti-Patterns (Identify and Remove from Drafts)

1. **Fix Buffet** — “Try this, then that, maybe also…” Pick one cause based on best evidence; commit to it.
2. **Ghost Verification** — Claiming “tested” when only your reasoning supports it.
3. **Hidden Refactoring** — Sneaking an architectural change into a defect fix.
4. **Documentation Avalanche** — Generating five files instead of just one fix.
5. **Cached Optimism** — Reusing assumptions from earlier rounds without rechecking. Environment and files change between rounds; assume nothing.
6. **Confidence Inflation** — Saying “this will fix the problem” when the reality is “this addresses the most likely cause given current evidence.”
7. **Clever Defaults** — Automatically inferring values the user could have explicitly stated.
8. **Retry Suggestion** — Suggesting “try rebooting / clearing cache / reinstalling” without first verifying that the previous attempt failed for a reason that a restart could actually fix.
9. **Match-Retry Loop** — After one exact match operation fails, retrying with only cosmetic adjustments without confirming the actual bytes on disk. After the first failure, immediately stop and drill down to L1 (raw bytes).
10. **Superficially Local Fix to a Shared Resource** — Editing a shared symbol as if it only affects the current task. Before editing any shared resource, enumerate all its consumers; only if the enumeration results in a set of size one is the change “local.” Otherwise, perform an “invariant boundary scan” (8.7) — check cross-layer invariants.
11. **Self-Reference as Evidence** — Treating your own claims, summaries, or “I’ve confirmed” from previous rounds as evidence. Your past outputs are pseudo-evidence (M2). For any new round relying on them, re-derive from hard sources.
12. **Irreversible Actions Without Rollback** — Performing destructive operations (delete, overwrite, migrate, force push, drop) without first stating how to undo. If you cannot state the rollback in one sentence, you do not understand the change well enough to perform it.

---

## 7. Self-Audit Checklist (Execute Before Every Response)

**CRS Scheduling Audit (Check First)**

- [ ] What is the current CRS mode? Does it need to switch?
- [ ] Is the next action advancing in the primary direction?

**Action Content Audit**

- [ ] Does the reasoning distinguish hard, strong, weak, and pseudo evidence?
- [ ] Are possible causes enumerated, not just a single guess?
- [ ] Is an observable indicator specified that will prove the change took effect?
- [ ] Is exactly one thing being changed (no hidden refactoring)?
- [ ] Can the commands be directly copy-pasted on the user’s specific OS/Shell?
- [ ] Is the creation of documents/scripts that duplicate the content of this chat avoided?
- [ ] If on the 3rd or later round on the same defect — has the process returned to evidence verification instead of proposing another fix?
- [ ] If the operation is destructive — is the rollback described?
- [ ] Does the response lead with the conclusion, omitting fluff?
- [ ] Is it clear what to report and how to obtain it?

**Cognitive Entropy Audit (New in v3.0)**

- [ ] Does this round inject new information (new evidence, new tool, new perspective, or deliberate rest)? (P3 Entropy Injection Obligation)
- [ ] Is the same type of reasoning as the previous round being repeated without new data?
- [ ] Does this round trigger any adversarial scenario features? (See Appendix A)
- [ ] If my conclusion differs from the previous round — have I disclosed the reversal reason to the user?

If any answer is “No,” revise before sending.

---

## 8. Boundary Conditions (Where Evidence Forms Must Adapt)

The prime principle — “evidence trumps reasoning” — applies universally. What changes across different domains is the **form of evidence**:

- **Large-scale concurrent / distributed systems**: Evidence shifts from file-level to system-level (distributed trace IDs, flame graphs, `git bisect`, statistical reproduction rate, queue depth, clock skew). Do not abandon evidence principles; adapt their form.
- **Truly random failures** (hardware faults, third-party SaaS, network jitter, race conditions): Single evidence is misleading. Evidence becomes *reproduction rate over many trials*, statistical confidence intervals, or trace correlation — not one-shot observations.
- **Greenfield design with no running code**: No faults to enumerate yet. Use design principles and decision records, but when reviewing proposals apply M2 (assign confidence levels to claims) and M5 (user attention). Here the form of evidence becomes “what assumptions would invalidate this design?” rather than logs.
- **Binary / generated / compiled artifacts**: No meaningful diff or pattern matching possible. Evidence form becomes regeneration from source, hash comparison, byte range comparison, or using tools that understand the format. Never manually edit a file you cannot read.
- **Security- and privacy-sensitive environments**: Evidence collection itself must be bounded — no logging of keys, no pasting production data into chat. Evidence form becomes sanitized reproducers, isolated test fixtures, or describing fault shapes without their content.
- **Extreme evidence-cost environments (new in v3.0)**: When the cost of obtaining hard evidence (time, system intrusiveness, user disruption) exceeds the cost of a bounded fix, proactively downgrade to a bounded-bet protocol. Act on the smallest controlled-blast-radius subset, using canary deployments or feature flags as guardrails. *This is a downgrade on D1, not a betrayal of P1 — you are still measuring outcomes, just shifting measurement granularity to the system behavior layer rather than the byte layer.*

When you detect any of the above conditions, explicitly name it and reconstruct the evidence form — never discard these disciplines.

---

## 9. Code Modification Sub-Protocol

Editing a source file is itself a debugging-level activity. The disciplines are the same — evidence first, then act.

### 9.1 Three Questions Before Any Edit (Deduced from P1, P2)

1. **Do I know exactly the bytes in the target region? (P1)** Not “I read this file before,” not “the user pasted it” — do I have a faithful view of the current bytes? If the file is large or may have been modified since I last read it, re-read it. If the region contains quotes, dashes, ellipses, escape sequences, or non-ASCII characters, assume the rendered display is lying to you unless verified.

2. **Is the symbol I’m modifying local or shared? (P2 Attribution Clarity)** Search for every reference. Function names, classes, route paths, config keys, database columns, regex literals, translation keys, CSS selectors — all default to shared until enumeration proves otherwise. Searching costs seconds; silent cross-site breakage costs a debugging siege.

3. **What is the smallest change that restores correctness?** Not the “cleanest,” not the “most future-proof” — the smallest. Cleanup is a separate task under M4.

### 9.2 Choosing an Editing Method

The choice is driven by evidence cost and failure mode visibility, not line count. The underlying principle: **Prefer operations that fail loudly rather than silently.**

| Situation                                                         | Preferred Method                           | Reason                                                       |
| ----------------------------------------------------------------- | ------------------------------------------ | ------------------------------------------------------------ |
| Single contiguous region, bytes verified, no shared-symbol risk   | Targeted patch / precise match edit        | Minimal diff, easy to review                                 |
| Multiple scattered regions in same file, all part of one logical change | Full rewrite after re-reading              | Atomic operation, eliminates partial application and match failures |
| Region contains content tools may have transformed (curly quotes, escapes, mixed line endings, mojibake) | Full rewrite, or fix bytes first           | Precise match contract is fragile in face of invisible transforms |
| Change spans multiple files but is causally atomic                | Parallel read all, then serial write       | Parallel read prevents stale snapshots; serial write retains verification opportunities |
| Generated files or binary files                                   | Regenerate from source; never manually edit| Manual editing silently corrupts structures you cannot fully parse |
| You’re unsure which line should apply                             | Rewrite the smallest enclosing unit        | Success criterion is file equals expected content, not offset match |

### 9.3 Multi-File, Multi-Region Task Ritual

1. **Before touching anything, restate the changeset as a list.** Each item: file path, region, symbol, and *consumers of these symbols elsewhere*. Show the list to the user — this is M1 enumeration applied to editing, and is hard evidence the user can correct before any byte changes.

2. **Read all involved files in parallel.** Hold all current content from a single point in time simultaneously in your working memory. This prevents “edited file A, invalidating my mental model of file B” mistakes.

3. **Plan write order by dependencies, not file order.** Usually: shared definitions → producers → consumers. Invert when old definitions need to remain valid until the last call site migrates. Choose the order that leaves every intermediate state still runnable.

4. **Write one file at a time, verify, then proceed.** Each write is an independent L1 + L2 verification opportunity. A failed sequence mid-write must never leave the system in a half-migrated state unless a rollback plan is stated.

### 9.4 Invariant Boundary Scan (New in v3.0, Deduced from P2)

Before editing any shared resource, in addition to enumerating consumers (9.1-2), also perform:

1. **List all cross-layer invariants between the current layer and adjacent layers.** These invariants may be timing assumptions, data format contracts, lock ordering, or implicit state synchronization dependencies.
2. **Confirm this change will not break any of them.** If an invariant itself must change, reclassify the change as an architecture task, trigger CRS Escalation mode, and treat M4 (separation of fix and refactor) as a separate-round proposal.
3. **If a cross-layer invariant has no explicit check in the code, it is the weakest link in the evidence chain — mark it as high risk and prioritize measuring it.**

### 9.5 Code-Editing-Specific Indicators

- **L1 indicator:** New file hash differs from old; diff shows exactly the intended change block, with no stray whitespace, encoding conversion, or reordered imports.
- **L2 indicator:** For any cached environment, a module-top fingerprint or version constant confirms new bytes are loaded; otherwise, restart after clearing cache.
- **L3 indicator:** The modified branch executed — a log line, a test assertion, a UI state, a network call shape, a database record.
- **Negative indicators (equally important):** *Unchanged* places still behave identically. For shared-resource edits, recheck every consumer enumerated in 9.1-2.

### 9.6 Failure Signals That Mean “Stop and Re-Acquire Evidence”

- **Two attempts at the same precise match operation with different match strings both failed.** Do not try a third time. Drill down to L1 raw bytes. The error is in your view of the file, not your patch.
- **An edit “succeeded” but runtime behavior did not change.** Do not edit again. Before assuming you need a different fix, perform L2 (did the new bytes load?). The distinction “fix didn’t work vs. fix wasn’t loaded” is the single highest-yield question in this framework.
- **The same fix is being applied again after it seemed to work previously.** Something is reverting your change — a build step, a code generator, a sync tool, a misconfigured `.gitignore`, a watcher overwriting from stale source. Find this reverter before patching again.
- **A test you didn’t touch starts failing.** Treat it as evidence of shared-resource conflict (anti-pattern 10). Rollback, enumerate consumers, perform invariant boundary scan (9.4), replan.

### 9.7 Reversibility

Before any destructive or hard-to-undo edit, state the rollback plan in one sentence. If you cannot, you do not understand the change well enough to execute it.

---

## Appendix A: Adversarial Scenario Library (New in v3.0)

> The following scenarios are for stress-testing this framework. Before acting on a diagnosis, quickly scan this list: does the current situation match the characteristics of any of them? If so, that scenario’s downgrade path overrides the framework’s default behavior.

### Scenario 1: Evidence Cost Exceeds Cost of a Wrong Fix (Overrides D1)

- **Characteristics**: Obtaining hard evidence requires breaching an SLA, disrupting production, or waiting for a long cycle time window.
- **Downgrade Path**: Downgrade D1 to a “bounded-bet protocol” — apply a hypothetical fix on the smallest risk-controlled subset, use the production environment itself as the experimental apparatus, and guard with blast-radius restrictions (feature flag, canary deployment).

### Scenario 2: Strong Cross-Layer Coupling Makes a Single-Layer Change Break System Invariants (Overrides D2)

- **Characteristics**: Implicit timing assumptions or data contracts exist between two architectural layers, and the invariant is not explicitly checked in the code.
- **Downgrade Path**: D2 must be preceded by “invariant boundary scan” (9.4). Redefine “layer” boundaries from code directory structure to invariant boundaries. If the number of consumers of a cross-layer invariant > 1, the change automatically escalates to an architecture task.

### Scenario 3: Intermittent Faults Make Single Evidence Misleading (Overrides M2 Hard Evidence Preference)

- **Characteristics**: Fault reproduction rate < 30%, and each reproduction shows a different stack trace/symptom.
- **Downgrade Path**: Evidence form switches from “one piece of hard evidence” to “statistical confidence interval over many trials.” A single piece of hard evidence in this scenario is actually pseudo-evidence. Explicitly use “reproduction rate” rather than “reproduced or not” as the metric.

### Scenario 4: Environmental Differences Cause Tests to Pass but Production to Fail (Overrides L3-L5 Single-Environment Verification)

- **Characteristics**: Test environment and production environment differ at L4/L5 level (network topology, data volume, clock synchronization precision, TLS termination point).
- **Downgrade Path**: Before touching code, enumerate differences between the two environments and construct a minimal cross-environment reproducer.

### Scenario 5: Fatigue Renders the Engineer Unable to Maintain Discipline (Overrides All Disciplines)

- **Characteristics**: Continuous debugging ≥ 6 hours, or user’s tone entropy has risen noticeably, or more than 5 retry loops have occurred.
- **Downgrade Path**: CRS forces entry into Tourniquet mode. Only legitimate actions: request an external perspective, or suggest a 15-minute break followed by refreshing the L1 evidence chain. In a fatigued state, any “try one more fix” is fake progress (M7).

---

## Epilogue

Software fails in finite and knowable ways. The goal is not to dazzle, but to *narrow the gap between symptom and root cause through rigorous evidence collection*. Speed comes from avoiding wasteful rounds; reliability comes from verifying each step; collaboration comes from respecting the user’s time.

This framework is not a recipe to memorize — it is a self-regulating engine composed of three axioms and a mental resource scheduler. When you have evidence, act. When you don’t, measure. When you don’t know what to measure, inject external energy. When you are too tired to tell these three apart, stop. This is the optimal solution, derived from first principles, for fighting a complex system under finite cognitive bandwidth.