# LAB-001 — Problem Definition

**Document:** 01 — Problem Definition\
**Project:** LAB-001 — Industrial Machine Control Core\
**Version:** v0.1\
**Status:** Draft\
**Last updated:** 2026-09-05

---

# 1. Context

Industrial machines typically require a control system that coordinates machine states, operator commands, sensors, actuators and abnormal conditions.

Even relatively simple machines may need to handle:

* startup and initialization,
* machine readiness,
* manual operation,
* automatic sequences,
* interlocks,
* permissives,
* controlled stopping,
* process faults,
* reset conditions,
* diagnostic information,
* and recovery after abnormal events.

As machine functionality grows, control software can become difficult to understand and maintain if these behaviours are added incrementally without a defined architecture.

This project investigates how a generic industrial machine-control core can be structured so that important machine behaviour remains predictable, testable and reusable.

---

# 2. Problem Statement

A PLC application can perform the required machine functions while still being poorly structured.

Typical problems may include:

* machine states being represented implicitly across unrelated logic,
* unclear transitions between operating conditions,
* duplicated actuator logic,
* inconsistent fault handling,
* insufficient distinction between permissives and interlocks,
* reset logic scattered throughout the application,
* difficult troubleshooting,
* limited testability,
* and poor scalability when new equipment is added.

Such systems may function during normal operation while becoming difficult to understand when abnormal conditions occur.

The engineering problem addressed by LAB-001 is therefore:

> **How can a reusable industrial machine-control architecture be designed so that machine states, operating modes, permissives, interlocks, faults and recovery are handled in a predictable, maintainable and verifiable manner?**

---

# 3. Project Intent

The project shall develop a generic control-software foundation suitable for a discrete industrial machine or machine module.

The architecture should demonstrate principles that can later be reused in applications such as:

* automated workholding,
* material handling,
* CNC machine tending,
* assembly equipment,
* inspection stations,
* robot cells,
* and other sequential manufacturing equipment.

The project is not intended to replicate one specific commercial machine.

Instead, it shall establish a reusable control framework that later portfolio projects can extend with real process functions.

---

# 4. Reference Machine Concept

To ensure that the control architecture remains testable, LAB-001 shall assume the existence of a simple generic industrial machine.

The reference machine represents a discrete manufacturing process with:

* operator start and stop commands,
* Manual and Automatic operating modes,
* a small number of simulated sensors,
* simulated actuators,
* machine permissives,
* process interlocks,
* and fault conditions.

The exact physical process is intentionally abstracted during the first revision.

A representative process may later include:

1. Detect workpiece.
2. Confirm machine ready.
3. Actuate a process device.
4. Confirm actuator feedback.
5. Perform a simulated process step.
6. Return the device to its safe or home position.
7. Mark the cycle complete.

This provides sufficient process behaviour to verify the control architecture without making LAB-001 dependent on a particular mechanical system.

---

# 5. Primary Engineering Objective

The primary objective is to design, implement and verify a structured PLC control architecture that provides deterministic behaviour during:

* startup,
* initialization,
* machine readiness,
* Manual operation,
* Automatic operation,
* normal stop,
* abnormal process conditions,
* fault handling,
* reset,
* and recovery.

The architecture should be understandable independently of the final physical machine implementation.

---

# 6. Secondary Objectives

The project should also:

1. Establish a reusable machine-state model.
2. Separate state management from device-level control where practical.
3. Define clear behaviour for Manual and Automatic modes.
4. Establish a consistent permissive philosophy.
5. Establish a consistent interlock philosophy.
6. Centralize or otherwise structure fault management.
7. Provide basic diagnostic information.
8. Enable deliberate fault-injection testing.
9. Demonstrate requirement-based verification.
10. Create control concepts reusable in later portfolio projects.

---

# 7. Stakeholders

Although LAB-001 is a simulated engineering project, the control architecture should consider typical industrial stakeholders.

## 7.1 Operator

Primary needs:

* understand whether the machine is ready,
* start and stop permitted operation,
* select permitted operating mode,
* understand why the machine cannot start,
* recognize active faults,
* initiate appropriate reset or recovery.

---

## 7.2 Maintenance Technician

Primary needs:

* understand current machine state,
* inspect relevant sensor and actuator status,
* identify active interlocks,
* identify fault causes,
* operate devices manually where permitted,
* recover the machine after faults.

---

## 7.3 Controls / Automation Engineer

Primary needs:

* understand software architecture,
* modify functionality safely,
* add devices or process states,
* trace machine behaviour,
* test changes,
* diagnose unexpected control behaviour.

---

## 7.4 Production / Process Engineering

Primary needs:

* predictable machine operation,
* understandable process-state information,
* reproducible cycle behaviour,
* appropriate response to process abnormalities.

---

## 7.5 Future Integrated Systems

Later projects may introduce:

* HMI,
* robot,
* SCADA,
* pneumatic systems,
* production-data systems.

LAB-001 should therefore avoid architectures that unnecessarily prevent later integration.

---

# 8. Functional Boundaries

LAB-001 focuses on the **machine-control software layer**.

Conceptual boundary:

```text
               Operator Commands
                      │
                      ↓
              ┌───────────────┐
              │ Machine       │
              │ Control Core  │
              └───────┬───────┘
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
        States      Devices      Faults
          │           │           │
          └───────────┼───────────┘
                      ↓
             Simulated Process
```

The project shall manage logical control behaviour, while physical process behaviour is simulated.

---

# 9. Included Scope

The initial LAB-001 scope includes the following.

## 9.1 Machine State Management

The controller shall manage clearly defined machine states.

Candidate states include:

* OFF
* INITIALIZING
* READY
* AUTOMATIC
* PAUSED
* STOPPING
* FAULT

The exact state model shall be finalized during requirements and architecture development.

---

## 9.2 Operating Modes

At minimum:

* Manual
* Automatic

The relationship between machine state and operating mode shall be explicitly defined.

---

## 9.3 Initialization

The controller shall establish whether the simulated machine is in a valid initial condition before declaring it ready for operation.

---

## 9.4 Permissives

The project shall demonstrate conditions that must be satisfied before specific operations are allowed.

Representative examples:

* no active blocking fault,
* machine initialized,
* required device available,
* required process condition present.

---

## 9.5 Interlocks

The project shall prevent logically invalid or conflicting commands.

Representative examples:

* actuator commands that conflict with current device state,
* automatic start during an invalid machine condition,
* incompatible simultaneous actions.

---

## 9.6 Manual Operation

Simulated devices shall be operable manually where appropriate while respecting defined interlocks.

---

## 9.7 Automatic Operation

A representative automatic sequence shall be implemented.

The sequence exists primarily to test the control architecture rather than reproduce a specific production process.

---

## 9.8 Stop Behaviour

The project shall distinguish between at least:

* normal completion,
* requested stop,
* fault-induced interruption.

More advanced stop categories may be introduced later if justified.

---

## 9.9 Fault Management

Representative faults shall be detected and managed.

Potential examples include:

* actuator timeout,
* conflicting sensor signals,
* failed initialization,
* invalid sequence condition,
* missing required process condition.

---

## 9.10 Reset and Recovery

The project shall define:

* which faults may be reset,
* which conditions must clear before reset,
* the state entered after reset,
* whether automatic operation may resume directly.

---

## 9.11 Diagnostics

The application shall provide sufficient internal status information to support verification and troubleshooting.

---

## 9.12 Simulation

Control behaviour shall be verified using S7-PLCSIM V20 where technically feasible.

---

# 10. Out of Scope

The following are deliberately excluded from LAB-001 v1.0 unless later added through a documented scope change.

## 10.1 Functional Safety

The project shall not claim implementation or validation of:

* emergency-stop safety functions,
* safety PLC logic,
* SIL,
* PL,
* Category ratings,
* certified safety architecture.

Logical simulation of a generic safety-permissive input may be used for control purposes, but this shall not be presented as a functional-safety implementation.

---

## 10.2 Physical Hardware

No physical PLC, sensors or actuators are required for the initial project.

---

## 10.3 Advanced HMI

A dedicated operator-interface architecture belongs primarily to LAB-002.

Minimal monitoring may be used during development where required.

---

## 10.4 Pneumatic Process Design

Detailed pneumatic behaviour belongs primarily to LAB-003.

---

## 10.5 Robot Integration

PLC-to-robot communication belongs primarily to LAB-005.

---

## 10.6 SCADA / Higher-Level Data Systems

OPC UA, Ignition, historian functionality and IT/OT integration belong primarily to LAB-004.

---

## 10.7 Motion Control

The following are excluded:

* servo-axis control,
* coordinated motion,
* interpolation,
* advanced drive control.

---

## 10.8 Complete Electrical Design

Full electrical cabinet and machine wiring design is outside the primary scope.

Representative I/O concepts may be documented.

---

# 11. Assumptions

The initial project is based on the following assumptions.

## A-001

The reference machine represents a discrete sequential manufacturing process.

## A-002

Sensors and actuators may be represented using simulated PLC signals.

## A-003

Physical response times may be approximated using simulated timers.

## A-004

The control-system architecture is evaluated primarily for logical behaviour rather than real-time physical performance.

## A-005

S7-PLCSIM provides sufficient functionality to verify the intended PLC behaviour for the initial project.

## A-006

No physical machine-safety validation is performed.

## A-007

The architecture is intended to be reusable, but universal applicability to every industrial machine is not assumed.

---

# 12. Constraints

## CON-001 — Development Platform

The primary implementation shall use Siemens TIA Portal V20.

---

## CON-002 — Verification Platform

The initial implementation shall be verified using S7-PLCSIM V20 where possible.

---

## CON-003 — Hardware Availability

The project shall not depend on physical automation hardware for completion.

---

## CON-004 — Project Scope

The project shall prioritize control architecture over process complexity.

---

## CON-005 — Documentation

Major control behaviour shall be documented sufficiently to allow external technical review.

---

## CON-006 — Portfolio Relevance

Implementation decisions should prioritize demonstrable engineering reasoning rather than unnecessary software complexity.

---

# 13. Key Engineering Questions

The project should answer the following questions.

## Q-001

What is the appropriate distinction between machine **state** and operating **mode**?

## Q-002

Which conditions should prevent entering Automatic operation?

## Q-003

How should permissives differ from interlocks?

## Q-004

How should faults affect the active machine state?

## Q-005

Should all faults force the same machine response?

## Q-006

What conditions must exist before a fault reset is accepted?

## Q-007

What state should follow successful fault recovery?

## Q-008

How should actuator commands be separated from sequence logic?

## Q-009

How should the controller expose information useful for diagnostics?

## Q-010

How can the architecture remain reusable without becoming excessively abstract?

These questions should be resolved through requirements, architecture decisions and testing rather than assumed prematurely.

---

# 14. Preliminary Success Criteria

The project shall be considered technically successful if the implemented control core can demonstrate:

1. Predictable startup behaviour.
2. Explicit machine-state management.
3. Controlled operating-mode selection.
4. Valid automatic sequencing.
5. Prevention of automatic operation when required permissives are absent.
6. Prevention of conflicting commands through interlocks.
7. Detection of representative process faults.
8. Predictable transition into a fault condition.
9. Rejection of invalid fault reset.
10. Successful recovery when valid reset conditions are restored.
11. Clear diagnostic visibility into important machine conditions.
12. Traceability between defined requirements and verification results.

---

# 15. Portfolio Success Criteria

The project shall additionally demonstrate:

* structured problem definition,
* requirements engineering,
* control-system architecture,
* technical decision-making,
* PLC implementation,
* abnormal-condition handling,
* verification methodology,
* technical documentation,
* and honest description of simulation limitations.

The project should provide evidence of engineering methodology rather than merely demonstrating that a PLC program executes.

---

# 16. Relationship to Future Projects

LAB-001 is intended to establish the control foundation for subsequent work.

```text
LAB-001
Machine Control Core
     │
     ├──► LAB-002
     │    HMI & Alarm Architecture
     │
     ├──► LAB-003
     │    Electro-Pneumatic Module
     │
     ├──► LAB-004
     │    Industrial Communication & SCADA
     │
     ├──► LAB-005
     │    PLC ↔ Robot Integration
     │
     └──► FLAGSHIP-001
          Automated CNC Manufacturing Cell
```

Future projects may reuse design principles, software structures or documented interfaces developed during LAB-001.

Reusable elements shall nevertheless be reviewed against the requirements of each new application rather than assumed to be universally valid.

---

# 17. Current Project Boundary

LAB-001 v0.1 can therefore be summarized as:

> **Design and verify the logical control core of a generic sequential industrial machine using Siemens PLC simulation, with emphasis on architecture, states, modes, interlocks, permissives, faults and recovery rather than physical process complexity.**

This definition shall form the basis for the initial requirements specification.
