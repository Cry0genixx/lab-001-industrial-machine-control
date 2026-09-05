# LAB-001 — Functional Description

**Document:** 04 — Functional Description\
**Project:** LAB-001 — Industrial Machine Control Core\
**Version:** v0.1\
**Status:** Draft\
**Last updated:** 2026-09-05

---

# 1. Purpose

This document describes the intended functional behaviour of LAB-001 — Industrial Machine Control Core.

The functional description defines how the control system shall behave during:

* startup,
* initialization,
* mode selection,
* Manual operation,
* Automatic operation,
* normal stopping,
* abnormal conditions,
* fault handling,
* reset,
* and recovery.

The document bridges the gap between the requirements specification and detailed PLC implementation.

---

# 2. Reference Process

LAB-001 uses a simplified generic discrete manufacturing process.

The process contains:

* one simulated workpiece-presence condition,
* one simulated two-position actuator,
* two actuator position-feedback signals,
* one simulated process step,
* operator command inputs,
* machine permissives,
* and representative fault conditions.

The system is intentionally generic.

The purpose is to verify control architecture rather than model one specific physical machine.

---

# 3. Reference Process Sequence

A successful automatic cycle shall conceptually perform the following sequence:

```text
1. Verify machine ready
2. Verify workpiece present
3. Command actuator to extended position
4. Confirm actuator extended
5. Execute simulated process
6. Command actuator to retracted position
7. Confirm actuator retracted
8. Complete cycle
```

This sequence may later represent operations such as:

* workpiece clamping,
* positioning,
* tooling engagement,
* inspection,
* or another simple discrete machine process.

---

# 4. Machine States

The control system shall use the following machine states:

```text
DISABLED
INITIALIZING
READY
RUNNING
STOPPING
FAULT
```

Only one machine state shall be active at any given time.

---

# 5. Operating Modes

The following operating modes shall be supported:

```text
NONE
MANUAL
AUTOMATIC
```

Machine state and operating mode shall remain separate concepts.

Example:

```text
State = READY
Mode  = MANUAL
```

and:

```text
State = READY
Mode  = AUTOMATIC
```

are both valid combinations.

---

# 6. Startup Behaviour

When the PLC application begins execution, the machine-control application shall enter:

```text
DISABLED
```

The controller shall not assume that the simulated machine is ready merely because the PLC program is running.

During `DISABLED`:

* automatic operation is not permitted,
* device commands shall be inactive,
* sequence logic shall not execute,
* diagnostic values remain available,
* an Enable / Initialize request may permit transition to `INITIALIZING`.

---

# 7. Enable and Initialization Request

The project shall provide a logical machine-enable request.

When:

```text
State = DISABLED
```

and the machine-enable conditions are valid, the controller may transition to:

```text
INITIALIZING
```

The transition shall not directly enter `READY`.

---

# 8. Initialization Behaviour

The purpose of `INITIALIZING` is to establish whether the simulated machine is in a known and acceptable condition.

Initialization shall evaluate at minimum:

* no active blocking fault,
* valid actuator feedback,
* no contradictory actuator feedback,
* required simulated process conditions,
* device in acceptable initial position or capable of reaching it.

The preferred initial actuator condition is:

```text
Retracted = TRUE
Extended  = FALSE
```

---

# 9. Initialization of Actuator Position

If the actuator is already confirmed retracted when initialization begins, the device may immediately satisfy the home-position requirement.

If the actuator is not confirmed retracted, the controller may request retraction as part of initialization.

Conceptually:

```text
INITIALIZING
      │
      ├─ Retracted already confirmed
      │       ↓
      │   Continue initialization
      │
      └─ Retracted not confirmed
              ↓
         Request Retract
              ↓
       Wait for feedback
```

---

# 10. Successful Initialization

Initialization shall complete successfully when:

* required initial conditions are valid,
* actuator home position is confirmed,
* no blocking fault is active.

The controller shall then transition:

```text
INITIALIZING → READY
```

---

# 11. Initialization Failure

Initialization shall fail if a required machine condition cannot be established.

Representative causes include:

* actuator fails to retract,
* contradictory actuator feedback,
* simulated initialization failure,
* blocking fault condition.

A timeout during initialization shall generate a fault.

Result:

```text
INITIALIZING → FAULT
```

The machine shall not enter `READY`.

---

# 12. READY Behaviour

`READY` represents an initialized, non-running machine.

During `READY`:

* automatic sequence is inactive,
* operating mode may be selected,
* Manual commands may be permitted in Manual mode,
* Automatic Start may be accepted in Automatic mode,
* diagnostic values remain available.

`READY` does not automatically mean that every automatic-start permissive is valid.

For example:

```text
Machine initialized = TRUE
Workpiece present   = FALSE
```

may still result in:

```text
State = READY
AutoStartPermissive = FALSE
```

This distinction is intentional.

---

# 13. Mode Selection

Mode changes shall normally be accepted only while:

```text
State = READY
```

Available operator requests:

```text
Request Manual
Request Automatic
```

---

# 14. Manual Mode Selection

If:

```text
State = READY
```

and Manual mode is requested, the controller shall set:

```text
Mode = MANUAL
```

Manual mode shall not automatically actuate any device.

---

# 15. Automatic Mode Selection

If:

```text
State = READY
```

and Automatic mode is requested, the controller shall set:

```text
Mode = AUTOMATIC
```

Automatic mode selection shall not itself start the automatic sequence.

A separate valid Start request is required.

---

# 16. Invalid Mode Change

Mode changes shall be rejected while the machine is actively executing an automatic cycle.

Therefore:

```text
State = RUNNING
```

shall prevent direct switching between Manual and Automatic modes.

The machine must first leave the active running condition.

---

# 17. Manual Operation

Manual operation shall be permitted when:

```text
State = READY
Mode  = MANUAL
```

The operator may request individual actuator movement.

Initial Manual commands:

```text
Manual Extend
Manual Retract
```

---

# 18. Manual Extend

A Manual Extend request shall generate an actuator-extend request only if applicable interlocks are satisfied.

Representative conditions:

* Manual mode active,
* machine in `READY`,
* no blocking fault,
* Retract command not active,
* contradictory feedback not detected.

If these conditions are invalid, the final actuator command shall remain inactive.

---

# 19. Manual Retract

A Manual Retract request shall generate an actuator-retract request only if applicable interlocks are satisfied.

The command shall be rejected if an incompatible command or device condition exists.

---

# 20. Manual Command Priority

Manual requests shall not write final outputs directly.

Command chain:

```text
Operator Request
      ↓
Manual Control
      ↓
Device Request
      ↓
Interlock Evaluation
      ↓
Device Control
      ↓
Final Command
```

This structure ensures that the same device-level protections remain active regardless of whether a command originates from Manual or Automatic operation.

---

# 21. Automatic Start Conditions

An Automatic Start request shall only be accepted when:

```text
State = READY
Mode  = AUTOMATIC
```

and all mandatory automatic-start permissives are satisfied.

Initial permissives should include:

* machine initialized,
* no blocking fault,
* actuator available,
* required initial actuator position confirmed,
* workpiece present,
* process condition valid.

---

# 22. Automatic Start Rejection

If any mandatory permissive is missing:

```text
Start Request = TRUE
```

shall not cause transition to `RUNNING`.

The machine shall remain:

```text
State = READY
```

and the missing permissive shall remain visible for diagnostics.

---

# 23. Automatic Start Acceptance

When all start conditions are valid and Start is requested:

```text
READY → RUNNING
```

The automatic sequence shall initialize to its first active process step.

---

# 24. Automatic Sequence Model

The initial sequence shall use explicit sequence steps.

Suggested step enumeration:

```text
SEQ_IDLE
SEQ_VERIFY_PART
SEQ_EXTEND
SEQ_PROCESS
SEQ_RETRACT
SEQ_COMPLETE
```

The sequence step is subordinate to the machine state.

Sequence progression shall only occur while:

```text
State = RUNNING
```

---

# 25. SEQ_IDLE

`SEQ_IDLE` represents an inactive automatic sequence.

The sequence shall normally remain in `SEQ_IDLE` while the machine is not in `RUNNING`.

When a valid automatic cycle begins, the sequence shall transition to:

```text
SEQ_VERIFY_PART
```

---

# 26. SEQ_VERIFY_PART

Purpose:

Confirm that the required workpiece or process condition remains available.

If:

```text
WorkpiecePresent = TRUE
```

the sequence may proceed to:

```text
SEQ_EXTEND
```

If the condition is lost before progression, the sequence shall not continue.

Whether prolonged absence becomes a fault or controlled stop shall be determined by the applicable process requirement.

For LAB-001, a missing workpiece detected before actuator motion should preferably prevent progression without immediately creating a blocking hardware-type fault.

---

# 27. SEQ_EXTEND

The automatic sequence shall request:

```text
ExtendRequest = TRUE
```

The Device Control block shall evaluate the request and, when permitted, issue:

```text
ExtendCommand = TRUE
```

The sequence shall wait for:

```text
ExtendedFeedback = TRUE
```

before progressing.

---

# 28. Extension Timeout

If extended feedback is not received within:

```text
tMovementTimeout
```

the device shall generate an extension-timeout fault.

The Fault Manager shall register the blocking fault.

The Machine Supervisor shall transition:

```text
RUNNING → FAULT
```

The automatic sequence shall stop progressing.

---

# 29. SEQ_PROCESS

After confirmed actuator extension, the controller shall execute a simulated process step.

The process may initially be represented using a timer.

Example:

```text
ProcessDuration = 1.0 s
```

When the simulated process completes successfully:

```text
SEQ_PROCESS → SEQ_RETRACT
```

The process timer is a simulation construct and shall not represent a validated industrial process time.

---

# 30. SEQ_RETRACT

The sequence shall request actuator retraction.

The device-control chain shall behave identically to extension:

```text
Sequence Request
      ↓
Device Control
      ↓
Retract Command
      ↓
Simulated Feedback
```

Progression requires:

```text
RetractedFeedback = TRUE
```

---

# 31. Retraction Timeout

Failure to receive retracted feedback within the configured timeout shall generate a blocking fault.

Result:

```text
RUNNING → FAULT
```

The cycle shall not be counted as complete.

---

# 32. SEQ_COMPLETE

When the actuator has successfully returned to its required final position, the sequence enters:

```text
SEQ_COMPLETE
```

The controller shall generate an identifiable cycle-complete condition.

If cycle counting is implemented:

```text
CycleCounter := CycleCounter + 1
```

shall occur only once per successfully completed cycle.

---

# 33. Automatic Cycle Completion

After cycle completion, the initial LAB-001 behaviour shall be:

```text
RUNNING → READY
```

The automatic sequence shall return to:

```text
SEQ_IDLE
```

A new Start request shall be required for another cycle.

Continuous automatic cycling is intentionally excluded from the initial implementation.

### Rationale

Single-cycle behaviour makes:

* state transitions,
* verification,
* fault injection,
* and sequence analysis

clearer during the first architecture implementation.

Continuous production may later be added as an extension.

---

# 34. Stop Request During RUNNING

If the operator issues:

```text
Stop Request = TRUE
```

during automatic operation, the controller shall transition:

```text
RUNNING → STOPPING
```

The stop request shall not be treated as a blocking fault.

---

# 35. Controlled Stop Philosophy

For LAB-001, controlled stop behaviour shall prioritize deterministic return to a known condition.

The initial approach shall be:

1. Stop automatic sequence progression.
2. Cancel process-level action requests.
3. Request actuator retraction where appropriate.
4. Wait for confirmed retracted position.
5. Return sequence to `SEQ_IDLE`.
6. Enter `READY`.

Conceptually:

```text
RUNNING
   │
   │ Stop Request
   ▼
STOPPING
   │
   │ Actuator returned to known position
   ▼
READY
```

---

# 36. Stop Failure

If the actuator fails to reach the required recovery position during `STOPPING`, the appropriate timeout fault shall be generated.

Result:

```text
STOPPING → FAULT
```

Therefore, a normal stop may escalate into a fault if the machine cannot reach its defined stopped condition.

---

# 37. Stop from READY

A Stop request while already in:

```text
READY
```

shall not create a fault.

It may be ignored or treated as an already-satisfied command.

---

# 38. Fault Detection

Representative blocking faults shall include:

```text
FAULT_ACTUATOR_EXTEND_TIMEOUT
FAULT_ACTUATOR_RETRACT_TIMEOUT
FAULT_FEEDBACK_CONFLICT
FAULT_INITIALIZATION_TIMEOUT
```

Additional faults may be introduced if they materially improve verification.

---

# 39. Contradictory Feedback

The two-position actuator uses mutually exclusive position feedback:

```text
ExtendedFeedback
RetractedFeedback
```

The following combination is invalid:

```text
ExtendedFeedback = TRUE
RetractedFeedback = TRUE
```

When this occurs, the device shall report:

```text
FeedbackConflictFault = TRUE
```

This shall become a blocking fault.

---

# 40. Intermediate Position

The following feedback combination may be valid during movement:

```text
ExtendedFeedback = FALSE
RetractedFeedback = FALSE
```

This represents an intermediate or unknown physical position.

It shall not automatically be interpreted as contradictory feedback.

However, if the commanded destination is not reached within the applicable timeout, a movement-timeout fault shall occur.

---

# 41. Blocking Fault Behaviour

When a blocking fault becomes active:

1. The Fault Manager shall register the fault.
2. Automatic sequence progression shall stop.
3. The Machine Supervisor shall enter `FAULT`.
4. Final device commands shall assume their defined fault-state values.
5. Diagnostic information shall remain available.
6. Automatic restart shall be inhibited.

---

# 42. Fault-State Output Behaviour

For the initial two-position actuator:

```text
ExtendCommand  = FALSE
RetractCommand = FALSE
```

during `FAULT`.

This is a deliberate logical control choice for LAB-001.

It shall not be interpreted as a universal safe-state requirement for all real industrial actuators.

---

# 43. Fault Latching

Blocking faults requiring operator recovery shall remain logically registered until a valid reset is accepted.

A transient disappearance of the original failure condition shall not by itself clear the fault.

---

# 44. Reset Request

The operator may request Reset while:

```text
State = FAULT
```

Reset shall be interpreted as:

> Request fault recovery evaluation.

It shall not mean:

> Force all faults inactive.

---

# 45. Reset Rejection

Reset shall be rejected while the active fault cause remains present.

Example:

```text
FeedbackConflict = TRUE
ResetRequest     = TRUE
```

Result:

```text
State remains FAULT
Fault remains active
```

---

# 46. Successful Fault Reset

When:

* the underlying fault cause is removed,
* Reset is requested,
* and required reset conditions are valid,

the Fault Manager may clear the recoverable latched fault.

The Machine Supervisor shall then transition:

```text
FAULT → INITIALIZING
```

---

# 47. Recovery Through Initialization

The machine shall not transition directly from `FAULT` to `READY`.

Instead:

```text
FAULT
  ↓
INITIALIZING
  ↓
READY
```

This ensures that:

* actuator position,
* readiness,
* device availability,
* and important initial conditions

are revalidated.

---

# 48. No Automatic Restart After Recovery

After successful recovery:

```text
State = READY
```

Even if:

```text
Mode = AUTOMATIC
```

the automatic sequence shall remain inactive.

A new Start request is mandatory.

---

# 49. Mode Retention Through Fault

The selected operating mode shall initially be retained through a fault.

Example:

```text
Before fault:
State = RUNNING
Mode  = AUTOMATIC

During fault:
State = FAULT
Mode  = AUTOMATIC

After recovery:
State = READY
Mode  = AUTOMATIC
```

A new Start request is still required.

This allows the operator to retain operating context without allowing automatic restart.

---

# 50. Disable Behaviour

A future or simulated Disable request may transition the machine from an appropriate non-running state to:

```text
DISABLED
```

For LAB-001 v1.0, disabling shall primarily be permitted from `READY`.

Transition:

```text
READY → DISABLED
```

When disabled:

* sequence returns to idle,
* device requests are removed,
* operating mode may be set to `NONE`,
* machine must reinitialize before returning to `READY`.

---

# 51. Device Availability

The representative actuator shall provide:

```text
xAvailable
```

A device may be considered available when:

* no active device fault exists,
* feedback is logically valid,
* device control is enabled.

The exact expression shall be finalized during PLC implementation.

---

# 52. Device Command Arbitration

Device requests may originate from:

* Manual Control,
* Automatic Sequence,
* initialization logic,
* stopping/recovery logic.

The final command-selection logic shall ensure that only one valid source controls the actuator at a time.

Conceptual priority:

```text
FAULT / DISABLED override
        ↓
Initialization / Recovery
        ↓
Stopping
        ↓
Automatic request
        ↓
Manual request
```

The final priority implementation shall be reviewed to ensure that Manual and Automatic operation remain intuitive.

---

# 53. Initialization Command Ownership

Initialization may require actuator retraction.

This request shall still pass through Device Control rather than bypassing it.

Therefore:

```text
Initialization Request
        ↓
Device Request Arbitration
        ↓
Device Interlocks
        ↓
Final Retract Command
```

---

# 54. Process Simulation

The simulator shall emulate actuator response.

Example normal extension behaviour:

```text
ExtendCommand = TRUE
        ↓
Simulated Delay
        ↓
RetractedFeedback = FALSE
ExtendedFeedback  = TRUE
```

Normal retraction:

```text
RetractCommand = TRUE
        ↓
Simulated Delay
        ↓
ExtendedFeedback  = FALSE
RetractedFeedback = TRUE
```

---

# 55. Simulation Fault Injection

The simulation layer shall provide deliberate abnormal-condition controls.

Initial examples:

```text
xSimFailExtend
xSimFailRetract
xSimFeedbackConflict
xSimInitializationFailure
xSimWorkpiecePresent
```

These values shall be used only for testing and simulation.

---

# 56. Simulated Extend Failure

When:

```text
xSimFailExtend = TRUE
```

and an Extend command is issued, the simulator shall withhold successful Extended feedback.

The controller shall eventually detect the configured movement timeout.

---

# 57. Simulated Retract Failure

When:

```text
xSimFailRetract = TRUE
```

the simulator shall prevent successful Retracted feedback.

This shall permit verification during:

* initialization,
* automatic cycle,
* controlled stopping,
* fault recovery.

---

# 58. Simulated Feedback Conflict

When:

```text
xSimFeedbackConflict = TRUE
```

the simulator shall create:

```text
ExtendedFeedback  = TRUE
RetractedFeedback = TRUE
```

The control system shall detect and register the invalid condition.

---

# 59. Workpiece Presence

The simulated process shall include:

```text
xWorkpiecePresent
```

This signal shall be a required automatic-start permissive.

Without workpiece presence:

```text
AutoStartPermissive = FALSE
```

and a Start request shall be rejected.

---

# 60. Loss of Workpiece During Cycle

For LAB-001 v1.0, loss of workpiece presence after actuator engagement shall be treated as an abnormal process condition.

Initial proposed behaviour:

```text
RUNNING → FAULT
```

if workpiece presence is lost after the sequence has committed to the process.

### Rationale

This gives LAB-001 a useful example of a permissive whose significance changes depending on process state.

Before cycle start:

> Missing workpiece prevents start.

During active processing:

> Unexpected disappearance indicates abnormal process behaviour.

---

# 61. Diagnostics

The following values shall be observable:

```text
Machine State
Operating Mode
Automatic Sequence Step
Auto Start Permissive
Individual Permissives
Blocking Fault Active
Active Fault Code

Actuator Extend Request
Actuator Retract Request
Actuator Extend Command
Actuator Retract Command
Extended Feedback
Retracted Feedback
Actuator Available

Workpiece Present
Cycle Complete
Cycle Counter
```

This information shall later support LAB-002 HMI integration.

---

# 62. Expected Normal Cycle Example

Initial condition:

```text
State                = READY
Mode                 = AUTOMATIC
WorkpiecePresent     = TRUE
ActuatorRetracted    = TRUE
NoBlockingFault      = TRUE
AutoStartPermissive  = TRUE
```

Operator:

```text
StartRequest = TRUE
```

Expected sequence:

```text
READY
 ↓
RUNNING

SEQ_VERIFY_PART
 ↓
SEQ_EXTEND
 ↓
Extended confirmed
 ↓
SEQ_PROCESS
 ↓
Process complete
 ↓
SEQ_RETRACT
 ↓
Retracted confirmed
 ↓
SEQ_COMPLETE
 ↓
READY
```

Cycle counter increments once.

---

# 63. Example Missing-Permissive Scenario

Initial condition:

```text
State            = READY
Mode             = AUTOMATIC
WorkpiecePresent = FALSE
```

Operator:

```text
StartRequest = TRUE
```

Expected behaviour:

```text
State remains READY
Sequence remains IDLE
No actuator command occurs
AutoStartPermissive = FALSE
```

Diagnostic information shall identify the missing condition.

---

# 64. Example Extension-Failure Scenario

Normal cycle begins.

Sequence enters:

```text
SEQ_EXTEND
```

Simulator:

```text
xSimFailExtend = TRUE
```

Expected behaviour:

```text
ExtendCommand = TRUE
ExtendedFeedback never arrives
Movement timeout expires
Actuator timeout fault registered
RUNNING → FAULT
Sequence progression stops
```

---

# 65. Example Invalid Reset Scenario

Fault:

```text
FAULT_FEEDBACK_CONFLICT
```

remains physically/simulated active.

Operator:

```text
ResetRequest = TRUE
```

Expected:

```text
Fault remains latched
State remains FAULT
Reset rejected
```

---

# 66. Example Successful Recovery Scenario

Starting condition:

```text
State = FAULT
Feedback conflict removed
```

Operator requests Reset.

Expected:

```text
Fault Manager validates reset
        ↓
Fault cleared
        ↓
FAULT → INITIALIZING
        ↓
Initial device condition verified
        ↓
READY
```

If previous mode was Automatic:

```text
Mode = AUTOMATIC
```

may remain selected.

Automatic cycle remains inactive until new Start request.

---

# 67. Example Controlled Stop

Starting condition:

```text
State = RUNNING
Sequence = SEQ_PROCESS
```

Operator:

```text
StopRequest = TRUE
```

Expected:

```text
RUNNING → STOPPING
Process request removed
Actuator requested to retract
Retracted feedback confirmed
Sequence reset to IDLE
STOPPING → READY
```

Cycle counter shall not increment unless the automatic cycle had already reached its formally defined successful completion condition.

---

# 68. Functional Design Principles Demonstrated

LAB-001 should demonstrate the following engineering principles:

1. Overall machine lifecycle is separate from detailed sequence progression.
2. Operating mode is separate from machine state.
3. Start permission is explicitly evaluated.
4. Commands and feedback are separate signals.
5. Device control owns final actuator outputs.
6. Manual operation does not bypass interlocks.
7. Automatic sequence requests actions rather than directly driving outputs.
8. Fault detection is separated from machine-state response.
9. Reset is conditional.
10. Recovery revalidates the machine.
11. Failed cycles do not count as successful production.
12. Simulation and fault injection are treated as verification infrastructure.

---

# 69. Functional Scope for v1.0

The required LAB-001 v1.0 behaviour is therefore:

```text
DISABLED
    ↓
INITIALIZING
    ↓
READY
    │
    ├── MANUAL device operation
    │
    └── AUTOMATIC Start
              ↓
           RUNNING
              │
       ┌──────┴───────┐
       ↓              ↓
Complete            Stop
       ↓              ↓
    READY         STOPPING
                      ↓
                    READY

Blocking Fault
      ↓
    FAULT
      ↓
Valid Reset
      ↓
INITIALIZING
      ↓
    READY
```

This behaviour shall form the basis for detailed software implementation and verification.
