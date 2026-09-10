# FlowField 3.0 Workflow Model

## Stateful Orchestration for Field Operations

FlowField 3.0 models field work as a **stateful operational process**.

A field activity is not simply a sequence of forms.

It is an executable workflow whose progression depends on:

- current state
- completed tasks
- structured module outputs
- operational events
- validation rules
- dependencies
- blocking conditions
- escalation policies
- closure requirements

The workflow engine coordinates these elements while the field user sees only the actions that are relevant at that moment.

---

## Core Model

At a high level:

```text
Current State
     |
     v
Operational Event
     |
     v
Rule Evaluation
     |
     +--------------------+
     |                    |
     v                    v
Condition Met       Condition Failed
     |                    |
     v                    v
Transition          Block / Branch
     |                    |
     +----------+---------+
                |
                v
            New State
```

This loop continues until the activity reaches a valid terminal condition.

---

## Workflow Status

A FlowField workflow can conceptually move through global states such as:

```text
INITIALIZED
     |
     v
IN_PROGRESS
     |
     +----------> BLOCKED
     |               |
     |               v
     |          condition resolved
     |               |
     +<--------------+
     |
     +----------> TERMINATED
     |
     v
CLOSED
```

The exact internal state graph can vary by deployment and activity configuration.

The important architectural principle is that transitions are explicit and governed.

---

## State Is Not a Screen

Workflow state and user interface state are intentionally separate.

A technician may see:

```text
Take required photo
```

while the orchestration engine understands:

```text
workflow = IN_PROGRESS
module = MOD_DATA_CAPTURE
task = evidence_required
closure_allowed = false
```

The UI represents the next operational action.

The workflow engine owns process state.

---

## Tasks

Tasks are generated or enabled by configured capability modules.

Conceptually:

```text
Activity Definition
        |
        v
Configured Modules
        |
        v
Module Tasks
        |
        v
MOD_FLOW
        |
        v
Executable Sequence
```

Tasks do not originate directly from a free-form activity description.

They originate from configured capabilities and are governed by the orchestration layer.

---

## Task States

A task can conceptually have lifecycle states such as:

```text
PENDING
   |
   v
AVAILABLE
   |
   v
IN_PROGRESS
   |
   +------> BLOCKED
   |
   v
COMPLETED
```

Some activities may also support states such as:

```text
SKIPPED
NOT_APPLICABLE
FAILED
REQUIRES_REVIEW
```

Their availability depends on workflow policy.

A mandatory task cannot simply disappear because the user moves to another screen.

---

## Dependencies

FlowField supports dependencies between tasks and capabilities.

A task may become available only when another task has completed.

```text
Task A
  |
  v
Task B
  |
  v
Task C
```

Dependencies can also cross capability boundaries.

```text
MOD_CONTEXT
     |
     v
Context Valid
     |
     v
MOD_SAFETY
     |
     v
Safety Valid
     |
     v
Operational Tasks
```

In regulated activities, safety may therefore act as a prerequisite for later operational capabilities.

---

## Events

Modules communicate relevant operational changes through structured events.

Conceptually:

```text
Capability
    |
    v
Structured Event
    |
    v
MOD_FLOW
    |
    v
Workflow Decision
```

Representative public examples include:

```text
context_verified
access_denied
safety_failed
asset_mismatch
measurement_out_of_range
evidence_missing
criticality_detected
action_required
document_incomplete
closure_requested
```

These names are illustrative.

The complete internal event catalog is intentionally not part of the public repository.

---

## Why Events Matter

Without an event boundary, modules can become tightly coupled.

For example, this architecture is undesirable:

```text
DATA_CAPTURE
   directly changes
CRITICALITY
   directly changes
ACTIONS
   directly changes
workflow state
```

FlowField instead follows:

```text
MOD_DATA_CAPTURE
      |
      v
    Event
      |
      v
   MOD_FLOW
      |
      v
Orchestration Decision
```

When another capability must participate, it can be activated through the controlled orchestration path.

This reduces cross-module coupling and keeps capability responsibilities clear.

---

## Conditions

An event does not always produce the same result.

The workflow evaluates the event within the current operational context.

Example:

```text
Measurement Recorded
         |
         v
     In Range?
      /     \
    Yes      No
     |        |
     v        v
 Continue   Evaluate Policy
              |
         +----+----+
         |         |
         v         v
      Warning    Criticality
```

The condition is deterministic when it represents an operational rule.

---

## Branching

FlowField workflows do not need to be strictly linear.

A workflow can branch according to structured conditions.

```text
          Inspection
              |
              v
        Issue Detected?
          /        \
        No          Yes
        |            |
        v            v
   Continue      Classify Issue
                     |
                     v
                Severity?
               /    |     \
             Low   High  Critical
              |     |       |
              v     v       v
          Continue Action Escalate
```

The technician still experiences a guided process.

The orchestration engine manages the branches.

---

## Blocking

A workflow can prevent progression when an operational requirement has not been satisfied.

Example:

```text
Safety Verification
        |
        v
      PASS?
     /     \
   Yes      No
    |        |
    v        v
Continue   BLOCKED
             |
             v
       Correct Condition
             |
             v
          Revalidate
```

Potential blocking conditions can include:

- failed authorization
- unsafe working conditions
- missing required evidence
- invalid asset identification
- incomplete documentation
- unresolved mandatory corrective actions
- failed closure requirements

Blocking is a process decision, not a visual warning.

---

## Hard Stops

Some events may require termination rather than temporary blocking.

Conceptually:

```text
Stop-Work Condition
        |
        v
Evaluate Policy
        |
        v
TERMINATED
```

This is particularly relevant when continuing the operation would violate safety or authorization rules.

---

## Interrupt Events

Not every event occurs in the expected sequence.

Some operational events can occur at any moment.

For example:

```text
Normal Execution
      |
      |
      +----> Stop-Work Event
                    |
                    v
                 MOD_FLOW
                    |
                    v
             Immediate Response
```

The safety model therefore supports conditions that act as workflow interrupts rather than ordinary sequential tasks.

---

## Escalation

A workflow can escalate when a condition exceeds the authority or responsibility of the current operator.

```text
Critical Condition
        |
        v
Escalation Required
        |
        v
Supervisor / External Process
        |
        +----------+
        |          |
        v          v
    Approved    Rejected
        |          |
        v          v
    Continue     Block
```

Escalation policy can vary according to:

- severity
- activity type
- organization
- role
- compliance requirements
- customer configuration

The public architecture does not define customer-specific escalation rules.

---

## Corrective Actions

Detection of a problem and remediation of that problem are separate concepts.

```text
Evidence
    |
    v
MOD_CRITICALITY
    |
    v
Validated Issue
    |
    v
MOD_ACTIONS
    |
    v
Corrective Action
```

`MOD_FLOW` decides how the state of that corrective action affects the parent workflow.

For example:

```text
Corrective Action
       |
       v
Still Open?
    /      \
  Yes       No
   |         |
   v         v
Block      Continue
Closure
```

Whether an open action blocks closure is configuration-driven.

---

## Evidence Requirements

FlowField can require specific evidence before progression or closure.

Example:

```text
Required Evidence
      |
      +--> Photo A
      +--> Photo B
      +--> Measurement
      `--> Checklist
              |
              v
       Requirements Met?
          /        \
        Yes          No
         |            |
         v            v
     Continue     Keep Blocked
```

Evidence requirements belong to the relevant capability configuration.

`MOD_FLOW` evaluates their completion state.

---

## Closure Is a Workflow State

Closure is not treated as an independent operational capability.

It is the result of satisfying the configured workflow conditions.

```text
Closure Requested
       |
       v
Validate Preconditions
       |
       +--> Mandatory tasks complete?
       |
       +--> Required evidence present?
       |
       +--> Documentation complete?
       |
       +--> Blocking actions resolved?
       |
       +--> Required approvals complete?
       |
       v
    All Valid?
     /     \
   Yes      No
    |        |
    v        v
 CLOSED    Reject Closure
```

This distinction prevents a workflow from being marked complete simply because a user presses a "Finish" button.

---

## Controlled Closure

A successful closure can conceptually produce:

```text
workflow_status = CLOSED
closure_check   = OK
closed_at       = timestamp
```

A failed closure request instead preserves the active workflow and communicates the unresolved requirement.

```text
workflow_status = IN_PROGRESS / BLOCKED
closure_check   = FAILED
reason          = unresolved requirement
```

---

## Audit Trail

Workflow changes produce an operational history.

Conceptually:

```text
Activity Created
      |
      v
Event
      |
      v
Transition
      |
      v
Event
      |
      v
Transition
      |
      v
...
      |
      v
Activity Closed
```

An audit record can preserve information such as:

```text
timestamp
workflow
event type
previous state
resulting state
actor
relevant structured payload
```

The reference workflow implementation already follows an append-oriented event history model.

The public repository describes the principle without exposing production persistence details.

---

## Deterministic Transition Authority

The workflow engine is the authority for critical state transitions.

Conceptually:

```text
Request
   |
   v
Is transition allowed?
   |
 +---+---+
 |       |
Yes      No
 |       |
 v       v
Apply   Reject
State
```

A client should not be able to arbitrarily move an activity from one state to another.

This keeps state progression predictable and auditable.

---

## AI and Workflow Authority

FlowField can use AI without giving the AI model uncontrolled authority over the workflow.

```text
AI
 |
 +--> interpret
 +--> assist
 +--> suggest
 +--> analyze
 +--> classify candidates

MOD_FLOW
 |
 +--> validate
 +--> enforce
 +--> transition
 +--> block
 +--> escalate
 +--> authorize closure
```

This is a fundamental FlowField principle:

**AI can contribute intelligence.  
The orchestration layer retains authority.**

---

## Outy Integration

The reference architecture can use **Outy**, the edge-oriented AI layer developed within the CashOut ecosystem.

Outy can assist the interaction and interpretation layers while remaining separated from workflow authority.

```text
Technician / Web User
          |
          v
         Outy
          |
          v
 AI Interpretation / Assistance
          |
          v
     FlowField Core
          |
          v
 Deterministic Workflow
```

Outy is the reference integration.

FlowField can connect to alternative AI providers through the same provider-agnostic architecture.

---

## VLM and Visual Evidence

Visual AI follows the same governance model.

```text
Technician Photo
       |
       v
MOD_DATA_CAPTURE
       |
       v
VLM Analysis
       |
       v
Suggested Observation
       |
       v
Validation Boundary
       |
       v
Operational Event
       |
       v
MOD_FLOW
```

The current reference implementation can use a GPT-based cloud VLM.

A local or edge VLM can replace it when the deployment environment has sufficient compute capacity.

The original photograph remains operational evidence.

The VLM output is a derived interpretation.

---

## AI Does Not Replace Evidence

This distinction is important:

```text
Original Photo
     |
     +--> Evidence
     |
     `--> VLM
            |
            v
       Interpretation
```

The interpretation does not overwrite the original evidence.

This preserves traceability and allows later review.

---

## Example: Measurement Outside Range

Consider a configured technical measurement.

```text
Technician records value
          |
          v
MOD_DATA_CAPTURE
          |
          v
Deterministic validation
          |
     +----+----+
     |         |
   valid    out-of-range
     |         |
     v         v
 continue    event
               |
               v
            MOD_FLOW
               |
          +----+-----+
          |          |
          v          v
      request      create
      evidence   criticality path
```

The exact response depends on the activity configuration.

---

## Example: Visual Anomaly

```text
Technician captures image
          |
          v
       Evidence
          |
          v
      VLM analysis
          |
          v
Possible damaged component
          |
          v
Operator / Rule Validation
          |
        accepted
          |
          v
    MOD_CRITICALITY
          |
          v
       Severity
          |
          v
       MOD_FLOW
          |
      +---+----+
      |        |
      v        v
 Continue   Action / Block
```

The model assists the workflow without independently controlling it.

---

## Example: Access Denied

```text
Activity Started
      |
      v
MOD_CONTEXT
      |
      v
Physical Access
      |
   +--+--+
   |     |
Granted Denied
   |     |
   v     v
Continue Controlled Termination
```

An access denial is not treated as an ordinary failed form field.

It changes the operational path.

---

## Example: Safety Interrupt

```text
Inspection in progress
        |
        v
 Unsafe Condition Detected
        |
        v
     Stop Work
        |
        v
      MOD_FLOW
        |
        v
  Immediate Block /
    Termination
```

This allows the workflow to react to field reality rather than assuming execution always follows the happy path.

---

## Example: Missing Documentation

```text
Operational Work Complete
          |
          v
   Closure Requested
          |
          v
Documentation Complete?
       /       \
     Yes        No
      |          |
      v          v
   Continue     BLOCK
      |          |
      |       Complete Docs
      |          |
      +<---------+
      |
      v
    CLOSED
```

---

## Runtime Model

Conceptually, the orchestration runtime performs a continuous loop:

```text
1. Read current workflow state
2. Receive structured event
3. Read relevant module outputs
4. Evaluate configured rules
5. Validate transition
6. Update workflow state
7. Determine permitted next actions
8. Record the operational event
9. Return the next field action
```

The field client does not need to know the complete workflow graph.

It only needs the result:

```text
current state
permitted actions
required input
blocking reason
```

---

## Invisible Orchestration

`MOD_FLOW` should generally remain invisible to the field technician.

The technician sees:

```text
Take a photo of the equipment label.
```

The engine may simultaneously understand:

```text
asset identification completed
required evidence pending
criticality path inactive
closure unavailable
next permitted task = evidence capture
```

This is deliberate.

The complexity belongs to the platform, not to the technician.

---

## Workflow Configuration

Different activities can use the same engine with different rules.

Example:

```text
Activity A

Context
  |
Safety
  |
Capture
  |
Close
```

Another activity may require:

```text
Activity B

Context
  |
Safety
  |
Asset
  |
Capture
  |
Criticality
  |
Action
  |
Documentation
  |
Close
```

FlowField changes configuration rather than replacing the orchestration engine.

---

## Human-in-the-Loop

FlowField can preserve human authority where operational risk requires it.

For example:

```text
AI / Rule Suggestion
        |
        v
Supervisor Review
        |
     +--+--+
     |     |
 Accept  Reject
     |     |
     v     v
Transition No Transition
```

The validation policy can be configured according to the deployment.

---

## Offline and Edge-Oriented Execution

The workflow model is compatible with edge-oriented deployments.

Where architecture permits, field execution can keep local operational state and synchronize with central systems when connectivity becomes available.

Conceptually:

```text
Field Client
     |
     v
Local / Edge Runtime
     |
     +--> workflow interaction
     +--> evidence capture
     +--> optional edge AI
     |
 connectivity available
     |
     v
Central Platform / Enterprise Systems
```

Exact offline synchronization mechanisms are implementation-specific and outside the scope of this public reference document.

---

## Public Workflow Contract

The public FlowField workflow model can be summarized as:

```text
STATE
  +
EVENT
  +
RULE
  +
CONDITION
  =
CONTROLLED TRANSITION
```

With:

```text
dependencies
branching
blocking
escalation
actions
evidence
audit
closure validation
```

around that transition model.

---

## Public vs Internal Scope

This document intentionally describes the orchestration model without exposing:

- the complete internal event catalog
- proprietary event identifiers
- full transition tables
- complete workflow configuration schemas
- customer-specific policies
- internal rule evaluation implementation
- production orchestration code
- private escalation policies

Those elements belong to FlowField's implementation and commercial deployment layers.

---

## Summary

FlowField 3.0 treats a field activity as a controlled state machine.

```text
Operational Reality
        |
        v
Structured Events
        |
        v
Deterministic Rules
        |
        v
MOD_FLOW
        |
        +--> Continue
        +--> Branch
        +--> Block
        +--> Escalate
        +--> Open Action
        +--> Terminate
        `--> Close
```

This is what separates FlowField from a static field form or checklist system.

**Forms collect information.  
FlowField governs what happens next.**
