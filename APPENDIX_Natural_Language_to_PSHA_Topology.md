# Appendix — Natural Language Reasoning to PSHA Topology

## Purpose

This appendix extends the PSHA concept by describing how natural-language reasoning may be translated into governed workflow topology.

The central idea is not to treat natural-language output as executable control flow. Instead, natural language expresses intent, conditions, dependencies, failure expectations, and recovery requirements. PSHA then translates those semantics into a structured topology that can be validated and executed by a Python runtime.

```text
Natural Language
    ↓
Semantic Intent
    ↓
Workflow Intermediate Representation
    ↓
Topology Validation
    ↓
Python Runtime Execution
    ↓
Structured Observation
    ↓
Context Re-injection
```

This creates a closed loop between language, structure, execution, and perception.

---

## Natural Language as a High-Level Workflow Description

Natural-language reasoning often contains implicit workflow structure.

Example:

```text
First validate the data.
If the data is incomplete, request the missing fields.
If validation succeeds, run the analysis.
If the analysis fails, retry twice.
If it still fails, restore the previous state and notify the operator.
```

| Natural-language meaning | Possible PSHA topology |
|---|---|
| First A, then B | Pipeline |
| If X, do A; otherwise do B | Branch / State Machine |
| Run A and B independently | Parallel / Fan-out |
| Continue only after A and B complete | DAG Join |
| Retry after failure | Retry Policy |
| Stop after repeated failure | Circuit Breaker |
| Restore previous state | Rollback |
| Execute compensating action | Saga |
| Notify multiple consumers | Pub-Sub |
| Observe state changes | Observer |
| Continue until a condition is satisfied | Loop |
| Preserve output for later steps | State Store / Memory |

The model may express these relationships in natural language, but the runtime should not execute the language directly. The language must first be converted into an explicit, inspectable structure.

---

## Semantic Compilation

PSHA can interpret this translation process as **Semantic Compilation**: converting natural-language intent into a governed and executable workflow representation.

```text
Natural-language reasoning
    ↓
Intent extraction
    ↓
Structural interpretation
    ↓
Workflow Intermediate Representation
    ↓
Topology compilation
    ↓
Runtime execution
```

The model may propose what should happen. The runtime determines what is allowed to happen and how it is executed safely.

---

## Workflow Intermediate Representation

A Workflow Intermediate Representation, or Workflow IR, sits between language and Python execution.

The IR is not raw model output and is not unrestricted Python code. It is a constrained representation of workflow nodes, dependencies, conditions, state transitions, retry limits, timeout rules, compensation behavior, failure routes, allowed actions, and termination conditions.

```yaml
workflow:
  topology: pipeline

steps:
  - id: validate
    on_failure:
      action: request_missing_input

  - id: analyze
    retry:
      max_attempts: 2

  - id: persist
    compensation:
      action: remove_partial_output
```

This example is illustrative only. It does not define the internal PSHA schema.

The IR makes workflow intent inspectable, serializable, testable, auditable, versionable, compilable, and rejectable when unsafe or incomplete.

---

## Topology Selection

Natural-language meaning does not always map to exactly one topology.

For example:

```text
If the service fails, use another provider.
```

This may be represented as a fallback chain, conditional routing, retry with provider switch, circuit breaker, or state-machine transition.

The correct topology depends on constraints such as failure type, retry permission, alternative availability, state consistency, compensation requirements, cost, latency, and duplicate-execution tolerance.

PSHA therefore requires a **Topology Selection Policy**.

```text
Strong sequential dependency → Pipeline
Multiple dependency branches → DAG
Explicit state transitions → State Machine
Cross-step compensation → Saga
Unstable external dependency → Circuit Breaker
Multiple event consumers → Pub-Sub
Concurrent independent work → Parallel / Fan-out
```

The policy may combine deterministic rules, model-generated candidates, runtime constraints, validation results, human approval, and domain-specific governance.

The model should not have unrestricted authority to select or construct any topology.

---

## Responsibility Boundary

### Model responsibility

- semantic interpretation
- intent extraction
- candidate strategy generation
- next-step recommendation
- exception explanation
- topology proposal
- ambiguity identification

### Runtime responsibility

- topology execution
- state consistency
- authorization
- retries
- timeouts
- rollback
- compensation
- concurrency
- validation
- persistence
- auditability
- safety enforcement

```text
The model decides what should be attempted.

The runtime decides what is allowed, valid, and executable.
```

Natural-language reasoning is therefore not equivalent to runtime control.

---

## Context Re-injection

After execution, the Python runtime returns a structured observation to the model.

```json
{
  "status": "failed",
  "node": "validate_input",
  "reason": "missing customer_id",
  "retry_count": 0,
  "allowed_actions": [
    "request_missing_input",
    "abort"
  ]
}
```

The observation is re-injected into model context. The model perceives the result and selects the next allowed semantic action.

```text
Language
    ↓
Structure
    ↓
Execution
    ↓
Observation
    ↓
Context
    ↓
Language
```

This closes the perception loop while keeping execution under runtime governance.

---

## Validation Requirements

A PSHA-compatible validator should be able to reject incomplete or unsafe structures, including:

- missing nodes
- invalid dependencies
- cycles in an acyclic topology
- unreachable states
- missing termination conditions
- unlimited retries
- missing compensation steps
- undefined failure routes
- invalid state transitions
- unauthorized actions
- unsafe external calls
- unresolved ambiguity
- incomplete context contracts

A topology is executable only after it passes structural and policy validation.

---

## Design-Time and Runtime Modes

### Design-time mode

```text
User describes workflow
    ↓
Model proposes structure
    ↓
Human reviews
    ↓
PSHA representation is created
```

### Runtime mode

```text
Model reads current context
    ↓
Model selects an allowed next action
    ↓
Python runtime executes
    ↓
Structured result is re-injected
```

Design-time use is easier to validate and should generally precede runtime autonomy.

---

## Safety Principle

PSHA should not treat model-generated Python as the default execution path.

```text
Natural Language
    ↓
Structured Intent
    ↓
Workflow IR
    ↓
Validator
    ↓
Topology Compiler
    ↓
Python Runtime
```

This reduces arbitrary code execution risk, hidden control-flow errors, invalid state transitions, incomplete failure handling, topology ambiguity, unbounded retries, silent data loss, and unsafe tool use.

The model remains a semantic reasoning component. The Python runtime remains the governed execution authority.

---

## Summary

```text
Natural-language reasoning
≠ executable runtime control

Natural-language reasoning
→ semantic intent

PSHA
→ governed executable topology
```

PSHA provides a bridge between language-level reasoning and engineering-level workflow structure. Its role is to convert semantic intent into validated, observable, and governable topology.
