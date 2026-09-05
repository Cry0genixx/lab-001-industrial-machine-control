# LAB-001 — Requirements Specification

**Document:** 02 — Requirements Specification\
**Project:** LAB-001 — Industrial Machine Control Core\
**Version:** v0.1\
**Status:** Draft\
**Last updated:** 2026-09-05

---

# 1. Purpose

This document defines the initial functional, interface, design and project constraints for LAB-001 — Industrial Machine Control Core.

The requirements establish the expected behaviour of the control system before detailed implementation begins.

The requirements shall form the basis for:

* system architecture,
* state-machine design,
* PLC software implementation,
* simulation,
* verification planning,
* and final project acceptance.

---

# 2. Requirement Categories

The following requirement identifiers are used.

| Prefix  | Category                         |
| ------- | -------------------------------- |
| **FR**  | Functional Requirement           |
| **IR**  | Interface Requirement            |
| **DR**  | Design Requirement               |
| **PR**  | Performance / Timing Requirement |
| **CON** | Constraint                       |
| **DOC** | Documentation Requirement        |
| **VER** | Verification Requirement         |

Safety-related machine functions are outside the validated scope of LAB-001.

Where a simulated safety-permissive signal is used, it shall be treated only as a standard control input and not as evidence of a functional-safety implementation.

---

# 3. Requirement Priority

Requirements use the following priority levels.

| Priority   | Meaning                                                |
| ---------- | ------------------------------------------------------ |
| **Must**   | Required for LAB-001 v1.0                              |
| **Should** | Important but may be deferred if technically justified |
| **Could**  | Optional extension                                     |
| **Future** | Intentionally outside LAB-001 v1.0                     |

---

# 4. Verification Methods

| Method                  | Description                                                 |
| ----------------------- | ----------------------------------------------------------- |
| **Inspection**          | Review of source code, configuration or documentation       |
| **Functional Test**     | Execute system behaviour under defined conditions           |
| **Fault Injection**     | Deliberately create abnormal conditions                     |
| **Simulation**          | Verify behaviour using simulated process/controller signals |
| **Analysis**            | Evaluate architecture, logic or calculations                |
| **Traceability Review** | Confirm requirement linkage to implementation and test      |

---

# 5. Machine-State Requirements

## FR-001 — Explicit Machine State

**Priority:** Must
**Verification:** Inspection + Functional Test

The controller shall maintain one explicitly identifiable active machine state.

The current machine state shall be available to other control functions for diagnostics and future HMI integration.

---

## FR-002 — Defined Machine States

**Priority:** Must
**Verification:** Inspection

The initial implementation shall support at minimum the following logical states:

* `OFF`
* `INITIALIZING`
* `READY`
* `AUTOMATIC`
* `STOPPING`
* `FAULT`

A `PAUSED` or equivalent held-production state may be included if justified during architecture development.

---

## FR-003 — Controlled State Transitions

**Priority:** Must
**Verification:** Functional Test

The controller shall transition between machine states only when explicitly defined transition conditions are satisfied.

---

## FR-004 — Invalid State Transitions

**Priority:** Must
**Verification:** Functional Test

Requests for undefined or invalid state transitions shall not cause the machine to enter an undefined or unintended state.

---

## FR-005 — Deterministic Startup State

**Priority:** Must
**Verification:** Functional Test

Upon controller initialization, the machine-control application shall enter a defined startup condition before production operation is permitted.

---

## FR-006 — Initialization State

**Priority:** Must
**Verification:** Functional Test

The controller shall use an initialization process to determine whether required simulated machine conditions are valid before entering `READY`.

---

## FR-007 — Initialization Failure

**Priority:** Must
**Verification:** Fault Injection

If initialization cannot complete because a required condition remains invalid beyond the permitted initialization period, the controller shall prevent entry into `READY` and shall provide a diagnosable abnormal condition.

---

## FR-008 — Ready State

**Priority:** Must
**Verification:** Functional Test

The controller shall enter `READY` only when the machine has completed initialization and no blocking fault condition is active.

---

## FR-009 — Fault Transition

**Priority:** Must
**Verification:** Fault Injection

A defined blocking fault occurring during an applicable operating state shall cause the controller to transition to `FAULT` or otherwise enter the defined fault-response condition.

---

## FR-010 — Recovery State

**Priority:** Must
**Verification:** Functional Test

Following successful fault reset, the controller shall return to a defined non-running state.

The initial intended recovery state is `READY`, subject to successful revalidation of required machine conditions.

---

# 6. Operating-Mode Requirements

## FR-011 — Explicit Operating Mode

**Priority:** Must
**Verification:** Functional Test

The controller shall maintain an explicitly identifiable operating mode independently from the machine state.

---

## FR-012 — Manual Mode

**Priority:** Must
**Verification:** Functional Test

The controller shall provide a `MANUAL` operating mode for permitted individual machine functions.

---

## FR-013 — Automatic Mode

**Priority:** Must
**Verification:** Functional Test

The controller shall provide an `AUTOMATIC` operating mode for sequential machine operation.

---

## FR-014 — Mode-State Separation

**Priority:** Must
**Verification:** Inspection + Analysis

Machine state and operating mode shall be represented as separate control concepts.

A machine state shall describe the current overall condition of the machine.

An operating mode shall describe the permitted method of operation.

---

## FR-015 — Mode Selection Conditions

**Priority:** Must
**Verification:** Functional Test

Operating-mode changes shall only be accepted under defined conditions.

Mode changes that would create ambiguous or unsafe control behaviour shall be rejected.

---

## FR-016 — Automatic Operation Permission

**Priority:** Must
**Verification:** Functional Test

Automatic operation shall only be permitted when:

* `AUTOMATIC` mode is selected,
* the machine is in an appropriate state,
* all required automatic-operation permissives are satisfied,
* and no blocking fault is active.

---

## FR-017 — Manual Operation Permission

**Priority:** Must
**Verification:** Functional Test

Manual operation shall only be permitted when `MANUAL` mode is active and the requested individual command satisfies its applicable interlocks.

---

## FR-018 — Automatic Sequence Disabled in Manual Mode

**Priority:** Must
**Verification:** Functional Test

The automatic sequence shall not start or continue as an active automatic process while the controller is in `MANUAL` mode.

---

## FR-019 — Manual Commands Disabled During Automatic Operation

**Priority:** Must
**Verification:** Functional Test

Manual actuator commands that could conflict with active automatic operation shall be inhibited while the machine is executing the automatic sequence.

---

# 7. Command Requirements

## FR-020 — Start Request

**Priority:** Must
**Verification:** Functional Test

The controller shall provide a simulated operator `Start` request.

---

## FR-021 — Start Acceptance

**Priority:** Must
**Verification:** Functional Test

A `Start` request shall be accepted only when all conditions required for automatic operation are valid.

---

## FR-022 — Start Rejection

**Priority:** Must
**Verification:** Functional Test

If a start request is rejected, the controller shall remain in a non-running state and expose sufficient diagnostic information to determine why the request was not accepted.

---

## FR-023 — Stop Request

**Priority:** Must
**Verification:** Functional Test

The controller shall provide a simulated operator `Stop` request.

---

## FR-024 — Controlled Stop

**Priority:** Must
**Verification:** Functional Test

A normal stop request during automatic operation shall initiate a defined controlled stop sequence rather than producing an undefined interruption of machine logic.

---

## FR-025 — Reset Request

**Priority:** Must
**Verification:** Functional Test

The controller shall provide a simulated operator `Reset` request for recoverable fault conditions.

---

## FR-026 — Reset Rejection

**Priority:** Must
**Verification:** Fault Injection

The controller shall reject a reset request when the underlying fault condition or another required reset condition remains invalid.

---

# 8. Permissive Requirements

## FR-027 — Automatic Permissive Evaluation

**Priority:** Must
**Verification:** Functional Test

The controller shall evaluate a defined set of permissives before allowing automatic operation.

---

## FR-028 — Permissive Visibility

**Priority:** Must
**Verification:** Inspection + Functional Test

Individual permissive states shall be available for diagnostics.

---

## FR-029 — Combined Automatic Permissive

**Priority:** Must
**Verification:** Functional Test

The controller shall provide an overall indication representing whether all mandatory conditions for automatic operation are satisfied.

---

## FR-030 — Loss of Pre-Start Permissive

**Priority:** Must
**Verification:** Functional Test

A missing permissive before cycle start shall prevent automatic cycle initiation.

---

## FR-031 — Loss of Permissive During Operation

**Priority:** Must
**Verification:** Functional Test + Fault Injection

For each permissive that may change during automatic operation, the required response to loss of that condition shall be explicitly defined.

The response may include:

* controlled stop,
* process hold,
* transition to `FAULT`,
* or another documented response.

A single universal response is not required for all permissives.

---

# 9. Interlock Requirements

## FR-032 — Command Interlocking

**Priority:** Must
**Verification:** Functional Test

Actuator commands shall be prevented when their defined interlock conditions are not satisfied.

---

## FR-033 — Conflicting Command Prevention

**Priority:** Must
**Verification:** Functional Test

The controller shall prevent mutually conflicting commands from being issued simultaneously.

---

## FR-034 — Interlock Visibility

**Priority:** Must
**Verification:** Functional Test

The status of important command interlocks shall be available for diagnostic use.

---

## FR-035 — Interlock Independence from Operator Request

**Priority:** Must
**Verification:** Functional Test

An operator request shall not override a blocking software interlock unless a specific and documented service function is intentionally implemented.

No such bypass function is required for LAB-001 v1.0.

---

# 10. Automatic Sequence Requirements

## FR-036 — Representative Automatic Cycle

**Priority:** Must
**Verification:** Simulation + Functional Test

The controller shall execute a representative automatic machine sequence sufficient to verify the control architecture.

---

## FR-037 — Defined Sequence Steps

**Priority:** Must
**Verification:** Inspection

The automatic process shall use explicitly identifiable sequence steps or equivalent structured state logic.

---

## FR-038 — Sequence Progression

**Priority:** Must
**Verification:** Functional Test

The automatic sequence shall progress only when the defined completion conditions for the current step are satisfied.

---

## FR-039 — Sequence Step Visibility

**Priority:** Must
**Verification:** Functional Test

The currently active automatic-sequence step shall be available for diagnostics.

---

## FR-040 — Cycle Complete

**Priority:** Must
**Verification:** Functional Test

Successful completion of the final automatic process step shall generate an identifiable cycle-complete condition.

---

## FR-041 — Cycle Counter

**Priority:** Should
**Verification:** Functional Test

The controller should increment a production or cycle counter after successful completion of an automatic cycle.

---

## FR-042 — Incomplete Cycle Handling

**Priority:** Must
**Verification:** Fault Injection

A cycle interrupted by a blocking fault shall not be counted as successfully completed.

---

# 11. Simulated Device Requirements

The initial reference process shall contain at least one simulated actuator and associated feedback sufficient to verify command, interlock and timeout behaviour.

---

## FR-043 — Simulated Actuator

**Priority:** Must
**Verification:** Simulation

The controller shall include at least one representative controlled device with a commanded state and simulated feedback.

---

## FR-044 — Device Command/Feedback Separation

**Priority:** Must
**Verification:** Inspection

The commanded actuator state and the confirmed feedback state shall be represented separately.

---

## FR-045 — Device Completion Feedback

**Priority:** Must
**Verification:** Simulation

Automatic sequence progression requiring actuator movement shall depend on confirmed simulated feedback rather than command state alone.

---

## FR-046 — Device Timeout

**Priority:** Must
**Verification:** Fault Injection

If an actuator fails to achieve the required feedback state within a defined timeout period, the controller shall generate a fault.

---

## FR-047 — Contradictory Device Feedback

**Priority:** Must
**Verification:** Fault Injection

If mutually exclusive simulated feedback signals are simultaneously active, the controller shall detect an abnormal condition.

---

# 12. Fault-Management Requirements

## FR-048 — Fault Identification

**Priority:** Must
**Verification:** Fault Injection

Each implemented fault shall have an identifiable fault source or fault code.

---

## FR-049 — Blocking Fault

**Priority:** Must
**Verification:** Fault Injection

The architecture shall support fault conditions that prevent further automatic operation until valid recovery conditions are satisfied.

---

## FR-050 — Fault Latching

**Priority:** Must
**Verification:** Fault Injection

Faults requiring operator acknowledgement or reset shall remain logically active until the defined reset conditions are satisfied.

---

## FR-051 — Fault Cause Retention

**Priority:** Should
**Verification:** Functional Test

Where practical, the controller should retain sufficient information to identify the cause of the active or most recent fault.

---

## FR-052 — Multiple Fault Support

**Priority:** Should
**Verification:** Fault Injection

The fault-management architecture should support more than one simultaneous or sequential fault condition without relying on a single undifferentiated fault bit.

---

## FR-053 — Fault Reset Conditions

**Priority:** Must
**Verification:** Fault Injection

A recoverable fault shall only be cleared when:

* the reset command is received,
* the underlying fault condition is no longer active,
* and any additional required recovery conditions are satisfied.

---

## FR-054 — No Automatic Restart After Fault

**Priority:** Must
**Verification:** Fault Injection

Clearing a fault shall not automatically restart an interrupted automatic production cycle.

A new valid start request shall be required unless a later project explicitly defines and verifies a different recovery philosophy.

---

## FR-055 — Fault-State Output Behaviour

**Priority:** Must
**Verification:** Fault Injection

The desired command state of controlled devices during `FAULT` shall be explicitly defined.

The implementation shall not leave output behaviour dependent on accidental execution order or residual automatic-sequence logic.

---

# 13. Stop and Recovery Requirements

## FR-056 — Stop-State Definition

**Priority:** Must
**Verification:** Inspection

The control architecture shall explicitly define the difference between:

* normal cycle completion,
* operator-requested stop,
* and fault-induced interruption.

---

## FR-057 — Normal Stop Destination

**Priority:** Must
**Verification:** Functional Test

After successful controlled stopping, the controller shall enter a defined non-running state.

The intended state is `READY`, provided readiness conditions remain valid.

---

## FR-058 — Recovery Validation

**Priority:** Must
**Verification:** Functional Test

Before returning to `READY` following fault recovery, the controller shall re-evaluate conditions required for machine readiness.

---

## FR-059 — Unknown Process Position

**Priority:** Should
**Verification:** Fault Injection + Analysis

If a fault results in an ambiguous simulated process position, the controller should require an explicit recovery or reinitialization process before automatic operation can resume.

---

# 14. Diagnostic Requirements

## FR-060 — Machine-State Diagnostic

**Priority:** Must
**Verification:** Functional Test

The active machine state shall be externally observable within the PLC application.

---

## FR-061 — Operating-Mode Diagnostic

**Priority:** Must
**Verification:** Functional Test

The active operating mode shall be externally observable within the PLC application.

---

## FR-062 — Sequence-Step Diagnostic

**Priority:** Must
**Verification:** Functional Test

The current automatic-sequence step shall be externally observable.

---

## FR-063 — Permissive Diagnostics

**Priority:** Must
**Verification:** Functional Test

The controller shall expose the status of individual mandatory automatic permissives.

---

## FR-064 — Interlock Diagnostics

**Priority:** Must
**Verification:** Functional Test

Relevant command interlock conditions shall be externally observable.

---

## FR-065 — Fault Diagnostics

**Priority:** Must
**Verification:** Functional Test

The active fault condition or fault identifier shall be externally observable.

---

## FR-066 — Command and Feedback Diagnostics

**Priority:** Should
**Verification:** Functional Test

Representative device command and feedback states should be externally observable for troubleshooting.

---

# 15. Interface Requirements

## IR-001 — Operator Command Interface

**Priority:** Must
**Verification:** Functional Test

The control core shall provide defined logical inputs for:

* Start,
* Stop,
* Reset,
* Manual mode request,
* Automatic mode request.

---

## IR-002 — Simulated Process Interface

**Priority:** Must
**Verification:** Simulation

The control core shall interface with simulated process signals representing sensors and actuator feedback.

---

## IR-003 — Future HMI Interface

**Priority:** Should
**Verification:** Inspection

Important state, mode, fault, permissive and diagnostic data shall be organized so that future HMI integration does not require major restructuring of the control architecture.

---

## IR-004 — Future Robot Interface

**Priority:** Should
**Verification:** Architecture Review

The architecture should permit later addition of an external machine or robot handshake without fundamental redesign of machine-state handling.

---

## IR-005 — Future SCADA Interface

**Priority:** Should
**Verification:** Architecture Review

Key machine-status information should be structured so that later exposure through OPC UA or another industrial interface is practical.

---

# 16. Software Design Requirements

## DR-001 — Structured Architecture

**Priority:** Must
**Verification:** Inspection

The PLC software shall be divided into logical functional areas rather than implemented as one monolithic program block.

---

## DR-002 — Separation of Concerns

**Priority:** Must
**Verification:** Inspection

Where practical, the architecture shall separate:

* machine-state management,
* mode management,
* automatic sequence,
* device control,
* fault management,
* and diagnostics.

---

## DR-003 — Reusable Device Logic

**Priority:** Must
**Verification:** Inspection + Functional Test

At least one device-control function shall be implemented using a reusable Function Block or equivalent reusable software construct.

---

## DR-004 — Descriptive Naming

**Priority:** Must
**Verification:** Inspection

Variables, blocks, states and functions shall use descriptive names reflecting engineering meaning.

---

## DR-005 — Enumerated States

**Priority:** Should
**Verification:** Inspection

Machine states and sequence steps should use enumerated or otherwise explicitly defined symbolic values rather than unexplained numerical constants.

---

## DR-006 — Named Parameters

**Priority:** Must
**Verification:** Inspection

Configurable engineering values such as timeout periods shall use named parameters or constants rather than undocumented literal values.

---

## DR-007 — Controlled Output Ownership

**Priority:** Must
**Verification:** Inspection

Each physical or simulated actuator command shall have a clearly identifiable software ownership path.

Multiple unrelated blocks shall not independently write competing final output commands.

---

## DR-008 — Simulation Separation

**Priority:** Should
**Verification:** Inspection

Simulated process behaviour should be separated from the primary machine-control logic so that future replacement with real I/O does not require fundamental restructuring.

---

# 17. Timing and Performance Requirements

LAB-001 is not intended to validate hard real-time process performance.

Nevertheless, representative timeout behaviour shall be implemented.

---

## PR-001 — Actuator Timeout

**Priority:** Must
**Verification:** Simulation + Fault Injection

Representative actuator movement shall have a configurable timeout.

Initial nominal value:

**2.0 s**

The final value is a simulation parameter and shall not be presented as a validated physical-machine requirement.

---

## PR-002 — Initialization Timeout

**Priority:** Must
**Verification:** Fault Injection

Initialization conditions requiring simulated device response shall have a defined timeout or other deterministic failure criterion.

---

## PR-003 — Reset Response

**Priority:** Should
**Verification:** Functional Test

A valid reset request should produce the intended logical response without unnecessary intentional delay.

---

# 18. Project Constraints

## CON-001 — PLC Platform

The primary implementation shall use Siemens TIA Portal V20.

---

## CON-002 — Simulation Platform

The initial controller verification shall use S7-PLCSIM V20 where technically feasible.

---

## CON-003 — No Required Physical Hardware

Successful completion of LAB-001 shall not depend on physical PLC hardware, sensors or actuators.

---

## CON-004 — No Functional-Safety Claim

The project shall not claim that simulated safety-related control signals constitute a validated or certified functional-safety solution.

---

## CON-005 — Control Architecture Priority

Process complexity shall remain secondary to development and verification of the control architecture.

---

## CON-006 — Portfolio Reproducibility

Published documentation shall provide sufficient information for a technically competent reviewer to understand the control concept even if proprietary Siemens project-file formats cannot be directly reviewed.

---

# 19. Documentation Requirements

## DOC-001 — State Diagram

**Priority:** Must
**Verification:** Inspection

A graphical state-machine diagram shall document the final machine-state architecture.

---

## DOC-002 — State Transition Table

**Priority:** Must
**Verification:** Inspection

The project shall contain a table describing:

* current state,
* transition condition,
* destination state.

---

## DOC-003 — Software Architecture Diagram

**Priority:** Must
**Verification:** Inspection

A diagram shall describe major PLC software blocks and their responsibilities.

---

## DOC-004 — I/O / Simulation Signal List

**Priority:** Must
**Verification:** Inspection

Simulated input, output and command signals shall be documented.

---

## DOC-005 — Functional Description

**Priority:** Must
**Verification:** Inspection

Startup, operation, stop, fault and recovery behaviour shall be described.

---

## DOC-006 — Design Decisions

**Priority:** Must
**Verification:** Inspection

Important architectural decisions shall be documented with rationale.

---

## DOC-007 — Verification Results

**Priority:** Must
**Verification:** Inspection

Test procedures and results shall be documented.

---

## DOC-008 — Lessons Learned

**Priority:** Must
**Verification:** Inspection

A project retrospective shall be completed before Portfolio Verified status.

---

# 20. Verification Requirements

## VER-001 — Requirement Traceability

**Priority:** Must

Every `Must` functional requirement shall be linked to at least one verification activity or explicit justification if direct verification is not applicable.

---

## VER-002 — Normal Operation Testing

**Priority:** Must

The verification plan shall test the intended normal operating sequence from initialization through successful cycle completion.

---

## VER-003 — Invalid Start Test

**Priority:** Must

At least one missing permissive shall be deliberately introduced to verify rejection of automatic start.

---

## VER-004 — Actuator Timeout Test

**Priority:** Must

Simulated actuator feedback shall be deliberately withheld to verify timeout detection.

---

## VER-005 — Contradictory Feedback Test

**Priority:** Must

Contradictory simulated device feedback shall be introduced to verify abnormal-condition detection.

---

## VER-006 — Fault Reset Rejection Test

**Priority:** Must

Reset shall be requested while a fault cause remains active to verify that the reset is rejected.

---

## VER-007 — Successful Recovery Test

**Priority:** Must

The fault condition shall be removed and a valid reset performed to verify controlled recovery.

---

## VER-008 — No Automatic Restart Test

**Priority:** Must

The controller shall be verified not to restart production automatically following fault reset.

---

## VER-009 — Stop Test

**Priority:** Must

An operator-requested stop during automatic operation shall be tested.

---

## VER-010 — Invalid Transition Testing

**Priority:** Must

Representative invalid mode or state requests shall be tested to confirm that undefined transitions do not occur.

---

# 21. Preliminary Traceability Matrix

The detailed verification matrix will be completed in `06-verification-plan.md`.

Initial mapping:

| Requirement Area      | Primary Verification              |
| --------------------- | --------------------------------- |
| State management      | Functional test                   |
| Mode management       | Functional test                   |
| Start permissives     | Functional test                   |
| Interlocks            | Functional test                   |
| Sequence control      | Simulation                        |
| Actuator feedback     | Simulation                        |
| Timeouts              | Fault injection                   |
| Fault management      | Fault injection                   |
| Reset                 | Functional test + fault injection |
| Diagnostics           | Inspection + functional test      |
| Software architecture | Inspection                        |
| Documentation         | Review                            |

---

# 22. Open Engineering Decisions

The following items are intentionally not finalized in this requirements baseline.

## TBD-001 — PAUSED State

Determine whether `PAUSED` should exist as:

* a dedicated machine state,
* a sequence condition,
* or be excluded from LAB-001 v1.0.

---

## TBD-002 — OFF State

Determine whether `OFF` represents:

* an application-level disabled state,
* an uninitialized logical state,
* or whether it should be removed because an executing PLC cannot meaningfully represent true powered-off behaviour.

This distinction shall be resolved in the architecture phase.

---

## TBD-003 — Mode Selection While READY

Define whether mode switching shall be allowed only in `READY`, or also in other non-running states.

---

## TBD-004 — Mode During FAULT

Define whether the selected operating mode shall:

* be retained through `FAULT`,
* be forced to a neutral mode,
* or require reselection after recovery.

---

## TBD-005 — Controlled Stop Philosophy

Define whether an operator stop:

* completes the current process step,
* immediately transitions to a defined stopping sequence,
* or uses process-dependent behaviour.

For LAB-001, deterministic and understandable behaviour is more important than reproducing every industrial stop philosophy.

---

## TBD-006 — Fault Severity Classes

Determine whether LAB-001 should implement:

* one blocking-fault class only,
* or both warning and blocking-fault classes.

---

## TBD-007 — Sequence Representation

Determine whether the automatic sequence will use:

* explicit sequence-step enumeration,
* nested state machine,
* dedicated sequence Function Block,
* or another structured method.

---

## TBD-008 — Device Model

Finalize the representative simulated device used for initial verification.

Preferred candidate:

**double-position actuator abstraction with commanded extend/retract behaviour and two mutually exclusive position-feedback signals.**

This provides useful test cases for:

* command/feedback separation,
* timeout,
* contradictory feedback,
* manual control,
* automatic sequencing,
* interlocks,
* and recovery.

---

# 23. Initial Requirements Baseline

This document represents:

**LAB-001 Requirements Baseline v0.1**

The requirements are expected to evolve during architecture development.

Changes shall be made deliberately and significant changes should be reflected in Git history.

The purpose of the baseline is not to pretend that all engineering decisions are final.

Its purpose is to provide a defined starting point against which architecture, implementation and verification can be evaluated.

---

# 24. Acceptance Principle

LAB-001 v1.0 shall not be considered verified merely because the PLC program executes without errors.

Successful project completion requires evidence that:

* intended behaviour is defined,
* the architecture implements that behaviour,
* representative normal conditions are tested,
* representative abnormal conditions are tested,
* failures are handled predictably,
* recovery is controlled,
* and major requirements are traceable to documented verification results.
