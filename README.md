# LAB-001 — Industrial Machine Control Core

**Status:** Planning\
**Version:** v0.1\
**Project Type:** Technical Engineering Laboratory

## Overview

LAB-001 develops a reusable control-software architecture for a generic industrial machine.

The project focuses on how machine states, operating modes, permissives, interlocks, faults and recovery can be handled in a predictable, maintainable and verifiable manner.

The primary objective is not to simulate one specific production machine, but to develop and verify a generic control core that can later be reused and extended in more complex automation projects.

The project will initially be implemented using Siemens TIA Portal V20 and S7-PLCSIM V20.

---

## Engineering Problem

Industrial machine-control software often grows progressively as new functions, sensors, actuators and fault conditions are added.

Without a defined architecture, this can result in control logic that is:

* difficult to understand,
* difficult to modify,
* difficult to troubleshoot,
* difficult to test,
* and difficult to integrate with other equipment.

This project therefore addresses the following engineering question:

> How can a reusable industrial machine-control architecture be designed so that machine states, operating modes, permissives, interlocks, faults and recovery are handled in a predictable, maintainable and verifiable manner?

---

## Primary Objectives

The project shall develop and verify a machine-control architecture capable of handling:

* system initialization,
* machine readiness,
* Manual operating mode,
* Automatic operating mode,
* controlled stopping,
* fault detection,
* fault state handling,
* fault reset and recovery,
* permissives,
* interlocks,
* basic diagnostics,
* predictable state transitions.

The architecture should also provide a suitable foundation for later integration with:

* HMI,
* pneumatic actuators,
* industrial robots,
* SCADA,
* industrial communication,
* and larger production systems.

---

## Competencies Demonstrated

The project is intended to create portfolio evidence in:

* PLC programming
* Siemens TIA Portal
* IEC 61131-3 programming
* Structured Text
* Function Blocks
* state-machine architecture
* sequential control
* machine operating modes
* permissives and interlocks
* fault handling
* diagnostics
* requirements engineering
* software architecture
* verification and validation
* technical documentation
* Git-based version control

---

## Initial System Concept

The machine-control core will initially use a state-based control model.

Provisional states:

```text
OFF
 │
 ↓
INITIALIZING
 │
 ↓
READY
 │
 ├───────────────┐
 ↓               │
AUTOMATIC        │
 │               │
 ├── PAUSED      │
 │               │
 ↓               │
STOPPING ────────┘

Any applicable state
        │
        ↓
      FAULT
```

This state model is preliminary.

The final state architecture shall be derived from documented functional requirements and may therefore change during project development.

---

## Initial Functional Scope

### Included

The initial project scope includes:

* machine-state architecture,
* Manual and Automatic operating modes,
* startup and initialization logic,
* permissive management,
* interlock management,
* stop behaviour,
* fault handling,
* reset logic,
* basic diagnostic information,
* PLC simulation,
* functional verification,
* fault-injection testing.

### Excluded from the Initial Scope

The initial version does not attempt to implement:

* physical machine hardware,
* functional safety PLC logic,
* certified machine-safety functions,
* robot integration,
* SCADA,
* advanced HMI,
* motion control,
* servo control,
* process-specific regulation,
* complete electrical hardware.

These may be introduced in later portfolio projects.

---

## Toolchain

Primary development environment:

* Siemens TIA Portal V20
* S7-PLCSIM V20

Supporting tools:

* Git
* GitHub
* Obsidian
* Visual Studio Code where useful

CODESYS may later be used to reimplement selected parts of the architecture to evaluate platform-independent IEC 61131-3 concepts.

---

## Engineering Workflow

The project follows the portfolio Engineering Project Standard.

Development sequence:

```text
Problem Definition
        ↓
Requirements
        ↓
System Architecture
        ↓
State Model
        ↓
Control Software Design
        ↓
Implementation
        ↓
Simulation
        ↓
Fault Injection
        ↓
Verification
        ↓
Lessons Learned
```

Implementation shall not be considered complete until defined requirements have been evaluated through verification.

---

## Planned Project Structure

```text
lab-001-industrial-machine-control/
│
├── README.md
│
├── docs/
│   ├── 01-problem-definition.md
│   ├── 02-requirements.md
│   ├── 03-system-architecture.md
│   ├── 04-functional-description.md
│   ├── 05-design-decisions.md
│   ├── 06-verification-plan.md
│   ├── 07-test-results.md
│   └── 08-lessons-learned.md
│
├── plc/
│
├── diagrams/
│   ├── state-machine/
│   └── architecture/
│
├── tests/
│
├── media/
│   └── screenshots/
│
└── references/
```

Folders shall only be populated when they contain meaningful project material.

---

## Planned Engineering Artefacts

The project is expected to produce:

* formal problem definition,
* requirements specification,
* machine-state diagram,
* state-transition table,
* software architecture diagram,
* PLC program,
* reusable function blocks,
* fault-management concept,
* test specification,
* fault-injection test cases,
* verification report,
* lessons-learned document.

---

## Verification Strategy

Verification will initially be performed using S7-PLCSIM.

The project shall test both normal and abnormal behaviour.

Representative verification scenarios include:

* successful initialization,
* valid mode selection,
* valid state transitions,
* rejected invalid transitions,
* rejected automatic start without permissives,
* controlled stop,
* detected process fault,
* transition to Fault state,
* successful reset,
* rejected reset while fault conditions remain active,
* recovery to a known operating state.

Each important requirement shall later be linked to one or more verification cases.

---

## Project Limitations

The initial project is a software and control-architecture laboratory.

It does not represent a complete production machine.

PLC behaviour may be simulated, but simulation does not validate:

* real actuator dynamics,
* physical electrical behaviour,
* real machine-safety performance,
* network timing under production conditions,
* or hardware failure behaviour.

These limitations shall be stated explicitly in the final project documentation.

---

## Success Criteria

LAB-001 will be considered successfully verified when:

* the control architecture is clearly documented,
* important behaviour is defined through requirements,
* state transitions behave predictably,
* invalid operating conditions are rejected,
* representative faults are handled correctly,
* recovery behaviour is verified,
* project documentation is internally consistent,
* and a technical reviewer can understand the system without verbal explanation.

---

## Future Reuse

LAB-001 is intended to become the control-software foundation for later portfolio projects, including:

* LAB-002 — HMI & Alarm Architecture
* LAB-003 — Electro-Pneumatic Machine Module
* LAB-004 — Industrial Communication & SCADA
* LAB-005 — PLC ↔ Robot Integration
* FLAGSHIP-001 — Automated CNC Manufacturing Cell

The long-term objective is to reuse proven control concepts rather than redesign the basic machine architecture independently for every project.
