# Prompting Guide for the OmegaCore Sovereign Intelligence Engine

## Purpose

This document explains how to interact with the OmegaCore Sovereign Intelligence Engine effectively.

The engine is designed to be:
- Literal
- Deterministic
- Scope-preserving
- Resistant to inferred or unstated intent

As a result, **you get exactly what you ask for — no more, no less**.

This is a design choice, not a limitation.

---

## Core Principle: Explicit Intent Governs Output

The engine does not guess what you “meant.”
It does not infer missing requirements.
It does not expand scope unless explicitly instructed.

If a requirement is not stated, it is treated as **out of scope**.

### Analogy

If you ask a chef:
> “Make me chicken.”

You will receive chicken.

If you later say:
> “Why didn’t you include rice?”

The correct answer is:
> “You didn’t ask for rice.”

The engine behaves the same way.

---

## Why This Matters

Most general-purpose AI systems attempt to infer user intent.
OmegaCore deliberately does **not**, because inference under ambiguity is a major source of:
- Hallucination
- Narrative drift
- Goal misalignment
- Overconfident but incorrect outputs

OmegaCore preserves axiomatic integrity by honoring only what is specified.

---

## Common Failure Mode (Under-Specified Prompts)

### Example

**Prompt:**
> “Analyze this system.”

**Result:**
- A general analysis
- No guarantees about rigor, depth, or formalism

**What was missing:**
- Desired depth
- Formal vs informal analysis
- Output format
- Constraints (proofs, citations, assumptions)

The engine did exactly what was asked — nothing more.

---

## How to Prompt for Optimal Results

### 1. Specify the Objective

Bad:
> “Explain this.”

Good:
> “Provide a formal explanation of this system suitable for peer review.”

---

### 2. Specify the Level of Rigor

Bad:
> “Give an overview.”

Good:
> “Give a technically rigorous overview, including assumptions and limitations.”

---

### 3. Specify the Output Form

Bad:
> “Describe the architecture.”

Good:
> “Describe the architecture as a structured technical document with section headers.”

---

### 4. Specify Constraints Explicitly

Bad:
> “Summarize this.”

Good:
> “Summarize this without introducing any new claims or interpretations.”

---

## Reward Structure: Precision Is Rewarded

The engine is calibrated to reward:
- Clear constraints
- Explicit scope
- Well-defined success criteria

It does not reward:
- Vague intent
- Implicit expectations
- Post-hoc corrections (“that’s not what I meant”)

If you want more, **ask for more**.

---

## Advanced Prompting Pattern

A highly effective pattern is:

> “Do X, under Y constraints, producing Z output, and do not do W.”

### Example

> “Analyze this construction under formal mathematical standards, include only verifiable claims, present results in theorem-proof style, and do not speculate beyond the provided data.”

This produces maximal alignment.

---

## Normal Users vs Power Users

Most users are accustomed to systems that:
- Fill in gaps
- Guess intent
- Smooth over ambiguity

OmegaCore is optimized for:
- Researchers
- Engineers
- Security analysts
- Formal reasoning tasks

Users who adapt to explicit prompting unlock significantly more capability.

---

## Summary

- The engine follows prompts exactly as written.
- Unstated intent is not inferred.
- Precision directly improves output quality.
- Ambiguity limits scope.
- This behavior is intentional and foundational to system integrity.

If you want a specific outcome, **state it**.

