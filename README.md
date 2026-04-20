# PSHA System — Python-Structural Hook Architecture for AI Agent Workflows

**(Python型態 Skill-Hook 拓撲架構)**

> Using Python's structural concepts — design patterns, topology, state management — as the architectural language of AI agent skill↔hook workflows, not just as implementation tools.

---

## Overview

Most AI agent frameworks use Python to *execute* workflows but define workflow structure as flat prompt instructions or linear sequences. PSHA inverts this: Python's design vocabulary becomes the language that determines the **shape and topology** of how skills and hooks connect, communicate, and fail.

The insight: if AI frameworks are built in Python, Python's structural thinking should also define how those frameworks are *designed* — not just how they run.

---

## Core Concept

```
Workflow structure is not a list of steps.
It is a topology with a shape.
That shape is described in Python.
```

**Traditional approach:**
```
Skill → Hook → Execute → Done
(linear, flat, no structural vocabulary)
```

**PSHA approach:**
```
Skill (Intent Layer)
  └─ Hook (lifecycle-bound, paired at definition)
       └─ Python (Architectural Language Layer)
            ├─ Topology selection: Pipeline / DAG / Circuit Breaker / Saga / Pub-Sub
            ├─ Design patterns: State Machine / Rollback / Defensive / Observer
            └─ Context re-injection (Perception Layer → AI senses output, closes loop)
```

---

## Three-Layer Architecture

| Layer | Role | Mechanism |
|-------|------|-----------|
| **Intent Layer** (Skill) | Defines workflow goals and step semantics | Carries hook configuration in YAML frontmatter |
| **Architectural Language Layer** (Python) | Determines the shape of the flow | Design patterns and topology structures applied to skill↔hook pipeline |
| **Perception Layer** (Context Injection) | AI senses Python output and decides next action | Structured output re-injected into model context, closing the loop |

---

## Design Principles

PSHA is built on four structural principles applied across three granularity layers (single unit → node composition → flow orchestration). Each principle maps to established Python design vocabulary — applied here as architectural decisions about flow shape, not just implementation choices.

*Detailed specification not published. Contact for collaboration.*

---

## Minimal Paradigm

The following illustrates the core structural idea — Python pattern vocabulary determining flow topology, not just executing it:

```python
# Skill carries its paired hook at definition (lifecycle binding)
# YAML frontmatter embeds hook config — no global hook registry needed

# Python layer selects topology based on flow requirements
class SkillHookBridge:
    topology: str  # "pipeline" | "circuit_breaker" | "dag" | "saga" | ...

    def on_trigger(self, event):
        result = self.apply_topology(event)      # shape determined by Python pattern
        return self.inject_context(result)        # output re-injected → AI perceives → loop closes
```

This is a structural sketch. A partial working implementation exists and has been validated in a production AI agent harness.

---

## Why This Matters

Industry frameworks ask: *"Which framework should I use?"*

PSHA asks: *"What shape should this flow have?"*

These are different questions. The second has been systematically unaddressed — engineering backgrounds optimize execution, not structure. Python's design vocabulary has always been available. It just wasn't applied to the architectural layer.

---

## Status

| Item | Status |
|------|--------|
| Concept design | ✅ Complete (2026-04-04) |
| Architecture specification | ✅ Complete |
| Partial implementation | ✅ Validated in production harness |
| Full open-source release | ⏳ Planned |

---

## Contact

Interested in collaboration, research use, or implementation details:
**basazak1@gmail.com**

---

## Author

**Yang, Shih-Hung (William Yang)**
Concept originated: 2026-04-04

---

*License: CC BY 4.0 (concept documentation) · Implementation rights reserved*
