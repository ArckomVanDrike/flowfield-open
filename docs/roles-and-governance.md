# FlowField 3.0 Roles and Governance

## Organizational Control for Field Operations

FlowField 3.0 separates two complementary architectures:

```text
PROCESS ARCHITECTURE
Capabilities -> Tasks -> Events -> MOD_FLOW

ORGANIZATIONAL ARCHITECTURE
Direction -> Project Manager -> Responsible -> Technician
```

The first controls how operational work progresses.

The second controls who can define, supervise, validate and execute that work.

Together they provide operational governance.

---

## Organizational Model

The public FlowField governance model is based on four operational levels:

```text
                     DIRECTION
                         |
                         v
                 PROJECT MANAGER
                         |
              +----------+----------+
              |                     |
              v                     v
        RESPONSIBLE            RESPONSIBLE
        Team / Area            Team / Area
              |                     |
         +----+----+           +----+----+
         |         |           |         |
         v         v           v         v
    TECHNICIAN TECHNICIAN TECHNICIAN TECHNICIAN
```

Each level has a different relationship with the workflow.

---

# Direction

## Purpose

Direction represents organization-level governance.

Its primary perspective is not an individual field task but the operational system as a whole.

Typical responsibilities can include:

- portfolio visibility
- global operational status
- KPI monitoring
- criticality visibility
- SLA performance
- audit oversight
- compliance oversight
- organizational reporting
- escalation visibility
- governance policies

Conceptually:

```text
Projects
   +
Teams
   +
Activities
   +
Criticalities
   +
Actions
   +
SLA
   +
Audit
   |
   v
DIRECTION
```

Direction observes and governs.

It does not normally execute field tasks.

---

# Project Manager

## Purpose

The Project Manager governs activities and operational programs.

This role sits between organizational strategy and operational execution.

Typical responsibilities can include:

- defining operational activities
- configuring Activity Types
- using the guided activity configuration process
- reviewing generated workflows
- configuring operational requirements
- planning activities
- assigning operational responsibility
- monitoring activity progress
- reviewing exceptions
- coordinating escalation
- reviewing project-level reporting

Conceptually:

```text
Operational Requirement
        |
        v
Project Manager
        |
        v
Activity Definition
        |
        v
FlowField Configuration
        |
        v
Executable Workflow
```

The Project Manager works primarily through the web governance environment.

---

# Responsible

## Purpose

The Responsible role represents operational supervision.

Depending on the organization, this role may correspond to:

- team leader
- area responsible
- site supervisor
- operational supervisor
- service coordinator
- maintenance responsible

The exact job title can vary.

The architectural responsibility remains similar.

Typical responsibilities can include:

- receiving assigned activities
- coordinating a team or operational area
- assigning work to technicians
- monitoring execution
- reviewing operational exceptions
- handling escalations
- reviewing criticalities
- managing corrective actions
- validating completion where required
- following SLA and deadlines

Conceptually:

```text
Project / Activity
       |
       v
   Responsible
       |
   +---+---+
   |       |
   v       v
Tech A   Tech B
   |       |
   +---+---+
       |
       v
Operational Results
       |
       v
Review / Escalation / Validation
```

The Responsible is the bridge between workflow governance and field execution.

---

# Technician

## Purpose

The Technician or Field Operator executes operational work.

The technician does not design the workflow.

The technician follows the workflow generated and governed by FlowField.

Typical capabilities include:

- view assigned activities
- start permitted work
- perform check-in
- verify access
- complete safety steps
- identify assets
- capture photographs
- capture video or audio
- record measurements
- complete technical checklists
- report anomalies
- associate operational evidence
- perform assigned corrective steps
- complete permitted closure actions

Conceptually:

```text
Assigned Activity
       |
       v
Next Permitted Task
       |
       v
Technician Executes
       |
       v
Structured Output
       |
       v
FlowField
       |
       v
Next Permitted Task
```

The technician interacts with the operational process.

FlowField manages the process logic.

---

## Technician Experience

The mobile interface should expose only what is operationally relevant.

```text
What do I need to do now?
           |
           v
      Mobile Client
           |
           v
   Current Permitted Task
```

The technician should not need to understand:

- workflow graphs
- orchestration rules
- module configuration
- escalation logic
- internal state machines
- event routing
- provider integrations

That complexity belongs to the platform.

---

# Role and Scope

Role alone is not sufficient for enterprise governance.

FlowField combines role with operational scope.

Conceptually:

```text
Identity
   +
Role
   +
Organization
   +
Project
   +
Team / Area
   +
Activity Scope
   |
   v
Authorization
```

A Responsible in one operational area should not automatically gain authority over every project in the organization.

A technician should receive only activities within the scope assigned to them.

---

## Authorization Model

FlowField can evaluate authorization using contextual information such as:

```text
operator
role
organization
project
team
site
activity type
workflow state
```

Conceptually:

```text
User Request
     |
     v
Identity + Role + Scope
     |
     v
Authorization Check
     |
  +--+--+
  |     |
Allow Deny
  |     |
  v     v
Action  Controlled Response
```

Authorization is part of operational control, not merely interface visibility.

---

# Governance Meets Orchestration

The organizational model and the workflow model intersect at the authorization boundary.

```text
            ORGANIZATIONAL ROLE
                    |
                    v
             Operational Scope
                    |
                    v
              Authorization
                    |
                    v
                 MOD_FLOW
                    |
                    v
           Permitted Operations
```

This means FlowField controls both:

```text
WHAT can happen next
          +
WHO is allowed to do it
```

---

# Example Permission Boundaries

The following matrix represents the public governance model.

Exact permissions remain deployment-specific.

| Capability | Direction | Project Manager | Responsible | Technician |
|---|:---:|:---:|:---:|:---:|
| Organization overview | Yes | Scoped | Scoped | No |
| Portfolio / KPI visibility | Yes | Scoped | Scoped | No |
| Define Activity Types | Governance-dependent | Yes | Limited / optional | No |
| Configure workflow requirements | Governance-dependent | Yes | Limited / optional | No |
| Assign operational responsibility | Optional | Yes | Scoped | No |
| Assign technicians | No / optional | Optional | Yes | No |
| Execute field tasks | No | No | Optional | Yes |
| Capture field evidence | No | No | Optional | Yes |
| Report anomalies | Review | Review | Review / create | Yes |
| Manage corrective actions | Oversight | Oversight | Yes | Assigned actions |
| Review escalations | Yes | Yes | Yes | Raise only |
| Validate operational closure | Policy-dependent | Policy-dependent | Yes where required | Only when permitted |
| Audit visibility | Yes | Scoped | Scoped | Own activity |

This is a governance model, not a hardcoded RBAC table.

Customer deployments can adapt the exact permission boundaries.

---

# Separation of Experiences

FlowField therefore supports different experiences according to responsibility.

```text
DIRECTION
Executive / Governance View

PROJECT MANAGER
Planning / Configuration / Control View

RESPONSIBLE
Operational Supervision View

TECHNICIAN
Guided Field Execution View
```

These are not four copies of the same dashboard.

Each role should receive information appropriate to its operational responsibility.

---

# Direction Experience

A Direction-level interface may focus on:

```text
Portfolio status
Projects
Activities
Criticalities
Open actions
SLA
Compliance
Trends
Audit
```

The objective is governance and visibility.

---

# Project Manager Experience

A Project Manager interface may focus on:

```text
Activity Types
Workflow configuration
Projects
Planning
Assignments
Activity status
Exceptions
Reporting
```

The objective is configuration and operational control.

---

# Responsible Experience

A Responsible interface may focus on:

```text
Assigned operations
Technician workload
Open activities
Blocked workflows
Criticalities
Corrective actions
Escalations
Validation queue
```

The objective is supervision and resolution.

---

# Technician Experience

A Technician interface may focus on:

```text
My Activities
Priority
Current Task
Required Evidence
Field Inputs
Issues
Next Step
Completion
```

The objective is execution.

---

# Assignment Model

FlowField distinguishes between activity ownership and task execution.

Conceptually:

```text
Project Manager
      |
      v
Operational Responsibility
      |
      v
Responsible
      |
      v
Technician Assignment
      |
      v
Field Execution
```

Corrective actions can also have their own assignee and lifecycle.

```text
Criticality
    |
    v
Corrective Action
    |
    v
Assigned Responsible / Team / Provider
    |
    v
Resolution
    |
    v
Validation
```

---

# Escalation Model

Operational events may move upward through the organizational structure.

```text
Technician
    |
    | issue / block
    v
Responsible
    |
    | unresolved / critical
    v
Project Manager
    |
    | strategic / severe
    v
Direction
```

Not every event must reach every level.

Escalation depends on workflow policy, severity, SLA and organizational configuration.

---

# Corrective Action Governance

FlowField separates the person who detects a problem from the person responsible for resolving it.

```text
Technician
    |
    v
Detects / Reports
    |
    v
Criticality
    |
    v
MOD_ACTIONS
    |
    v
Assigned Responsible
    |
    v
Remediation
    |
    v
Validation
```

This provides explicit operational ownership.

---

# Closure Governance

Closure can involve more than one organizational level.

A simple workflow may allow the technician to complete the activity directly when all deterministic conditions are satisfied.

A regulated workflow may require additional validation.

```text
Technician completes work
          |
          v
Deterministic checks
          |
          v
Responsible validation
          |
          v
Closure
```

Another deployment may require:

```text
Technician
    |
Responsible
    |
Project Manager Approval
    |
CLOSED
```

The required approval chain is configuration-driven.

---

# AI and Organizational Authority

AI does not replace organizational responsibility.

Outy, cloud AI or a VLM may:

- interpret
- assist
- summarize
- suggest
- analyze evidence
- recommend classification

They do not implicitly inherit organizational authority.

```text
AI Suggestion
      |
      v
Role + Scope + Workflow Policy
      |
      v
Authorized Decision
```

This keeps AI capability separate from operational accountability.

---

# Visual AI Example

A technician captures an image.

```text
Technician
    |
    v
Photo
    |
    v
VLM Analysis
    |
    v
Possible Criticality
    |
    v
Technician / Responsible Validation
    |
    v
Workflow Decision
```

Depending on policy, a Responsible or another authorized role can review the AI-assisted observation before it affects the operational process.

---

# Audit and Accountability

Role-based activity contributes to the FlowField audit model.

A relevant operational record may preserve:

```text
who
role
scope
what action
which activity
when
result
workflow effect
```

This allows the system to distinguish between:

```text
who executed
who assigned
who reviewed
who validated
who approved
```

where required by the deployment.

---

# Organizational Flexibility

The four-role model describes the standard public FlowField structure:

```text
Direction
Project Manager
Responsible
Technician
```

Real organizations may use different titles.

For example:

```text
Direction
    =
Operations Director
Engineering Manager
Service Director

Project Manager
    =
PM
Program Manager
Maintenance Planner

Responsible
    =
Supervisor
Team Leader
Area Manager
Site Coordinator

Technician
    =
Field Technician
Inspector
Operator
Engineer
Contractor
```

The title can change.

The governance function remains.

---

# Two-Dimensional Architecture

FlowField can therefore be understood through two dimensions.

```text
HORIZONTAL: PROCESS

CONTEXT -> SAFETY -> ASSET -> DATA -> CRITICALITY -> ACTIONS
                            |
                            v
                          FLOW


VERTICAL: GOVERNANCE

DIRECTION
    |
PROJECT MANAGER
    |
RESPONSIBLE
    |
TECHNICIAN
```

The platform combines both.

```text
Process Control
      +
Organizational Control
      =
Governed Field Operations
```

---

# Public Scope

This document defines the public organizational governance model of FlowField 3.0.

It intentionally does not expose:

- production RBAC implementation
- customer-specific permission matrices
- identity provider configuration
- internal authorization rules
- customer organizational structures
- proprietary escalation policies
- security credentials or infrastructure

Those elements belong to deployment-specific and commercial implementations.

---

# Summary

FlowField 3.0 does not only orchestrate tasks.

It orchestrates field operations within an organizational structure.

```text
Direction
    |
Project Manager
    |
Responsible
    |
Technician
    |
Field Execution
```

At the same time:

```text
Capabilities
    |
Events
    |
MOD_FLOW
    |
Controlled Process
```

The two models meet through authorization, scope and workflow policy.

**FlowField controls both what happens next and who is allowed to make it happen.**
