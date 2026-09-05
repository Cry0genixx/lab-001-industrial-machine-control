# LAB-001 — System Architecture

**Document:** 03 — System Architecture
**Project:** LAB-001 — Industrial Machine Control Core
**Version:** v0.1
**Status:** Draft
**Last updated:** 2026-09-05

---

# 1. Purpose

This document defines the initial control-system architecture for LAB-001.

The architecture translates the project requirements into a structured PLC software design.

The primary architectural goals are:

* predictable machine behaviour,
* clear separation of responsibilities,
* deterministic ownership of outputs,
* maintainability,
* diagnosability,
* testability,
* and reuse in future automation projects.

The architecture is intentionally more important than process complexity in LAB-001.

---

# 2. Architectural Principles

The following principles govern the initial design.

## AP-001 — Separate Machine State from Operating Mode

Machine state and operating mode shall be represented as separate concepts.

**Machine State** answers:

> What overall condition is the machine currently in?

**Operating Mode** answers:

> How is the machine currently permitted to be operated?

Example:

```text id="jd4m4k"
Machine State: READY
Operating Mode: MANUAL
```

or:

```text id="kw1jg6"
Machine State: READY
Operating Mode: AUTOMATIC
```

The mode does not replace the state.

---

## AP-002 — One Owner per Final Output

Each simulated actuator output shall have one clearly defined software owner.

Sequence logic and Manual-control logic may request actions, but shall not independently write directly to the final actuator output.

This prevents:

* conflicting writes,
* hidden command priority,
* scan-order dependency,
* and difficult troubleshooting.

---

## AP-003 — Sequence Logic Requests Actions

The automatic sequence shall express **process intent**, not directly control every physical output.

Example:

Prefer:

```text id="w4gh51"
Sequence requests ClampExtend
```

rather than:

```text id="l01x9t"
Sequence directly writes Output_Q0_2 := TRUE
```

The device layer determines whether the requested action is permitted.

---

## AP-004 — Interlocks Remain Effective in Manual Operation

Manual mode shall not automatically bypass actuator interlocks.

Manual operation allows direct operator requests to individual devices, but valid interlocks shall continue to govern final commands.

---

## AP-005 — Fault Detection and Fault Response Are Distinct

The architecture shall distinguish between:

1. detecting an abnormal condition,
2. registering / latching the fault,
3. determining the required machine response,
4. resetting the fault.

This prevents device-specific detection logic from becoming responsible for overall machine-state control.

---

## AP-006 — Simulation Shall Be Replaceable

Simulated process behaviour shall be separated from machine-control logic.

Future replacement of simulated feedback with physical I/O should therefore require minimal restructuring of the control architecture.

---

# 3. High-Level Architecture

The initial architecture is divided into the following functional layers:

```text id="8svw7g"
┌─────────────────────────────────────────────┐
│              COMMAND INTERFACE              │
│ Start / Stop / Reset / Mode Requests        │
└─────────────────────┬───────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────┐
│            MACHINE SUPERVISION              │
│                                             │
│ Machine State                               │
│ Operating Mode                              │
│ Permissives                                 │
│ Fault Response                              │
└───────────────┬─────────────────┬───────────┘
                │                 │
                ▼                 ▼
┌───────────────────────┐ ┌───────────────────┐
│  AUTOMATIC SEQUENCE   │ │   MANUAL CONTROL  │
│ Process step requests │ │ Operator requests │
└────────────┬──────────┘ └─────────┬─────────┘
             │                      │
             └──────────┬───────────┘
                        ▼
              ┌────────────────────┐
              │   DEVICE CONTROL   │
              │                    │
              │ Interlocks         │
              │ Command arbitration│
              │ Feedback handling  │
              │ Timeout detection  │
              └─────────┬──────────┘
                        │
                        ▼
              ┌────────────────────┐
              │    FINAL OUTPUTS   │
              └─────────┬──────────┘
                        │
                        ▼
              ┌────────────────────┐
              │ PROCESS SIMULATION │
              └─────────┬──────────┘
                        │
                        ▼
                  Simulated Inputs
```

A separate fault-management function receives abnormal-condition information from relevant subsystems and feeds machine supervision.

---

# 4. Machine State Model

## 4.1 Architectural Decision: Remove `OFF`

The preliminary project definition included an `OFF` state.

For LAB-001 v1.0, `OFF` shall be replaced with:

**`DISABLED`**

### Rationale

When the PLC application is actively executing, the physical machine is not truly powered off.

Using `OFF` would therefore blur the distinction between:

* actual electrical power-off,
* control-system disabled condition,
* and software initialization.

`DISABLED` more accurately represents:

> The controller is running, but machine operation is not currently enabled.

True electrical power-off remains outside the observable PLC application state.

---

# 5. Proposed Machine States

The initial architecture shall use:

```text id="t2ehb8"
DISABLED
INITIALIZING
READY
RUNNING
STOPPING
FAULT
```

`PAUSED` is excluded from LAB-001 v1.0 unless implementation experience demonstrates a clear need.

---

# 6. State Definitions

## `DISABLED`

The PLC application is executing, but production operation is disabled.

Characteristics:

* no automatic sequence active,
* controlled outputs commanded to defined inactive states,
* initialization may be requested,
* diagnostic information remains available.

---

## `INITIALIZING`

The controller is validating or establishing the conditions required for machine readiness.

Typical activities:

* evaluate device feedback,
* confirm initial positions,
* clear transient internal sequence states,
* evaluate machine readiness,
* detect initialization failure.

The state shall terminate deterministically in either:

* `READY`,
* or `FAULT`.

---

## `READY`

The machine is initialized and available for permitted operator actions.

Characteristics:

* no automatic process is currently running,
* Manual or Automatic mode may be selected according to mode rules,
* automatic cycle may start only when Automatic mode and all start permissives are valid.

---

## `RUNNING`

An automatic production sequence is active.

Entry requires:

* Automatic mode,
* valid machine readiness,
* required start permissives,
* accepted Start request.

The active automatic sequence step shall be externally observable.

---

## `STOPPING`

The machine is executing a controlled stop.

The state exists to distinguish:

> requested process stop

from:

> blocking fault response.

The exact stop sequence may be process dependent, but LAB-001 shall implement deterministic behaviour.

Following successful stopping:

```text id="6s95kp"
STOPPING → READY
```

provided readiness conditions remain valid.

---

## `FAULT`

A blocking fault prevents continued automatic operation.

Characteristics:

* automatic process progression is inhibited,
* outputs assume explicitly defined fault-state commands,
* fault information remains latched where required,
* reset is accepted only when recovery conditions are valid.

Following successful reset and revalidation:

```text id="jx8wej"
FAULT → INITIALIZING
```

is preferred over direct:

```text id="8ke035"
FAULT → READY
```

### Rationale

Returning through `INITIALIZING` forces the controller to re-evaluate the simulated machine condition after a fault.

This provides a more defensible recovery architecture than assuming the machine is automatically ready after fault reset.

---

# 7. Initial State Transition Model

```text id="bpplpe"
                 Enable
DISABLED ─────────────────► INITIALIZING
                               │
                    Valid      │      Invalid / Timeout
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
                  READY                 FAULT
                    │                     │
           Valid Start                    │ Valid Reset
                    │                     │
                    ▼                     │
                  RUNNING                 │
                    │                     │
          Stop      │                     │
                    ▼                     │
                 STOPPING                 │
                    │                     │
                    ▼                     │
                  READY                   │
                                          │
                    ◄─────────────────────┘
                     INITIALIZING

Blocking faults may cause applicable states to transition to FAULT.
```

Detailed transition conditions shall be documented in the functional description.

---

# 8. Operating Mode Model

The initial modes shall be:

```text id="g880js"
NONE
MANUAL
AUTOMATIC
```

---

# 9. Mode Definitions

## `NONE`

No productive operating mode is selected.

Used during:

* startup,
* disabled condition,
* initialization where appropriate.

---

## `MANUAL`

Individual permitted device actions may be requested by the operator.

Characteristics:

* automatic sequence disabled,
* actuator interlocks remain active,
* manual command requests pass through device-control logic.

---

## `AUTOMATIC`

The machine may execute the automatic sequence when required conditions are satisfied.

Selecting Automatic mode alone does not start production.

A valid Start request is still required.

---

# 10. Mode Transition Philosophy

For LAB-001 v1.0, normal mode selection shall be permitted primarily while the machine is in `READY`.

Recommended rule:

```text id="k39zzv"
READY + ModeRequestManual    → MANUAL
READY + ModeRequestAutomatic → AUTOMATIC
```

Mode changes during `RUNNING` shall not be accepted.

A request to change from Automatic to Manual while running shall therefore first require the machine to leave `RUNNING`.

---

# 11. Mode Behaviour During Fault

Initial architectural choice:

**Selected operating mode shall be retained through `FAULT`, but shall not authorize productive operation while the machine remains faulted.**

Example:

```text id="xxbgyg"
Before fault:
Mode = AUTOMATIC

During fault:
Mode = AUTOMATIC
State = FAULT
```

This preserves useful information about the pre-fault operating context.

After successful recovery through `INITIALIZING`, the machine may return to `READY` with the previous mode still selected.

A new Start request remains mandatory.

---

# 12. PLC Software Decomposition

The PLC application should initially contain functional blocks or equivalent logical modules representing:

```text id="dyxmdf"
Main
│
├── MachineSupervisor
│   ├── StateManager
│   ├── ModeManager
│   └── PermissiveManager
│
├── AutoSequence
│
├── ManualControl
│
├── FaultManager
│
├── DeviceControl
│   └── Actuator_01
│
├── Simulation
│
└── Diagnostics
```

The final TIA implementation may adjust block names, but responsibilities should remain recognizable.

---

# 13. Main Program Responsibility

The main cyclic program shall primarily coordinate block execution.

It should not become the primary location for detailed process logic.

Conceptually:

```text id="5g2mv9"
Read / Generate Inputs

Run Device Feedback Evaluation
Run Fault Detection
Run Fault Manager
Run Machine Supervisor
Run Manual Control
Run Automatic Sequence
Run Device Command Arbitration
Run Simulation
Update Diagnostics
```

Actual execution order shall be reviewed carefully because PLC scan order affects behaviour.

---

# 14. Machine Supervisor

The Machine Supervisor is responsible for high-level machine behaviour.

Responsibilities:

* machine-state management,
* state transitions,
* mode acceptance,
* automatic-start authorization,
* controlled-stop initiation,
* interaction with blocking faults,
* recovery transitions.

The Machine Supervisor shall **not** directly implement detailed actuator logic.

---

# 15. State Manager

The State Manager maintains:

* current state,
* previous state where useful,
* state-entry indication,
* transition conditions,
* transition requests.

Suggested data:

```text id="k334zy"
eStateCurrent
eStatePrevious
xStateChanged
xStateEntry
```

This supports diagnostics and deterministic entry actions.

---

# 16. Mode Manager

The Mode Manager is responsible for:

* processing mode requests,
* determining whether a mode change is permitted,
* retaining selected mode,
* exposing current mode.

Suggested modes:

```text id="bd4zfx"
MODE_NONE
MODE_MANUAL
MODE_AUTOMATIC
```

The Mode Manager shall not directly start automatic production.

---

# 17. Permissive Manager

The Permissive Manager evaluates conditions required for higher-level operations.

Initial example permissives:

```text id="havrqt"
xPermInitialized
xPermNoBlockingFault
xPermDeviceAvailable
xPermProcessConditionOK
```

Combined indication:

```text id="ys7zdt"
xPermAutoStart
```

Conceptually:

```text id="pf5ky7"
xPermAutoStart :=
    xPermInitialized
AND xPermNoBlockingFault
AND xPermDeviceAvailable
AND xPermProcessConditionOK;
```

Individual permissives shall remain visible diagnostically.

---

# 18. Permissive vs. Interlock Definition

For LAB-001, the following distinction shall be used.

## Permissive

A condition required before a higher-level operation may begin or continue.

Example:

> Machine may start an automatic cycle only when the actuator is available.

---

## Interlock

A condition that prevents a specific action or device command.

Example:

> Extend command is blocked while Retract command is active.

Therefore:

```text id="8rqgpp"
PERMISSIVE
Machine-level authorization

INTERLOCK
Action/device-level inhibition
```

This distinction is not universal across all industrial organizations, but it provides a clear and consistent project convention.

---

# 19. Automatic Sequence Controller

The automatic sequence shall implement the representative process.

It is responsible for:

* current sequence step,
* process progression,
* requesting device actions,
* evaluating step-completion conditions,
* signalling cycle complete,
* stopping sequence progression on blocking conditions.

The sequence shall **not** write final actuator outputs directly.

---

# 20. Initial Sequence Concept

A simple representative process is proposed:

```text id="kvwuxg"
STEP_00_IDLE

STEP_10_VERIFY_PART

STEP_20_EXTEND_ACTUATOR

STEP_30_PROCESS

STEP_40_RETRACT_ACTUATOR

STEP_50_COMPLETE
```

The actual process remains generic.

The purpose is to exercise:

* command requests,
* feedback,
* timeout,
* sequence transitions,
* cycle completion,
* stop,
* fault handling.

---

# 21. Manual Control

Manual Control receives simulated operator requests.

Example:

```text id="8kwc1l"
xManualExtendRequest
xManualRetractRequest
```

Manual Control converts these into **device requests**, not final outputs.

Manual requests shall only become effective when:

* Manual mode is active,
* machine state permits manual operation,
* device interlocks allow the requested action.

---

# 22. Command Arbitration

Automatic and Manual functions may both request device actions at different times.

A defined arbitration layer shall determine which request is valid.

Initial priority philosophy:

```text id="rm4yjf"
FAULT / Disabled behaviour
        ↓ highest priority

Machine state restrictions
        ↓

Device interlocks
        ↓

Manual or Automatic request
        ↓

Final command
```

Manual and Automatic requests shall not be simultaneously active under normal architecture.

---

# 23. Device-Control Layer

The representative device shall be implemented using a reusable Function Block.

Initial abstraction:

**Two-position actuator**

Possible interface:

### Inputs

```text id="j2afap"
xCmdExtendRequest
xCmdRetractRequest
xFbExtended
xFbRetracted
xEnable
xReset
tMovementTimeout
```

### Outputs / Status

```text id="cw6bcx"
xCmdExtend
xCmdRetract
xExtended
xRetracted
xMoving
xAvailable
xFaultTimeout
xFaultFeedbackConflict
```

The device FB owns:

```text id="9fecxp"
xCmdExtend
xCmdRetract
```

as final logical commands.

---

# 24. Device-Control Responsibilities

The device block shall manage:

* mutually exclusive output commands,
* feedback interpretation,
* movement status,
* timeout monitoring,
* contradictory feedback detection,
* availability status.

It should not determine the overall machine state.

Instead, it reports device faults upward.

---

# 25. Fault Architecture

Fault handling is divided into three layers.

```text id="w4hv5z"
FAULT DETECTION
      ↓
FAULT REGISTRATION
      ↓
MACHINE RESPONSE
```

---

# 26. Fault Detection

Subsystems detect their own abnormal conditions where this provides the clearest responsibility.

Example device faults:

```text id="j4x4fe"
Actuator extension timeout
Actuator retraction timeout
Contradictory position feedback
```

---

# 27. Fault Manager

The Fault Manager receives fault conditions and manages:

* active fault status,
* latching,
* fault identifier,
* blocking-fault summary,
* reset evaluation.

Initial structure may include:

```text id="hjy1pb"
xFaultActuatorTimeout
xFaultFeedbackConflict
xFaultInitialization

xAnyBlockingFault
uiActiveFaultCode
```

The architecture should allow later expansion to multiple fault entries.

---

# 28. Fault Severity

LAB-001 v1.0 shall initially implement:

**Blocking Fault**

only.

Warnings are deferred.

### Rationale

The goal of LAB-001 is to prove robust control and recovery architecture without adding unnecessary alarm-system complexity that belongs more naturally in LAB-002.

The architecture should nevertheless allow later addition of:

```text id="dp8wq7"
Warning
Blocking Fault
```

without fundamental redesign.

---

# 29. Fault Response Ownership

The Fault Manager identifies fault status.

The Machine Supervisor determines the machine-state response.

Example:

```text id="p66osq"
Device detects timeout
        ↓
Fault Manager latches fault
        ↓
xAnyBlockingFault = TRUE
        ↓
Machine Supervisor
        ↓
State → FAULT
```

This avoids allowing individual device blocks to change the overall machine state directly.

---

# 30. Reset Architecture

Reset shall be treated as a **request**, not an unconditional command.

Conceptually:

```text id="5l2osv"
Reset Request
     │
     ↓
Are fault causes cleared?
     │
   Yes / No
     │
     ↓
Fault Manager accepts or rejects reset
```

A reset shall not simply force:

```text id="p550op"
Fault := FALSE
```

without validating recovery conditions.

---

# 31. Recovery Architecture

Initial fault recovery sequence:

```text id="vugfrv"
FAULT
  │
  │ Fault cause removed
  │ Valid Reset Request
  ▼
INITIALIZING
  │
  │ Machine conditions verified
  ▼
READY
```

Automatic operation shall require a new Start request.

---

# 32. Output Behaviour During Fault

For the representative actuator:

```text id="rswhc5"
FAULT state:
Extend command = FALSE
Retract command = FALSE
```

unless later process requirements justify another defined response.

The key requirement is not that every future device must be de-energized.

The key architectural rule is:

> Fault-state output behaviour shall be explicit and intentional.

---

# 33. Process Simulation Layer

Simulation shall model basic device behaviour independently from machine-control logic.

Example:

```text id="mkw4xs"
Control output:
xCmdExtend
     ↓
Simulation delay
     ↓
xFbExtended
```

Fault injection controls shall allow the simulation to deliberately:

* suppress feedback,
* activate contradictory feedback,
* prevent initialization,
* hold a process condition invalid.

---

# 34. Simulation Architecture

Suggested simulation signals:

```text id="4x2glu"
xSimEnable
xSimFailExtend
xSimFailRetract
xSimFeedbackConflict
xSimWorkpiecePresent
xSimInitializationFailure
```

These signals are test infrastructure and shall remain separate from the production control logic.

---

# 35. Diagnostics Architecture

Diagnostics shall expose at minimum:

```text id="zikgdr"
Current machine state
Current operating mode
Current automatic step
Automatic-start permissive
Individual permissives
Actuator command
Actuator feedback
Device availability
Active blocking fault
Fault identifier
```

LAB-002 may later present these values through a proper HMI.

---

# 36. Data Organization

Where practical, related PLC data should be grouped logically.

Possible structures:

```text id="hvdkhw"
stMachine
stCommands
stPermissives
stFaults
stActuator01
stSimulation
stDiagnostics
```

Exact Siemens data types and DB implementation will be decided during software design.

The objective is to avoid a large collection of unrelated global tags.

---

# 37. Proposed PLC Block Structure

Initial candidate structure:

```text id="j8brtk"
OB1
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
└── FC_Diagnostics / supporting logic
```

Possible supporting types:

```text id="7kipov"
UDT_MachineStatus
UDT_MachineCommands
UDT_Permissives
UDT_FaultStatus
UDT_ActuatorStatus

ENUM / symbolic constants:
E_MachineState
E_OperatingMode
E_AutoStep
```

Final implementation depends on the capabilities and preferred structure within TIA Portal V20.

---

# 38. Preliminary Execution Order

PLC scan order must be intentional.

Initial candidate:

```text id="vttn60"
1. Acquire command / simulated input data

2. Evaluate device feedback

3. Detect local faults

4. Update Fault Manager

5. Evaluate machine permissives

6. Update Machine Supervisor

7. Execute Automatic Sequence

8. Evaluate Manual Requests

9. Resolve Device Commands

10. Update Process Simulation

11. Update Diagnostics
```

This order requires verification during implementation.

In particular, one-scan delays and dependencies between:

* feedback,
* faults,
* state transitions,
* and output commands

shall be documented if they materially affect behaviour.

---

# 39. Architecture Decision Summary

## ADR-001 — `DISABLED` instead of `OFF`

**Decision:** Use `DISABLED`.

**Reason:** PLC software cannot meaningfully represent actual physical power-off while executing.

---

## ADR-002 — Separate Machine State and Mode

**Decision:** State and mode are independent variables.

**Reason:** They represent different dimensions of machine behaviour.

---

## ADR-003 — No `PAUSED` State in Initial Version

**Decision:** Exclude from v1.0.

**Reason:** Avoid adding functionality until a clear process requirement exists.

---

## ADR-004 — Fault Recovery Through Initialization

**Decision:**

```text id="1pnp7c"
FAULT → INITIALIZING → READY
```

**Reason:** Machine readiness shall be revalidated after a blocking fault.

---

## ADR-005 — One Final Output Owner

**Decision:** Device-control block owns final actuator commands.

**Reason:** Avoid competing writes and scan-order ambiguity.

---

## ADR-006 — Sequence Uses Device Requests

**Decision:** Automatic sequence requests actions rather than writing outputs directly.

**Reason:** Separates process logic from device behaviour.

---

## ADR-007 — Manual Mode Does Not Bypass Interlocks

**Decision:** Interlocks remain effective.

**Reason:** Manual control changes command source, not fundamental device protection logic.

---

## ADR-008 — Blocking Faults Only in LAB-001

**Decision:** Warning architecture deferred.

**Reason:** Preserve focus on core control architecture.

---

## ADR-009 — Fault Manager Does Not Own Machine State

**Decision:** Fault Manager reports blocking fault; Machine Supervisor controls state transitions.

**Reason:** Maintain separation of responsibilities.

---

## ADR-010 — Simulation Separate from Controller Logic

**Decision:** Process simulator shall be an independent functional area.

**Reason:** Allows later replacement by physical I/O and supports controlled fault injection.

---

# 40. Architecture-to-Requirement Mapping

Initial examples:

| Architecture Element        | Primary Requirements         |
| --------------------------- | ---------------------------- |
| Machine State Manager       | FR-001–FR-010                |
| Mode Manager                | FR-011–FR-019                |
| Machine Supervisor          | FR-020–FR-031, FR-056–FR-059 |
| Device Control              | FR-032–FR-035, FR-043–FR-047 |
| Auto Sequence               | FR-036–FR-042                |
| Fault Manager               | FR-048–FR-055                |
| Diagnostics                 | FR-060–FR-066                |
| Command Interface           | IR-001                       |
| Process Simulation          | IR-002, DR-008               |
| Structured Software         | DR-001–DR-007                |
| Verification Infrastructure | VER-001–VER-010              |

A more complete traceability matrix shall be developed during verification planning.

---

# 41. Known Architecture Risks

## AR-001 — Excessive Abstraction

A generic architecture can become unnecessarily complicated if too much flexibility is introduced before a real application requires it.

**Mitigation:** Implement only abstractions required by LAB-001 and identified future reuse.

---

## AR-002 — State / Sequence Duplication

Machine State and Automatic Sequence Step could become overlapping concepts.

**Mitigation:**

Machine State represents overall machine lifecycle.

Sequence Step represents detailed progression only while `RUNNING`.

---

## AR-003 — Scan-Order Dependency

PLC logic may produce unintended one-cycle behaviour if blocks depend heavily on execution order.

**Mitigation:**

* intentional execution order,
* limited multiple writes,
* explicit ownership,
* verification of transitions.

---

## AR-004 — Fault Reset Complexity

Different faults may eventually require different recovery conditions.

**Mitigation:**

Begin with a generic reset framework but avoid assuming all future faults are identical.

---

## AR-005 — Simulation Contamination

Simulation-specific logic may accidentally enter production-control blocks.

**Mitigation:**

Maintain a distinct simulation layer and clearly named simulation tags.

---

# 42. Architecture Verification Questions

The implementation phase should answer:

1. Can the machine state be understood from one explicit state variable?
2. Can operating mode change without corrupting state logic?
3. Can the automatic sequence run without directly owning outputs?
4. Can Manual and Automatic command sources coexist without competing writes?
5. Can device faults be detected independently from machine-state logic?
6. Can a blocking fault reliably force the machine to `FAULT`?
7. Can reset be rejected while the underlying fault remains active?
8. Can recovery reliably pass through `INITIALIZING`?
9. Can simulation be disabled or replaced without restructuring the controller?
10. Can a technical reviewer identify which software block owns each major responsibility?

---

# 43. Current Architecture Baseline

LAB-001 v0.1 therefore uses the following central architecture:

```text id="5v3scq"
             COMMANDS
                │
                ▼
        MACHINE SUPERVISOR
          │            │
          │            ├── MODE MANAGER
          │
          ├── STATE MANAGER
          │
          └── PERMISSIVES
                │
        ┌───────┴────────┐
        ▼                ▼
 AUTO SEQUENCE      MANUAL CONTROL
        │                │
        └───────┬────────┘
                ▼
         DEVICE CONTROL
          │           ▲
          │           │
          ▼           │
       OUTPUTS      FEEDBACK
          │           │
          ▼           │
      PROCESS SIMULATION
                │
                ▼
             INPUTS

Device / system faults
        │
        ▼
   FAULT MANAGER
        │
        ▼
MACHINE SUPERVISOR
```

This architecture shall form the basis for the functional description and initial PLC implementation.
