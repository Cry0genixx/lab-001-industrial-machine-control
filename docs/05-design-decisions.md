# LAB-001 — Design Decisions

**Document:** 05 — Design Decisions
**Project:** LAB-001 — Industrial Machine Control Core
**Version:** v0.1
**Status:** Draft
**Last updated:** 2026-09-05

---

# 1. Purpose

This document records important engineering and software-design decisions for LAB-001.

The purpose is to make major implementation choices explicit and reviewable.

Each decision should document:

* context,
* considered alternatives,
* selected approach,
* rationale,
* consequences,
* and future review conditions.

This document is not intended to capture every small programming choice.

It focuses on decisions that materially affect:

* architecture,
* maintainability,
* testability,
* reuse,
* and future system integration.

---

# 2. Decision Format

Each major decision uses the following structure:

```text
ADR-XXX — Decision Title

Status:
Accepted / Proposed / Superseded

Context:
Why is a decision required?

Options:
Which realistic alternatives were considered?

Decision:
What was selected?

Rationale:
Why was it selected?

Consequences:
What benefits and limitations follow?

Review:
When should the decision be reconsidered?
```

---

# 3. ADR-001 — Siemens TIA Portal as Primary Platform

**Status:** Accepted

## Context

LAB-001 requires an industrial PLC environment capable of supporting:

* structured programming,
* Function Blocks,
* symbolic data,
* simulation,
* diagnostics,
* and later integration with HMI and industrial communication.

Available platforms include:

* Siemens TIA Portal,
* CODESYS,
* Mitsubishi GX Works.

## Options

### Option A — Siemens TIA Portal V20

Advantages:

* industrially relevant platform,
* S7-PLCSIM available,
* suitable for future HMI integration,
* suitable for future Profinet and OPC UA work,
* aligns with later portfolio projects.

### Option B — CODESYS

Advantages:

* strong IEC 61131-3 environment,
* highly suitable for Structured Text,
* vendor-independent learning value.

### Option C — Mitsubishi GX Works3

Advantages:

* relevant industrial platform,
* strong future cross-platform comparison potential.

## Decision

The primary LAB-001 implementation shall use:

**Siemens TIA Portal V20 + S7-PLCSIM V20**

CODESYS may later be used for a cross-platform reimplementation.

## Rationale

The Siemens environment provides the strongest immediate path from:

PLC core
→ HMI
→ Profinet
→ SCADA
→ robot integration
→ flagship project.

## Consequences

The project may contain some Siemens-specific implementation details.

To reduce vendor lock-in, the documented architecture shall distinguish between:

* general control principles,
* and Siemens-specific implementation.

## Review

Revisit during LAB-010 or if simulation limitations prevent meaningful verification.

---

# 4. ADR-002 — Simulated S7-1500-Class Controller

**Status:** Proposed

## Context

LAB-001 requires a representative PLC target for TIA Portal and PLCSIM.

The exact CPU is not itself an important portfolio outcome, but the project should use a realistic modern Siemens controller configuration.

## Options

* S7-1200 family
* S7-1500 family
* more specialized simulated controller

## Decision

Use a **representative S7-1500-class CPU supported by the installed simulation environment**, unless a technical limitation makes another CPU more practical.

A mid-range CPU is preferred over selecting an unnecessarily high-end controller.

## Rationale

The objective is software architecture rather than demonstrating maximum controller hardware capability.

S7-1500 provides an appropriate environment for:

* structured symbolic programming,
* reusable FBs,
* structured data,
* diagnostics,
* and later portfolio extensions.

## Consequences

Exact hardware specifications shall not be treated as a project achievement.

The chosen CPU shall be documented in the implementation notes.

## Review

Finalize when creating the TIA Portal project.

---

# 5. ADR-003 — SCL / Structured Text as Primary Architectural Language

**Status:** Accepted

## Context

LAB-001 must demonstrate:

* state machines,
* enumerated logic,
* structured sequencing,
* reusable software,
* fault management.

TIA Portal allows multiple IEC 61131-3 programming representations.

## Options

### Ladder Logic

Strengths:

* widely used in industrial maintenance environments,
* intuitive for basic Boolean control,
* highly readable for some device logic.

Weaknesses:

* larger state machines and structured logic may become visually fragmented.

### SCL / Structured Text

Strengths:

* concise representation of state machines,
* strong symbolic structure,
* suitable for CASE logic,
* easy to review in source form,
* scales well for reusable architecture.

### Mixed Language

Use SCL for supervisory architecture and LAD where it improves readability.

## Decision

**SCL shall be the primary language for machine-state management, sequence control and structured logic.**

LAD may be used selectively where it provides clearer representation of simple Boolean or device-level logic.

## Rationale

The project is intended to demonstrate control-software architecture rather than only platform-specific graphical programming.

SCL provides clearer evidence for:

* explicit states,
* transitions,
* reusable structures,
* and software design.

## Consequences

The portfolio will emphasize IEC 61131-3 software architecture while still acknowledging that real industrial environments commonly use mixed-language projects.

## Review

Reassess after the first complete implementation.

---

# 6. ADR-004 — Machine State and Operating Mode as Separate Enumerated Types

**Status:** Accepted

## Context

Machine lifecycle and operator-selected mode represent different concepts.

Combining them creates state combinations such as:

```text
READY_MANUAL
READY_AUTO
FAULT_MANUAL
FAULT_AUTO
```

which scales poorly.

## Options

### Combined state/mode enumeration

One variable represents all combinations.

### Separate state and mode variables

Example:

```text
eMachineState = READY
eOperatingMode = MANUAL
```

## Decision

Use separate symbolic types.

Conceptually:

```text
E_MachineState
E_OperatingMode
```

## Rationale

This prevents combinatorial state growth and makes behaviour easier to understand.

## Consequences

The software must explicitly define which mode/state combinations are valid.

This is considered beneficial because those rules become visible instead of implicit.

---

# 7. ADR-005 — Symbolic State Representation

**Status:** Accepted

## Context

Machine states and sequence steps must be understandable in:

* code,
* diagnostics,
* documentation,
* testing.

## Options

### Raw integers

Example:

```text
0
10
20
30
```

### Symbolic constants

### Enumerated data types

## Decision

Use symbolic enumerated representation where supported cleanly by the selected TIA implementation.

Example concept:

```text
TYPE E_MachineState :
(
    DISABLED,
    INITIALIZING,
    READY,
    RUNNING,
    STOPPING,
    FAULT
);
END_TYPE
```

Actual Siemens syntax shall follow TIA Portal requirements.

## Rationale

Symbolic representation improves readability and reduces undocumented numeric meaning.

## Consequences

Online diagnostics and exported source may be easier to understand.

---

# 8. ADR-006 — Explicit Automatic Sequence Enumeration

**Status:** Accepted

## Context

The automatic sequence requires clear progression and diagnostic visibility.

## Options

* distributed Boolean step bits,
* integer step numbers,
* enumerated sequence steps,
* nested state machine.

## Decision

Use one explicitly identifiable automatic-sequence step variable.

Conceptual steps:

```text
SEQ_IDLE
SEQ_VERIFY_PART
SEQ_EXTEND
SEQ_PROCESS
SEQ_RETRACT
SEQ_COMPLETE
```

## Rationale

A single active step simplifies:

* troubleshooting,
* verification,
* state visibility,
* invalid-state detection.

## Consequences

Each transition shall be explicit.

The design should avoid creating large amounts of hidden transition logic outside the sequence controller.

---

# 9. ADR-007 — Function Block for Machine Supervision

**Status:** Accepted

## Context

High-level machine state and supervisory behaviour require persistent internal state.

## Options

* implement directly in OB1,
* use FC,
* use FB with instance DB.

## Decision

Implement machine supervision as a dedicated **Function Block**.

Suggested name:

```text
FB_MachineSupervisor
```

## Responsibilities

* machine state,
* transition evaluation,
* mode authorization,
* start acceptance,
* stop response,
* fault-state transition,
* recovery transition.

## Rationale

An FB naturally supports persistent state and clear ownership.

## Consequences

Machine supervisory state becomes encapsulated rather than distributed through OB1.

---

# 10. ADR-008 — Dedicated Automatic Sequence Function Block

**Status:** Accepted

## Decision

Implement automatic sequencing separately from machine supervision.

Suggested name:

```text
FB_AutoSequence
```

## Responsibilities

* active sequence step,
* process requests,
* step transitions,
* process completion,
* cycle-complete indication.

## Rationale

Machine lifecycle and process sequence are different abstraction levels.

The machine can be:

```text
RUNNING
```

while the sequence is:

```text
SEQ_EXTEND
```

This distinction should remain explicit.

---

# 11. ADR-009 — Reusable Device Function Block

**Status:** Accepted

## Context

LAB-001 requires at least one actuator with:

* command,
* feedback,
* timeout,
* interlocks,
* fault detection.

## Decision

Implement a reusable two-position actuator block.

Suggested name:

```text
FB_Actuator2Pos
```

## Intended Responsibilities

* Extend request evaluation,
* Retract request evaluation,
* mutually exclusive commands,
* feedback interpretation,
* movement detection,
* timeout monitoring,
* feedback-conflict detection,
* availability status.

## Rationale

This creates genuine reusable software evidence rather than embedding device behaviour directly into sequence logic.

## Consequences

The device block shall remain generic enough to support later use in LAB-003.

---

# 12. ADR-010 — Device Block Owns Final Logical Commands

**Status:** Accepted

## Context

Commands may originate from:

* initialization,
* Manual mode,
* automatic sequence,
* stopping logic.

Multiple blocks directly writing the same output creates ambiguous ownership.

## Decision

Only the device-control layer shall generate final logical actuator commands:

```text
xCmdExtend
xCmdRetract
```

Other software components provide requests.

## Rationale

This creates one deterministic command path.

## Consequences

A command arbitration mechanism is required before or inside device control.

---

# 13. ADR-011 — Request-Based Device Interface

**Status:** Accepted

## Decision

Device commands shall be represented as requests.

Examples:

```text
xReqExtendAuto
xReqRetractAuto

xReqExtendManual
xReqRetractManual

xReqRetractInit
xReqRetractStop
```

These may later be consolidated into a cleaner command structure.

## Rationale

Request-based control exposes command origin and avoids hidden writes.

## Consequences

The design must avoid unnecessary growth in individual Boolean command variables.

A command structure or requested-device-state abstraction may later be preferable.

---

# 14. ADR-012 — Explicit Command Arbitration

**Status:** Accepted

## Context

Multiple valid subsystems may need temporary authority over a device.

## Decision

Command priority shall be intentional and documented.

Initial priority concept:

```text
1. FAULT / DISABLED override
2. Initialization / recovery
3. Controlled stopping
4. Automatic sequence
5. Manual operation
```

## Important Clarification

This ordering does not mean that Automatic operation is intrinsically "more important" than Manual operation.

Under normal conditions, state/mode logic ensures that Manual and Automatic requests are mutually exclusive.

Priority exists primarily to handle exceptional machine conditions deterministically.

## Review

Verify during implementation that priority logic does not create surprising operator behaviour.

---

# 15. ADR-013 — Fault Manager as Dedicated Function Block

**Status:** Accepted

## Decision

Implement centralized fault registration and reset management.

Suggested name:

```text
FB_FaultManager
```

## Responsibilities

* receive fault-condition inputs,
* latch appropriate faults,
* generate blocking-fault summary,
* expose fault identifier,
* evaluate reset conditions,
* clear recoverable faults.

## Not Responsible For

The Fault Manager shall not directly assign the overall machine state.

## Rationale

Fault state detection and machine lifecycle remain separate concerns.

---

# 16. ADR-014 — Blocking Faults Only in v1.0

**Status:** Accepted

## Decision

LAB-001 v1.0 shall implement only blocking faults.

Warnings and informational alarms are deferred primarily to LAB-002.

## Rationale

This keeps fault-management scope focused on:

* machine response,
* latching,
* reset,
* recovery.

## Consequences

The design should not make future severity levels unnecessarily difficult to add.

---

# 17. ADR-015 — Fault Representation

**Status:** Proposed

## Context

Several implementation methods are possible:

* independent Boolean bits,
* integer fault code,
* array/list of fault structures,
* bitfield + active fault code.

## Decision

For LAB-001, use a **small structured fault set**, likely consisting of:

* individual fault-condition flags,
* overall blocking-fault flag,
* active/primary fault identifier.

Example concept:

```text
xFaultExtendTimeout
xFaultRetractTimeout
xFaultFeedbackConflict
xFaultInitialization

xAnyBlockingFault
eActiveFault
```

## Rationale

A full industrial alarm database would add unnecessary complexity for LAB-001.

A single generic `Fault := TRUE` would provide too little diagnostic evidence.

## Consequences

Multiple simultaneous faults may require a defined priority for the displayed primary fault.

The individual fault bits shall remain available even if only one primary identifier is shown.

---

# 18. ADR-016 — Fault Reset Requires Condition Clearance

**Status:** Accepted

## Decision

Reset shall only clear a latched fault when its underlying condition is no longer active.

Conceptually:

```text
IF xResetRequest AND NOT xFaultCauseActive THEN
    ClearFault();
END_IF;
```

## Rationale

An unconditional reset would hide active faults and weaken recovery logic.

## Consequences

Each latched fault must retain enough information to determine whether reset is permitted.

---

# 19. ADR-017 — Fault Recovery Through INITIALIZING

**Status:** Accepted

## Decision

Use:

```text
FAULT
 ↓
INITIALIZING
 ↓
READY
```

rather than direct:

```text
FAULT → READY
```

## Rationale

This ensures the machine is returned to a known condition before readiness is declared.

## Consequences

Initialization logic must support use during both:

* initial startup,
* post-fault recovery.

---

# 20. ADR-018 — S7-PLCSIM Simulation Layer Separate from Control Logic

**Status:** Accepted

## Decision

Simulated process behaviour shall be implemented as a separate block or clearly separated program area.

Suggested block:

```text
FB_ProcessSimulation
```

## Responsibilities

* create actuator feedback based on output commands,
* simulate movement delay,
* provide workpiece condition,
* inject failures,
* simulate contradictory feedback.

## Rationale

The production-control architecture should not know whether feedback originates from:

* simulation,
* or future physical I/O.

## Consequences

A clean interface is required between control and simulation.

---

# 21. ADR-019 — Control/Simulation Signal Boundary

**Status:** Accepted

## Decision

The controller shall consume process feedback through normal logical input variables.

Example:

```text
xActuatorExtendedFb
xActuatorRetractedFb
xWorkpiecePresent
```

The simulation block may generate those values during simulation.

The control logic shall not depend directly on:

```text
xSimFailExtend
```

or other fault-injection controls.

## Rationale

Simulation settings belong to the test environment, not production control logic.

---

# 22. ADR-020 — Structured Data Using UDTs Where Beneficial

**Status:** Accepted

## Context

Large numbers of unrelated global tags reduce readability.

## Decision

Use structured PLC data where it improves organization.

Candidate UDTs:

```text
UDT_MachineCommands
UDT_MachineStatus
UDT_Permissives
UDT_ActuatorCommand
UDT_ActuatorStatus
UDT_FaultStatus
UDT_SimulationControl
```

## Rationale

Structured data improves:

* block interfaces,
* diagnostics,
* future HMI integration,
* future OPC UA integration.

## Consequences

Avoid creating UDTs purely for the sake of abstraction.

Small groups of signals may remain simple parameters where that is clearer.

---

# 23. ADR-021 — Instance DBs for Stateful Function Blocks

**Status:** Accepted

## Decision

Stateful FBs shall use their normal Siemens instance data mechanisms.

Expected stateful blocks include:

* Machine Supervisor,
* Auto Sequence,
* Fault Manager,
* Actuator,
* Process Simulation.

## Rationale

Persistent internal block state should remain with the block that owns the behaviour.

---

# 24. ADR-022 — OB1 Used Primarily for Coordination

**Status:** Accepted

## Decision

OB1 shall remain relatively simple.

Its primary purpose is to call the major control functions in intentional order.

Avoid implementing significant process behaviour directly in OB1.

## Rationale

A clean cyclic coordinator makes overall execution flow visible.

---

# 25. ADR-023 — Intentional PLC Scan Order

**Status:** Accepted

## Context

The PLC executes cyclically.

Execution order may affect when changes become visible.

## Decision

Block order shall be documented and deliberate.

Initial proposed order:

```text
1. Command/Input acquisition
2. Device feedback evaluation
3. Fault-condition evaluation
4. Fault Manager
5. Permissive evaluation
6. Machine Supervisor
7. Automatic Sequence
8. Manual command processing
9. Device command arbitration/control
10. Process Simulation
11. Diagnostics
```

## Important Consequence

Because simulation follows control outputs within the cycle, simulated feedback may naturally become available on a subsequent PLC scan.

This is acceptable unless verification demonstrates undesirable behaviour.

## Review

Test for:

* one-scan delays,
* state-entry behaviour,
* timeout initialization,
* fault timing.

---

# 26. ADR-024 — Rising-Edge Handling for Operator Commands

**Status:** Accepted

## Context

Start and Reset should normally behave as discrete operator actions.

If an input remains TRUE for several PLC scans, it should not repeatedly trigger an action.

## Decision

Use rising-edge detection for at least:

* Start request,
* Reset request,
* mode-selection commands where appropriate,
* Enable request where appropriate.

## Rationale

This models momentary operator commands and avoids repeated event execution.

## Consequences

Edge-detection state shall have clearly defined ownership.

Use Siemens-provided edge mechanisms or an equivalent explicit implementation.

---

# 27. ADR-025 — Stop Request Behaviour

**Status:** Proposed

## Context

Stop can be represented as:

* momentary edge-triggered request,
* maintained stop request,
* or both depending on machine philosophy.

## Decision

For LAB-001, treat Stop as a **request that remains effective until the machine has left RUNNING**.

The exact PLC implementation may use either:

* maintained input behaviour,
* or internal latching of a rising-edge request.

## Rationale

A one-scan stop pulse must not be lost before controlled stop logic acts on it.

---

# 28. ADR-026 — Manual Commands as Maintained Requests

**Status:** Accepted

## Decision

Manual Extend / Retract shall initially behave as maintained operator requests rather than one-shot commands.

Example:

```text
Button held → request active
Button released → request removed
```

## Rationale

This is intuitive for basic jog-style manual operation and useful for verifying interlocks.

## Consequences

Future HMI implementation must clearly communicate this behaviour.

---

# 29. ADR-027 — Explicit State Entry Detection

**Status:** Accepted

## Context

Some actions should occur once when entering a state.

Examples:

* reset sequence step,
* initialize timer,
* clear temporary completion flag.

## Decision

Provide explicit state-entry recognition.

Conceptually:

```text
xStateEntry := eStateCurrent <> eStatePrevious;
```

with previous state updated deterministically.

## Rationale

State-entry actions are clearer than ad hoc Boolean first-scan flags.

---

# 30. ADR-028 — Explicit Sequence Entry Handling

**Status:** Accepted

The same principle shall be applied where useful to automatic sequence steps.

Potential variables:

```text
eStepCurrent
eStepPrevious
xStepEntry
```

## Rationale

This simplifies:

* timer reset,
* one-time commands,
* logging,
* testing.

---

# 31. ADR-029 — Timer Ownership Near the Behaviour Being Timed

**Status:** Accepted

## Decision

Timeouts should generally be owned by the function responsible for the behaviour.

Examples:

**Actuator movement timeout**

belongs in:

```text
FB_Actuator2Pos
```

**Simulated process timer**

belongs in:

```text
FB_AutoSequence
```

or the process simulation function, depending on its purpose.

## Rationale

Timers scattered centrally create hidden dependencies.

---

# 32. ADR-030 — Configurable Timeout Parameters

**Status:** Accepted

## Decision

Engineering timeouts shall be configurable named parameters.

Example:

```text
tMovementTimeout := T#2s
```

rather than unexplained embedded timer values.

## Rationale

Supports:

* documentation,
* testability,
* later hardware replacement.

---

# 33. ADR-031 — Device Feedback State Interpretation

**Status:** Accepted

For the two-position actuator:

| Extended | Retracted | Interpretation         |
| -------: | --------: | ---------------------- |
|        0 |         1 | Retracted              |
|        1 |         0 | Extended               |
|        0 |         0 | Intermediate / Unknown |
|        1 |         1 | Invalid / Conflict     |

## Rationale

This provides a deterministic basis for:

* movement,
* initialization,
* diagnostics,
* fault detection.

---

# 34. ADR-032 — No Immediate Fault for Intermediate Feedback During Movement

**Status:** Accepted

## Decision

```text
Extended = FALSE
Retracted = FALSE
```

shall not automatically cause a blocking fault.

## Rationale

A real actuator may legitimately be between end positions during motion.

Fault occurs if:

* destination feedback fails to arrive before timeout,
* or process logic requires a known position and one cannot be established.

---

# 35. ADR-033 — Feedback Conflict Is Immediately Abnormal

**Status:** Accepted

## Decision

```text
Extended = TRUE
Retracted = TRUE
```

shall be treated as an invalid sensor combination.

## Rationale

The two limit conditions are defined as mutually exclusive.

---

# 36. ADR-034 — Workpiece Presence Changes Meaning by Process Context

**Status:** Accepted

## Decision

Before automatic start:

```text
WorkpiecePresent = FALSE
```

shall inhibit start.

During a committed active process:

unexpected loss of workpiece presence may generate a blocking process fault.

## Rationale

The same signal may represent:

* a precondition before operation,
* and abnormal process behaviour after operation begins.

This demonstrates context-aware permissive logic.

---

# 37. ADR-035 — Single-Cycle Automatic Operation in v1.0

**Status:** Accepted

## Decision

One accepted Start request shall execute one complete cycle.

After completion:

```text
RUNNING → READY
```

A new Start request is required.

## Rationale

Single-cycle behaviour simplifies:

* verification,
* debugging,
* state analysis,
* fault injection.

Continuous production can later be added intentionally.

---

# 38. ADR-036 — Controlled Stop Returns Device to Known Position

**Status:** Accepted

## Decision

A Stop request during RUNNING shall initiate:

```text
RUNNING → STOPPING
```

and the system shall attempt to return the representative actuator to its known retracted position.

## Rationale

This provides a meaningful controlled-stop process rather than simply disabling sequence execution.

## Consequences

A failed return may escalate:

```text
STOPPING → FAULT
```

---

# 39. ADR-037 — Fault-State Device Commands Inactive

**Status:** Accepted for LAB-001

## Decision

For the representative actuator:

```text
ExtendCommand = FALSE
RetractCommand = FALSE
```

during FAULT.

## Rationale

This is a simple deterministic logical response suitable for the generic simulated device.

## Limitation

This is **not** claimed to be a universally safe physical actuator state.

A real machine requires device- and safety-specific analysis.

---

# 40. ADR-038 — Selected Mode Retained Through Fault

**Status:** Accepted

## Decision

Mode shall remain unchanged during a blocking fault.

Example:

```text
State = FAULT
Mode = AUTOMATIC
```

may be valid.

## Rationale

The selected mode remains useful operating-context information.

## Consequence

State and permissive logic must prevent this retained mode from authorizing operation during FAULT.

---

# 41. ADR-039 — New Start Required After Recovery

**Status:** Accepted

## Decision

Fault recovery shall never automatically resume the interrupted cycle in LAB-001.

After:

```text
FAULT → INITIALIZING → READY
```

the system requires a new Start request.

## Rationale

This creates deterministic and easily verifiable recovery behaviour.

---

# 42. ADR-040 — Diagnostics Exposed as Structured Status

**Status:** Accepted

## Decision

Key internal control values shall be deliberately exposed through a diagnostic/status structure rather than relying solely on online monitoring of internal block variables.

Candidate values:

* machine state,
* operating mode,
* sequence step,
* start permissive,
* individual permissives,
* active fault,
* actuator command,
* actuator feedback,
* actuator availability,
* cycle counter.

## Rationale

This provides a clean future interface for:

* LAB-002 HMI,
* LAB-004 SCADA,
* verification.

---

# 43. ADR-041 — Cycle Counter Increments on Verified Completion Event

**Status:** Accepted

## Decision

Cycle count shall increment only when a complete cycle reaches the formal successful completion condition.

It shall not increment:

* on Start,
* on entry to RUNNING,
* after interrupted cycles,
* after fault reset.

## Rationale

Production metrics should represent actual successful process completion.

---

# 44. ADR-042 — No Artificial Complexity for Portfolio Appearance

**Status:** Accepted

## Decision

LAB-001 shall not introduce:

* excessive object-oriented abstractions,
* unnecessary library layers,
* complex messaging systems,
* arbitrary patterns

solely to appear advanced.

## Rationale

Engineering quality is demonstrated by:

* clarity,
* appropriateness,
* determinism,
* testing,
* maintainability.

Complexity without functional benefit weakens rather than strengthens the portfolio.

---

# 45. ADR-043 — Cross-Platform Concepts Documented Separately from Siemens Details

**Status:** Accepted

## Decision

Documentation shall distinguish general concepts such as:

* state machines,
* device ownership,
* fault management,
* interlocks,

from Siemens-specific elements such as:

* OBs,
* FBs,
* DBs,
* TIA-specific syntax.

## Rationale

This helps demonstrate transferable automation-engineering knowledge.

---

# 46. Proposed TIA Project Structure

The initial TIA Portal program structure should resemble:

```text
Program blocks
│
├── OB1_Main
│
├── FB_MachineSupervisor
│
├── FB_AutoSequence
│
├── FB_FaultManager
│
├── FB_Actuator2Pos
│
├── FB_ProcessSimulation
│
├── FC_Permissives          [optional]
│
└── FC_Diagnostics          [optional]
│
├── DB_MachineSupervisor
├── DB_AutoSequence
├── DB_FaultManager
├── DB_Actuator01
└── DB_ProcessSimulation
```

Depending on final implementation, some functionality may remain inside related FBs rather than being separated into FCs.

---

# 47. Proposed PLC Data Types

Candidate PLC data types:

```text
E_MachineState
E_OperatingMode
E_AutoStep
E_FaultCode
```

Candidate structured types:

```text
UDT_OperatorCommands
UDT_MachineStatus
UDT_Permissives
UDT_ActuatorRequests
UDT_ActuatorStatus
UDT_FaultStatus
UDT_SimulationControl
UDT_SimulationStatus
```

The actual number of UDTs shall be minimized to those that provide real organizational value.

---

# 48. Proposed Machine State Enumeration

Conceptually:

```text
DISABLED
INITIALIZING
READY
RUNNING
STOPPING
FAULT
```

If explicit numerical values are useful for diagnostic stability, they may use a spaced scheme such as:

```text
DISABLED      = 0
INITIALIZING  = 10
READY         = 20
RUNNING       = 30
STOPPING      = 40
FAULT         = 90
```

The symbolic identifier remains the primary meaning.

---

# 49. Proposed Auto Step Enumeration

Conceptually:

```text
SEQ_IDLE          = 0
SEQ_VERIFY_PART   = 10
SEQ_EXTEND        = 20
SEQ_PROCESS       = 30
SEQ_RETRACT       = 40
SEQ_COMPLETE      = 50
```

Spaced values may make future step insertion easier, but the software shall not depend on arithmetic relationships between step numbers.

---

# 50. Proposed Fault Codes

Initial fault identifiers may include:

```text
FAULT_NONE
FAULT_INIT_TIMEOUT
FAULT_EXTEND_TIMEOUT
FAULT_RETRACT_TIMEOUT
FAULT_FEEDBACK_CONFLICT
FAULT_WORKPIECE_LOST
```

Exact mapping shall be finalized before implementation.

---

# 51. Preliminary Signal Naming Convention

Boolean variables:

```text
x...
```

Examples:

```text
xStartRequest
xWorkpiecePresent
xFaultActive
```

Time values:

```text
t...
```

Example:

```text
tMovementTimeout
```

Enumerations:

```text
e...
```

Examples:

```text
eMachineState
eOperatingMode
eAutoStep
eActiveFault
```

Counters / unsigned values:

Use descriptive prefix appropriate to the actual data type.

Example:

```text
udiCycleCount
```

Naming may be adjusted to maintain consistency with Siemens conventions and readability.

---

# 52. Implementation Quality Criteria

Before LAB-001 moves from architecture to verification, the PLC implementation should satisfy the following review questions:

1. Is machine state owned by one logical function?
2. Is operating mode owned by one logical function?
3. Does the automatic sequence avoid direct final output writes?
4. Does each actuator have one final logical command owner?
5. Are simulation controls absent from production decision logic?
6. Are fault causes visible independently?
7. Is reset conditional?
8. Are operator events handled predictably across PLC scans?
9. Are timers owned by the behaviour they supervise?
10. Are state and sequence transitions readable in source code?
11. Can diagnostic data be accessed without inspecting dozens of unrelated tags?
12. Could the simulation layer later be replaced with physical I/O without major control redesign?

---

# 53. Decisions Requiring Validation During Implementation

The following decisions remain subject to practical verification:

## VAL-001

Exact S7-1500 CPU selection.

## VAL-002

Whether all supervisory logic belongs in one Machine Supervisor FB or whether State and Mode managers should become separate FBs.

## VAL-003

Whether command arbitration belongs inside the actuator FB or in a separate machine-level command function.

## VAL-004

Exact UDT boundaries.

## VAL-005

Best representation of multiple simultaneous fault conditions.

## VAL-006

Exact PLC scan order after testing.

## VAL-007

Whether the initialization behaviour can be kept generic without making the actuator FB process-specific.

These items should be resolved through implementation experience rather than theoretical abstraction alone.

---

# 54. Current Implementation Baseline

The initial LAB-001 PLC implementation shall therefore use the following conceptual structure:

```text
                    OB1
                     │
          ┌──────────┼───────────┐
          │          │           │
          ▼          ▼           ▼
      Commands    Feedback    Simulation
          │          │
          └──────┬───┘
                 ▼
          Fault Detection
                 │
                 ▼
          FB_FaultManager
                 │
                 ▼
       Permissive Evaluation
                 │
                 ▼
      FB_MachineSupervisor
           │             │
           ▼             ▼
   FB_AutoSequence   Manual Requests
           │             │
           └──────┬──────┘
                  ▼
          Command Arbitration
                  │
                  ▼
          FB_Actuator2Pos
                  │
                  ▼
             Commands
                  │
                  ▼
        FB_ProcessSimulation
                  │
                  ▼
             Feedback
```

The implementation shall prioritize clarity and deterministic ownership over unnecessary abstraction.

Any major departure from this baseline should be documented as a new or revised design decision.
