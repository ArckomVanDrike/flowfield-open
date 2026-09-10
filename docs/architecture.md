# FlowField 3.0 Architecture

## Field Operations Orchestration Architecture

FlowField 3.0 is designed as a modular orchestration platform for operational activities performed in the field.

Its architecture separates three concerns that are often mixed together in traditional field applications:

**what the organization wants to perform**,  
**how the operational process must behave**,  
and **what the technician needs to see at each moment**.

This separation allows FlowField to support different industries and operational scenarios without rebuilding the application around every new workflow.

---

## Architectural Model

At a high level, FlowField follows this sequence:

```text
Activity Definition
        |
        v
Capability Selection
        |
        v
Module Configuration
        |
        v
Flow Orchestration
        |
        v
Task Execution
        |
        v
Events / Evidence / Actions
        |
        v
Reporting & Audit
```

The activity defines the operational context.

The required capability modules determine what the process needs to do.

The orchestration layer determines how those capabilities interact during execution.

---

## Core Architectural Principle

FlowField separates the **logical orchestration layer** from the **operational user experience**.

```text
+--------------------------------------------------+
|                 Operational UX                   |
|                                                  |
|  Web Workspace              Mobile Field Client |
|  Definition & Governance    Guided Execution    |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|              Orchestration Layer                 |
|                                                  |
|  Module Configuration                            |
|  Validation Rules                                |
|  Workflow State                                  |
|  Dependencies                                    |
|  Conditions                                      |
|  Events                                          |
|  Escalation                                      |
|  Closure Control                                 |
+-------------------------+------------------------+
                          |
                          v
+--------------------------------------------------+
|                Capability Layer                  |
|                                                  |
| CONTEXT | SAFETY | ASSET | DATA CAPTURE         |
| CRITICALITY | DOCUMENTATION | ACTIONS | FLOW    |
+--------------------------------------------------+
```

The user interface may simplify multiple operational steps into a single screen.

The underlying logical objects remain independent.

This prevents user experience decisions from corrupting the workflow model.

---

## Capability-Based Architecture

FlowField does not model industries as independent hardcoded applications.

Instead, it uses reusable **capability blocks**.

| Capability | Responsibility |
|---|---|
| `MOD_CONTEXT` | Activity context, operator identity, site, geolocation, authorization and access |
| `MOD_SAFETY` | Safety controls, PPE, operational risk, work permits and stop-work conditions |
| `MOD_ASSET` | Asset identification, verification, installation, removal, replacement and inventory |
| `MOD_DATA_CAPTURE` | Operational evidence including media, measurements, notes and technical checklists |
| `MOD_CRITICALITY` | Detection, classification, severity and impact of anomalies or non-conformities |
| `MOD_DOCUMENTATION` | Formal documentation, certificates, compliance material and document lifecycle |
| `MOD_ACTIONS` | Corrective actions, remediation, responsibility, priorities, SLA and action state |
| `MOD_FLOW` | Workflow progression, conditions, dependencies, branching, blocking, escalation and closure |

Each activity type is created by combining the capabilities required for that particular operational context.

---

## Module Boundaries

Clear module boundaries are fundamental to the FlowField architecture.

For example:

```text
DATA_CAPTURE
     |
     | produces evidence
     v
CRITICALITY
     |
     | identifies a condition
     v
ACTIONS
     |
     | manages remediation
     v
FLOW
     |
     | controls process progression
     v
Workflow State
```

`MOD_DATA_CAPTURE` collects evidence.

It does not decide whether the evidence represents an anomaly.

`MOD_CRITICALITY` evaluates and classifies the anomaly.

It does not manage the remediation lifecycle.

`MOD_ACTIONS` manages the corrective activity.

`MOD_FLOW` decides how those events affect the workflow.

This separation keeps individual capabilities reusable and prevents business logic from becoming tightly coupled.

---

## Logical Layer

The logical layer is the actual orchestration architecture.

It contains:

- capability modules
- module configuration
- deterministic validation
- workflow state
- process rules
- standardized events
- task dependencies
- escalation rules
- closure conditions

Business process logic belongs here.

It must not depend on how a particular screen happens to be rendered.

---

## Operational Experience

The operational interface is intentionally simpler than the architecture behind it.

A technician may experience a single screen containing:

```text
Take photo
+ Record measurement
+ Add note
+ Report issue
```

Internally, FlowField may persist those operations through different capability domains.

```text
Photo / Measurement / Note
          |
          v
   MOD_DATA_CAPTURE

Reported Issue
          |
          v
   MOD_CRITICALITY

Required Remediation
          |
          v
      MOD_ACTIONS
```

The UX can be fluid.

The architecture remains structured.

---

## Data Persistence Principle

Visual simplification must never collapse logically separate data objects.

If a field screen combines evidence collection, anomaly classification and remediation into one interaction, the underlying records must still preserve their logical ownership.

```text
Operational Screen
       |
       +--> Evidence Payload
       |       MOD_DATA_CAPTURE
       |
       +--> Criticality Payload
       |       MOD_CRITICALITY
       |
       `--> Action Payload
               MOD_ACTIONS
```

This provides stronger traceability, reporting and auditability.

---

## Activity Lifecycle

A FlowField activity begins with operational context rather than with a static form.

```text
Activity Type
     |
     v
Required Capabilities
     |
     v
Configured Modules
     |
     v
Orchestration Runtime
     |
     v
Generated / Enabled Tasks
     |
     v
Field Execution
     |
     v
Controlled Closure
```

Tasks originate from activated capabilities.

They are then ordered and conditioned by the orchestration layer.

---

## Flow Orchestration

`MOD_FLOW` is the process coordination capability.

It does not collect operational evidence or replace the functional modules.

Its responsibility is to govern progression.

Conceptually:

```text
              +----------------+
              | Current State  |
              +-------+--------+
                      |
                      v
              +----------------+
              | Evaluate Event |
              +-------+--------+
                      |
             +--------+--------+
             |                 |
             v                 v
      Condition Met      Condition Failed
             |                 |
             v                 v
       Next Task          Block / Branch
             |                 |
             +--------+--------+
                      |
                      v
              Updated State
```

The orchestration layer can respond to operational conditions by:

```text
continue
block
branch
request evidence
require approval
open remediation
escalate
allow closure
deny closure
```

The exact internal rule implementation is outside the scope of this public repository.

---

## Event-Driven Interaction

FlowField modules interact through structured operational events rather than direct cross-module coupling.

Conceptually:

```text
Capability Module
       |
       | event
       v
Event Layer
       |
       v
MOD_FLOW
       |
       +--> state transition
       +--> validation
       +--> dependency update
       +--> escalation
       +--> action request
       `--> closure decision
```

This event-driven approach allows capabilities to evolve independently while maintaining predictable workflow behaviour.

The complete internal event catalog is not published in this repository.

---

## Dependency Model

Some dependencies are structural.

For example:

```text
MOD_CONTEXT
     |
     +----> establishes operational context
     |
     v
MOD_SAFETY
     |
     +----> may authorize or block execution
     |
     v
Operational Capabilities
```

Other dependencies are event-driven:

```text
MOD_ASSET
     |
     +--> anomaly detected
              |
              v
       MOD_CRITICALITY
              |
              +--> remediation required
                         |
                         v
                    MOD_ACTIONS
                         |
                         +--> completion state
                                  |
                                  v
                              MOD_FLOW
```

The orchestration engine coordinates these relationships without moving domain responsibilities between modules.

---

## Deterministic Validation

Critical operational controls are designed to remain deterministic.

Examples include:

```text
required evidence present?
authorization valid?
safety requirement satisfied?
mandatory task completed?
document complete?
blocking action still open?
closure conditions satisfied?
```

These decisions should be reproducible and auditable.

Intelligent assistance may help interpret or configure an activity, but it does not need to replace deterministic operational governance.

---

## Web Architecture Role

The web workspace is primarily responsible for:

```text
Define
Configure
Review
Assign
Govern
Monitor
```

A guided configuration process can collect the information necessary to determine the structure of an activity without requiring the user to manually design a workflow graph.

The system then converts that configuration into an executable operational model.

---

## Mobile Architecture Role

The mobile client is primarily responsible for:

```text
Receive
Execute
Capture
Confirm
Report
Complete
```

The field operator should not need to understand the orchestration architecture.

The client receives the current process state and presents only the relevant operational actions.

```text
Orchestrator
     |
     | permitted actions
     v
Mobile Client
     |
     | technician input
     v
Structured Event
     |
     v
Orchestrator
```

This creates a controlled feedback loop between field execution and workflow state.

---

## Example Cross-Module Scenario

Consider an infrastructure inspection.

```text
1. Operator checks in
        |
        v
   MOD_CONTEXT

2. Safety requirements validated
        |
        v
   MOD_SAFETY

3. Target equipment identified
        |
        v
   MOD_ASSET

4. Technician records measurements and photos
        |
        v
   MOD_DATA_CAPTURE

5. Measurement indicates a possible anomaly
        |
        v
   MOD_CRITICALITY

6. Remediation is required
        |
        v
   MOD_ACTIONS

7. Open corrective action prevents closure
        |
        v
   MOD_FLOW
```

To the technician this can appear as one continuous guided inspection.

Architecturally, each responsibility remains isolated.

---

## AI Integration Architecture

Artificial intelligence is intentionally separated from the deterministic orchestration core.

```text
+--------------------------------------------------+
|                 FlowField Core                   |
|                                                  |
| Workflow State                                   |
| Validation                                       |
| Dependencies                                     |
| Events                                           |
| Blocking Rules                                   |
| Audit                                            |
+------------------------+-------------------------+
                         |
                         v
+--------------------------------------------------+
|                AI Adapter Layer                  |
|                                                  |
| Interpretation                                   |
| Assistance                                       |
| Classification Support                           |
| Natural Language Interaction                     |
+------------------------+-------------------------+
                         |
             +-----------+-----------+
             |                       |
             v                       v
        Edge Provider           Cloud Provider
             |                       |
             v                       v
            Outy               External / Private
```

The AI layer can help interpret operational intent and assist configuration, but workflow authority remains within the FlowField orchestration architecture.

This creates an important architectural property:

**AI providers can be replaced without replacing the workflow engine.**

### Edge-First Reference Integration

The reference architecture uses **Outy**, developed within the CashOut ecosystem, as an edge AI integration.

In this model, FlowField can connect to intelligence deployed close to the operational environment through a dedicated adapter layer.

Outy is therefore a reference implementation of the AI interface rather than a mandatory dependency of FlowField.

### Provider-Agnostic Adapter

The same integration boundary can be connected to alternative AI infrastructure.

Conceptually:

```text
                 FlowField
                     |
                     v
                 AI Adapter
                     |
        +------------+------------+
        |            |            |
        v            v            v
       Outy       Cloud API    Private Model
       Edge        Provider      Endpoint
```

Potential integrations may include customer-managed infrastructure, hosted inference services, enterprise gateways or external cloud AI providers.

Provider-specific behavior belongs behind the adapter boundary.

The workflow engine should not contain provider-specific orchestration logic.

### Deterministic Core Principle

FlowField follows a hybrid architecture:

```text
AI
 |
 +--> interpret
 +--> assist
 +--> suggest
 +--> classify where appropriate

Deterministic Core
 |
 +--> validate
 +--> authorize
 +--> transition state
 +--> enforce requirements
 +--> block
 +--> allow closure
 +--> preserve audit history
```

This preserves operational predictability while still allowing AI capabilities to evolve independently.

---


## Visual Intelligence and VLM Integration

FlowField treats visual intelligence as a specialized AI capability connected to operational evidence.

Images captured in the field belong first to `MOD_DATA_CAPTURE`.

A Vision-Language Model can then analyze those images through the AI adapter layer.

```text
                   Technician
                       |
                       v
                 Capture Image
                       |
                       v
               MOD_DATA_CAPTURE
                       |
                       v
              Evidence Repository
                       |
                       v
                 VLM Adapter
                       |
          +------------+------------+
          |                         |
          v                         v
     Cloud Vision               Edge / Local
       Provider                      VLM
          |                         |
          +------------+------------+
                       |
                       v
                AI Observation
                       |
                       v
             Validation Boundary
                       |
             +---------+---------+
             |                   |
             v                   v
      MOD_CRITICALITY         MOD_FLOW
```

### Current Reference Integration

The current reference implementation can route field photographs to a GPT-based cloud vision API.

This is an implementation choice rather than an architectural dependency.

The same VLM boundary can be connected to alternative providers or to local inference infrastructure.

### Edge-Capable by Design

Visual inference may require significantly more compute than conversational or classification workloads.

For environments with sufficient hardware, FlowField can place the VLM close to the operational environment.

```text
Field Device
    |
    v
FlowField
    |
    v
Local / Edge VLM
```

For environments where suitable edge hardware is not available, the same interface can use cloud inference:

```text
Field Device
    |
    v
FlowField
    |
    v
AI Adapter
    |
    v
Cloud VLM
```

The workflow architecture remains unchanged.

### Evidence Ownership

The VLM does not replace the evidence object.

The original image remains the primary captured evidence.

The AI result is a derived interpretation associated with that evidence.

Conceptually:

```text
Original Photo
     |
     +--> immutable / retained evidence
     |
     `--> VLM Analysis
              |
              v
       Derived Observation
```

This distinction is important for auditability.

### Decision Boundary

A visual model may identify a possible defect or unsafe condition, but the analysis itself does not automatically need to become an authoritative operational decision.

```text
VLM Analysis
     |
     v
Possible Anomaly
     |
     v
Validation
     |
     +--> accepted --> MOD_CRITICALITY
     |
     `--> rejected / review required
```

Deployment policy can determine whether validation is:

- deterministic
- operator-confirmed
- supervisor-confirmed
- rule-assisted
- automatically accepted for low-risk scenarios

Critical workflow authority remains outside the VLM provider.

### Provider Independence

The visual AI adapter is designed so that provider-specific implementation stays outside the orchestration core.

```text
                    VLM Interface
                         |
       +-----------------+-----------------+
       |                 |                 |
       v                 v                 v
   GPT Vision       Private VLM        Edge VLM
   Cloud API         Endpoint          Runtime
```

This means organizations can select the visual inference environment based on:

- connectivity
- latency
- cost
- hardware availability
- privacy requirements
- enterprise policy
- deployment constraints

FlowField remains responsible for orchestration regardless of which VLM performs the visual analysis.

---

## Reporting and Audit

Reporting is a consequence of structured execution.

```text
Modules
   |
   +--> structured outputs
   |
Events
   |
   +--> operational history
   |
Actions
   |
   +--> remediation lifecycle
   |
Workflow
   |
   +--> state history
   |
   v
Reporting / Dashboard / Audit
```

Because information remains associated with its capability domain, reporting can distinguish between:

```text
what was observed
what was classified
what was required
what was performed
what was approved
what blocked the process
how the activity was closed
```

---

## Architectural Invariants

FlowField 3.0 follows several invariants:

1. Capability modules remain logically independent.
2. UI simplification does not merge domain data.
3. Process logic does not live in the presentation layer.
4. Tasks originate from configured capabilities.
5. Workflow progression is governed by the orchestration layer.
6. Cross-module communication is event-driven.
7. Critical validation can remain deterministic.
8. Closure is a workflow state, not an independent business capability.
9. Evidence, anomalies and corrective actions remain separate concepts.
10. Field execution must remain simpler than the system operating behind it.

These principles provide the foundation for extending FlowField without turning each new operational scenario into a new application.

---

## Extensibility

The same architecture can support different field environments by changing configuration rather than redesigning the platform.

```text
                    FlowField Core
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
      Utilities      Construction   Infrastructure
          |              |              |
          v              v              v
     Configured      Configured      Configured
     Capabilities    Capabilities    Capabilities
```

New industries can introduce specialized configuration and domain data while preserving the same orchestration principles.

---


## Organizational Governance Layer

FlowField's process architecture is complemented by an organizational governance layer.

```text
Direction
    |
Project Manager
    |
Responsible
    |
Technician
```

These roles represent different operational responsibility boundaries rather than different workflow engines.

The orchestration model remains shared.

```text
Role + Scope
     |
     v
Authorization
     |
     v
MOD_FLOW
     |
     v
Permitted Operation
```

The standard public model distinguishes:

- organization-level governance
- project and activity management
- operational supervision
- field execution

Exact permissions remain deployment-specific.

See [Roles and Governance](roles-and-governance.md).

---

## Public Architecture Scope

This repository describes the architectural model necessary to understand FlowField and evaluate its applicability.

The public documentation intentionally does not expose:

- proprietary orchestration rules
- complete internal configuration schemas
- full event catalogs
- customer-specific workflows
- private integrations
- production infrastructure details

Those components belong to commercial implementations of FlowField.

---

## Architecture Summary

FlowField 3.0 separates:

```text
WHAT needs to happen
        |
        v
Capabilities

HOW the process progresses
        |
        v
Orchestration

WHAT the technician sees
        |
        v
Operational UX
```

That separation is the foundation of the platform.

**FlowField is not a configurable checklist.**

It is an orchestration architecture designed to transform operational requirements into controlled, traceable and executable field workflows.
