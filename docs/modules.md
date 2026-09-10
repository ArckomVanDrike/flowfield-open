# FlowField 3.0 Modules

## Capability Model

FlowField 3.0 is built around eight reusable capability modules.

These modules are not applications, screens or industry-specific workflows.

They are independent operational capabilities that can be combined and configured to build different field processes.

```text
                    FLOWFIELD 3.0

 +--------------------------------------------------+
 |                ORCHESTRATION                     |
 |                                                  |
 |                   MOD_FLOW                       |
 +-------------------------+------------------------+
                           |
                           v
 +--------------------------------------------------+
 |              OPERATIONAL CAPABILITIES            |
 |                                                  |
 | MOD_CONTEXT          MOD_SAFETY                  |
 | MOD_ASSET            MOD_DATA_CAPTURE            |
 | MOD_CRITICALITY      MOD_DOCUMENTATION           |
 | MOD_ACTIONS                                      |
 +--------------------------------------------------+
```

An activity uses only the capabilities required for its operational context.

---

## Design Principle

Each module has a clear responsibility.

A module should:

- perform one operational capability
- own the data related to that capability
- validate its own structured requirements
- produce structured outputs
- emit events relevant to orchestration

A module should not absorb responsibilities that belong to another capability.

This separation is fundamental to FlowField's scalability.

---

# MOD_CONTEXT

## Purpose

`MOD_CONTEXT` establishes the operational context of the activity.

It represents the entry point into a field workflow.

Typical responsibilities include:

- operator identification
- session context
- activity identification
- site identification
- geolocation
- timestamps
- logical authorization
- physical access confirmation
- acknowledgement of required procedures

Conceptually:

```text
Who is performing the work?
          +
What activity is being performed?
          +
Where is it happening?
          +
Is execution authorized?
          |
          v
      MOD_CONTEXT
```

## Typical Outputs

Examples of contextual information include:

```text
operator
activity
site
location
session
authorization state
physical access state
start time
```

## Boundary

`MOD_CONTEXT` does not perform safety assessment.

Authorization and access belong to context.

Operational safety conditions belong to `MOD_SAFETY`.

---

# MOD_SAFETY

## Purpose

`MOD_SAFETY` manages operational safety and compliance requirements.

It can be used before work begins or during execution when safety conditions change.

Typical responsibilities include:

- PPE verification
- site safety conditions
- operational risk assessment
- safety briefing acknowledgement
- work permit verification
- responsible-person confirmation
- stop-work conditions

Conceptually:

```text
Can this activity be performed safely?
              |
              v
         MOD_SAFETY
              |
       +------+------+
       |             |
       v             v
    Continue       Block
```

## Typical Outputs

Examples include:

```text
PPE status
risk level
permit status
site safety state
safety acknowledgements
stop-work state
```

## Boundary

Safety observations may require evidence, but the evidence itself belongs to `MOD_DATA_CAPTURE`.

A safety condition can affect workflow progression through `MOD_FLOW`.

---

# MOD_ASSET

## Purpose

`MOD_ASSET` manages the identity and lifecycle of physical or logical assets involved in an activity.

Typical responsibilities include:

- asset selection
- QR identification
- NFC identification
- serial number verification
- asset state verification
- installation
- removal
- replacement
- inventory-related operations

Conceptually:

```text
Identify
   |
   v
Verify
   |
   v
Operate
   |
   +--> Install
   +--> Remove
   +--> Replace
   `--> Update State
```

## Typical Outputs

Examples include:

```text
asset identity
identifier type
verification result
state before intervention
state after intervention
installation information
replacement information
inventory information
```

## Boundary

Asset information remains separate from generic field evidence.

For example:

```text
Asset serial number     -> MOD_ASSET
Photo of the asset      -> MOD_DATA_CAPTURE
Detected defect         -> MOD_CRITICALITY
Required repair         -> MOD_ACTIONS
```

---

# MOD_DATA_CAPTURE

## Purpose

`MOD_DATA_CAPTURE` is the evidence collection capability.

Its role is intentionally neutral.

It records what was observed in the field.

Typical data types include:

- photographs
- video
- audio
- measurements
- technical notes
- structured notes
- checklists
- graphical annotations
- operational attachments

Conceptually:

```text
Field Reality
     |
     v
Capture Evidence
     |
     v
Structured Data
```

## Typical Outputs

Examples include:

```text
media
measurements
notes
checklist results
annotations
operational attachments
timestamps
location metadata
```

## Critical Boundary

`MOD_DATA_CAPTURE` does not decide whether something is wrong.

It collects.

```text
Observe
   |
   v
MOD_DATA_CAPTURE
   |
   v
Evidence
```

Questions such as:

```text
Is this an anomaly?
How severe is it?
Should an action be created?
Should the workflow be blocked?
```

belong to other modules.

---

# MOD_CRITICALITY

## Purpose

`MOD_CRITICALITY` manages anomalies, non-conformities and operational criticalities.

It transforms an observation into a classified operational condition.

Typical responsibilities include:

- opening a criticality
- category classification
- severity assessment
- issue description
- localization
- association with evidence
- impact assessment
- urgency assessment
- validation or rejection of a reported condition

Conceptually:

```text
Evidence
   |
   v
Possible Issue
   |
   v
Classification
   |
   v
Severity / Impact
   |
   v
Validated Criticality
```

## Typical Outputs

Examples include:

```text
criticality identity
category
severity
description
location reference
evidence references
impact
urgency
validation state
```

## Boundary

`MOD_CRITICALITY` answers:

**What is the problem and how important is it?**

It does not manage how the problem will be corrected.

That belongs to `MOD_ACTIONS`.

---

# MOD_DOCUMENTATION

## Purpose

`MOD_DOCUMENTATION` manages formal documents associated with an activity.

This is distinct from operational evidence.

Typical responsibilities include:

- formal reports
- certificates
- compliance documents
- regulatory attachments
- required forms
- document metadata
- completeness controls
- document versions

Conceptually:

```text
Formal Requirement
       |
       v
Document
       |
       v
Validation
       |
       v
Controlled Record
```

## Typical Outputs

Examples include:

```text
document reference
document type
version
status
metadata
completeness state
validation state
```

## Boundary

A photograph taken during an inspection is operational evidence.

It belongs to `MOD_DATA_CAPTURE`.

A signed compliance certificate is formal documentation.

It belongs to `MOD_DOCUMENTATION`.

---

# MOD_ACTIONS

## Purpose

`MOD_ACTIONS` manages the lifecycle of corrective or derived activities.

It connects detection with remediation.

Typical responsibilities include:

- corrective action creation
- assignment
- ownership
- priority
- due dates
- SLA
- remediation status
- follow-up
- completion

Conceptually:

```text
Criticality
     |
     v
Required Action
     |
     v
Assignment
     |
     v
Remediation
     |
     v
Completion
```

## Typical Outputs

Examples include:

```text
action identity
source criticality
assignee
priority
status
deadline
SLA state
completion information
```

## Boundary

`MOD_ACTIONS` does not classify the original anomaly.

It manages what must happen after a corrective action is required.

---

# MOD_FLOW

## Purpose

`MOD_FLOW` is the orchestration capability.

It governs how the complete operational process progresses.

Typical responsibilities include:

- task sequencing
- workflow state
- conditions
- dependencies
- branching
- blocking
- escalation
- mandatory completion controls
- closure authorization

Conceptually:

```text
                   EVENT
                     |
                     v
              +--------------+
              |   MOD_FLOW   |
              +------+-------+
                     |
        +------------+------------+
        |            |            |
        v            v            v
     Continue      Branch       Block
        |            |            |
        +------------+------------+
                     |
                     v
                 New State
```

## What MOD_FLOW Does Not Do

`MOD_FLOW` does not:

- take photographs
- classify anomalies
- identify assets
- create technical measurements
- manage documents internally
- perform remediation

It governs the process around those capabilities.

This distinction is central to FlowField.

---

# Module Interaction

The modules cooperate without collapsing their responsibilities.

A typical interaction may look like:

```text
MOD_CONTEXT
     |
     v
MOD_SAFETY
     |
     v
MOD_ASSET
     |
     v
MOD_DATA_CAPTURE
     |
     v
MOD_CRITICALITY
     |
     v
MOD_ACTIONS
```

Throughout this process:

```text
          +----------------+
          |    MOD_FLOW    |
          +----------------+
             ^   ^   ^   ^
             |   |   |   |
          events and states
```

`MOD_FLOW` observes relevant operational events and determines how the workflow should progress.

---

# Example: Infrastructure Inspection

An infrastructure inspection may activate:

```text
MOD_CONTEXT         enabled
MOD_SAFETY          enabled
MOD_ASSET           enabled
MOD_DATA_CAPTURE    enabled
MOD_CRITICALITY     enabled
MOD_DOCUMENTATION   enabled
MOD_ACTIONS         enabled
MOD_FLOW            enabled
```

The technician might experience:

```text
Check in
   |
Validate safety
   |
Scan equipment
   |
Take photographs
   |
Record measurements
   |
Report anomaly
   |
Create remediation
   |
Complete documentation
   |
Close activity
```

Internally, every operation remains associated with its capability domain.

---

# Example: Simple Site Survey

A simpler activity may require fewer capabilities:

```text
MOD_CONTEXT         enabled
MOD_SAFETY          optional
MOD_ASSET           disabled
MOD_DATA_CAPTURE    enabled
MOD_CRITICALITY     enabled
MOD_DOCUMENTATION   disabled
MOD_ACTIONS         optional
MOD_FLOW            enabled
```

The architecture does not require every activity to activate every module.

This is what makes the capability model reusable.

---

# Example: Asset Replacement

A replacement operation may emphasize:

```text
MOD_CONTEXT
MOD_SAFETY
MOD_ASSET
MOD_DATA_CAPTURE
MOD_FLOW
```

If a defect is detected:

```text
MOD_CRITICALITY
      |
      v
MOD_ACTIONS
```

can become relevant dynamically according to workflow configuration.

---

# Capability Composition

FlowField treats an activity as a composition of capabilities.

```text
Activity A
   |
   +-- CONTEXT
   +-- SAFETY
   +-- DATA_CAPTURE
   `-- FLOW

Activity B
   |
   +-- CONTEXT
   +-- ASSET
   +-- DATA_CAPTURE
   +-- CRITICALITY
   +-- ACTIONS
   `-- FLOW

Activity C
   |
   +-- CONTEXT
   +-- SAFETY
   +-- ASSET
   +-- DATA_CAPTURE
   +-- CRITICALITY
   +-- DOCUMENTATION
   +-- ACTIONS
   `-- FLOW
```

The platform remains the same.

The operational configuration changes.

---

# Why Capability Boundaries Matter

Without clear boundaries, field platforms tend to become collections of special cases.

For example:

```text
inspection_app_v1
inspection_app_v2
railway_app
energy_app
construction_app
maintenance_app
```

FlowField takes a different approach:

```text
              Shared Platform
                    |
                    v
          Reusable Capabilities
                    |
                    v
         Configured Workflows
                    |
       +------------+------------+
       |            |            |
       v            v            v
     Energy     Construction   Railway
```

This reduces duplication and makes orchestration reusable across operational domains.

---

# Configuration, Not Duplication

A module can behave differently depending on the activity.

For example, `MOD_DATA_CAPTURE` could require:

```text
Activity A
- 2 photographs
- 1 checklist
```

while another activity could require:

```text
Activity B
- 5 photographs
- temperature measurement
- voltage measurement
- technical note
- annotated image
```

The capability remains the same.

Its configuration changes.

---

# Operational Independence

Each module should remain independently understandable.

Conceptually, every capability answers a different question:

| Module | Question |
|---|---|
| `MOD_CONTEXT` | Who, what and where? |
| `MOD_SAFETY` | Is execution safe and compliant? |
| `MOD_ASSET` | What asset is being operated on? |
| `MOD_DATA_CAPTURE` | What was observed or measured? |
| `MOD_CRITICALITY` | Is there a problem and how severe is it? |
| `MOD_DOCUMENTATION` | What formal records are required? |
| `MOD_ACTIONS` | What must be done about the issue? |
| `MOD_FLOW` | What is allowed to happen next? |

---

# Public Module Scope

This document describes the public capability model of FlowField 3.0.

It intentionally does not expose:

- complete task catalogs
- internal event identifiers
- proprietary rule definitions
- complete validation schemas
- orchestration policies
- customer-specific configurations
- implementation details

Those elements belong to the internal and commercial layers of the platform.

---

# Summary

FlowField 3.0 uses eight capability modules:

```text
CONTEXT
SAFETY
ASSET
DATA_CAPTURE
CRITICALITY
DOCUMENTATION
ACTIONS
FLOW
```

Together they separate:

```text
Context
Safety
Operational Objects
Evidence
Analysis
Formal Records
Remediation
Process Control
```

This separation allows FlowField to create complex operational workflows while keeping the field experience simple.

**The activity changes.  
The capabilities remain reusable.  
The orchestration adapts.**
