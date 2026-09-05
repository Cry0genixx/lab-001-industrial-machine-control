# LAB-001 — Verification Plan

**Document:** 06 — Verification Plan\
**Project:** LAB-001 — Industrial Machine Control Core\
**Version:** v0.1\
**Status:** Draft\
**Last updated:** 2026-09-05

---

# 1. Purpose

This document defines the verification strategy for LAB-001 — Industrial Machine Control Core.

The purpose of verification is to determine whether the implemented control system satisfies the defined project requirements.

The project shall not be considered verified merely because the PLC program:

* compiles,
* enters RUN mode,
* or successfully executes one normal cycle.

Verification shall include deliberate testing of:

* normal operation,
* invalid operator requests,
* missing permissives,
* interlocks,
* actuator failures,
* contradictory feedback,
* fault handling,
* reset behaviour,
* recovery,
* and controlled stopping.

---

# 2. Verification Objective

The verification process shall provide evidence that the implemented architecture behaves predictably under both expected and abnormal conditions.

The primary verification question is:

> **Does the implemented machine-control core satisfy the documented requirements under the defined simulated operating conditions?**

---

# 3. Verification Environment

The initial verification environment shall use:

* Siemens TIA Portal V20
* S7-PLCSIM V20
* simulated operator commands
* simulated process feedback
* deliberate fault-injection controls

Physical PLC hardware is not required for LAB-001 v1.0.

---

# 4. Verification Boundary

LAB-001 verification covers primarily:

* PLC logical behaviour,
* state transitions,
* operating modes,
* automatic sequence behaviour,
* device command handling,
* simulated feedback,
* permissives,
* interlocks,
* timeout handling,
* fault management,
* reset,
* recovery,
* diagnostics.

The verification does **not** validate:

* physical actuator dynamics,
* electrical hardware,
* real safety functions,
* industrial network timing,
* physical emergency-stop behaviour,
* certified functional-safety performance,
* production-machine reliability.

---

# 5. Verification Methods

The following methods shall be used.

## 5.1 Inspection

Review of:

* PLC source code,
* block structure,
* symbolic names,
* data structures,
* state definitions,
* configuration,
* documentation.

---

## 5.2 Functional Testing

Execution of defined operating scenarios using simulated inputs and commands.

---

## 5.3 Fault Injection

Deliberate introduction of abnormal conditions.

Examples:

* actuator feedback withheld,
* contradictory feedback,
* missing workpiece,
* failed initialization.

---

## 5.4 Simulation

Use of simulated process behaviour to verify interaction between PLC commands and process feedback.

---

## 5.5 Analysis

Review of architectural behaviour that may not require execution.

Examples:

* output ownership,
* state/mode separation,
* software responsibilities.

---

## 5.6 Traceability Review

Confirmation that important requirements have corresponding verification activities and recorded results.

---

# 6. Test Result Status

Each test shall receive one of the following statuses.

| Result      | Meaning                                                            |
| ----------- | ------------------------------------------------------------------ |
| **PASS**    | Actual behaviour satisfies the defined acceptance criteria.        |
| **FAIL**    | Actual behaviour does not satisfy the defined acceptance criteria. |
| **PARTIAL** | Only part of the intended requirement could be verified.           |
| **BLOCKED** | Test could not be completed because of another unresolved issue.   |
| **NOT RUN** | Test has not yet been performed.                                   |

Tests shall not be marked PASS solely because no obvious error was observed.

---

# 7. Test Case Structure

Each formal test shall contain:

```text
Test ID
Related Requirement(s)
Objective
Test Type
Preconditions
Initial State
Test Inputs / Fault Injection
Procedure
Expected Result
Actual Result
Result
Evidence
Notes
```

---

# 8. Evidence Types

Verification evidence may include:

* PLCSIM screenshots,
* TIA watch-table values,
* trend captures where useful,
* PLC diagnostic values,
* test logs,
* exported result tables,
* Git commits,
* source-code references,
* short video recordings,
* manual test notes.

Evidence shall be stored in a reproducible and understandable form.

---

# 9. Test Naming Convention

Formal tests shall use:

```text
TC-001
TC-002
TC-003
...
```

Test groups may also use functional prefixes internally if this later improves readability.

Example:

```text
STATE-TC-001
FAULT-TC-001
```

For LAB-001 v1.0, a single sequential `TC-XXX` identifier system is sufficient.

---

# 10. Verification Sequence

Testing should occur approximately in the following order:

```text
Static Architecture Review
        ↓
Startup / Initialization
        ↓
Mode Management
        ↓
Manual Device Operation
        ↓
Automatic Start Logic
        ↓
Normal Automatic Cycle
        ↓
Controlled Stop
        ↓
Fault Injection
        ↓
Reset / Recovery
        ↓
Diagnostics
        ↓
Regression Test
        ↓
Final Traceability Review
```

This allows fundamental architecture problems to be discovered before more complex integration tests.

---

# 11. Phase A — Static Architecture Verification

## TC-001 — Machine State Ownership

**Requirements:** FR-001, DR-001, DR-002
**Type:** Inspection

### Objective

Verify that overall machine state is explicitly represented and owned by one logical supervisory function.

### Acceptance Criteria

* one explicit active machine-state variable exists,
* the variable uses symbolic states,
* unrelated blocks do not independently overwrite the final machine state.

---

## TC-002 — Operating Mode Separation

**Requirements:** FR-011, FR-014
**Type:** Inspection

### Objective

Verify that operating mode and machine state are represented independently.

### Acceptance Criteria

The implementation shall allow valid combinations such as:

```text
State = READY
Mode = MANUAL
```

without encoding the mode directly into the state value.

---

## TC-003 — Final Output Ownership

**Requirements:** DR-007
**Type:** Inspection

### Objective

Verify that the representative actuator has one clearly identifiable final logical command owner.

### Acceptance Criteria

* final Extend command has one writer,
* final Retract command has one writer,
* Manual and Automatic logic provide requests rather than competing final output writes.

---

## TC-004 — Simulation Separation

**Requirements:** DR-008, IR-002
**Type:** Inspection

### Objective

Verify that simulation-specific controls are separated from production-control decisions.

### Acceptance Criteria

Machine-control logic shall not rely directly on variables such as:

```text
xSimFailExtend
xSimFeedbackConflict
```

The controller shall instead consume normal simulated feedback signals.

---

## TC-005 — Block Responsibility Review

**Requirements:** DR-001, DR-002
**Type:** Inspection

### Objective

Verify recognizable separation between:

* Machine Supervisor,
* Automatic Sequence,
* Fault Manager,
* Device Control,
* Process Simulation.

### Acceptance Criteria

No major block shall contain large amounts of unrelated functionality without documented justification.

---

# 12. Phase B — Startup and Initialization

## TC-006 — Deterministic Startup

**Requirements:** FR-005
**Type:** Functional Test

### Preconditions

PLC application restarted or initialized.

### Expected Result

Machine shall enter:

```text
DISABLED
```

before production operation is permitted.

### Acceptance Criteria

The machine shall not directly enter `READY` or `RUNNING` after PLC startup.

---

## TC-007 — Enable to Initialization

**Requirements:** FR-006
**Type:** Functional Test

### Preconditions

```text
State = DISABLED
NoBlockingFault = TRUE
```

### Procedure

Issue valid Enable / Initialize request.

### Expected Result

```text
DISABLED → INITIALIZING
```

---

## TC-008 — Successful Initialization from Home Position

**Requirements:** FR-006, FR-008
**Type:** Functional Test

### Preconditions

Actuator:

```text
RetractedFeedback = TRUE
ExtendedFeedback  = FALSE
```

No blocking fault.

### Procedure

Enable the machine.

### Expected Result

```text
DISABLED
→ INITIALIZING
→ READY
```

### Acceptance Criteria

Machine shall enter `READY` without unnecessary actuator motion.

---

## TC-009 — Initialization Requires Retraction

**Requirements:** FR-006
**Type:** Simulation

### Preconditions

Actuator is not initially confirmed retracted.

### Procedure

Start initialization.

### Expected Result

Controller shall request actuator retraction through the normal device-control path.

After Retracted feedback is received:

```text
INITIALIZING → READY
```

---

## TC-010 — Initialization Retraction Timeout

**Requirements:** FR-007, PR-002
**Type:** Fault Injection

### Preconditions

Actuator not retracted.

Enable:

```text
xSimFailRetract = TRUE
```

### Procedure

Start initialization.

### Expected Result

* Retract command issued,
* Retracted feedback withheld,
* timeout expires,
* initialization fault registered,
* machine enters `FAULT`.

### Acceptance Criteria

Machine shall never enter `READY`.

---

## TC-011 — Initialization Feedback Conflict

**Requirements:** FR-007, FR-047
**Type:** Fault Injection

### Fault Injection

```text
ExtendedFeedback  = TRUE
RetractedFeedback = TRUE
```

### Expected Result

* feedback conflict detected,
* blocking fault registered,
* initialization cannot complete,
* state enters `FAULT`.

---

# 13. Phase C — Mode Management

## TC-012 — Select Manual Mode

**Requirements:** FR-012, FR-015
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = NONE
```

### Procedure

Issue Manual mode request.

### Expected Result

```text
Mode = MANUAL
```

Machine remains:

```text
State = READY
```

---

## TC-013 — Select Automatic Mode

**Requirements:** FR-013, FR-015
**Type:** Functional Test

### Preconditions

```text
State = READY
```

### Procedure

Issue Automatic mode request.

### Expected Result

```text
Mode = AUTOMATIC
State = READY
```

No automatic cycle shall start solely because the mode was selected.

---

## TC-014 — Reject Mode Change During RUNNING

**Requirements:** FR-015, FR-019
**Type:** Functional Test

### Preconditions

```text
State = RUNNING
Mode = AUTOMATIC
```

### Procedure

Issue Manual mode request.

### Expected Result

* request rejected,
* mode remains `AUTOMATIC`,
* automatic machine behaviour is not corrupted.

---

## TC-015 — Retain Mode During Fault

**Requirements:** Design decision ADR-038
**Type:** Functional Test

### Preconditions

```text
Mode = AUTOMATIC
State = RUNNING
```

Inject blocking fault.

### Expected Result

```text
State = FAULT
Mode = AUTOMATIC
```

---

# 14. Phase D — Manual Device Operation

## TC-016 — Manual Extend

**Requirements:** FR-017, FR-032
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = MANUAL
NoBlockingFault = TRUE
```

### Procedure

Activate Manual Extend request.

### Expected Result

Device-control layer issues Extend command when interlocks are valid.

---

## TC-017 — Manual Retract

**Requirements:** FR-017, FR-032
**Type:** Functional Test

Perform equivalent test for Retract.

---

## TC-018 — Manual Command Rejected Outside Manual Mode

**Requirements:** FR-017
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = AUTOMATIC
```

### Procedure

Activate Manual Extend.

### Expected Result

No final Extend command shall be generated.

---

## TC-019 — Conflicting Manual Commands

**Requirements:** FR-033
**Type:** Functional Test

### Procedure

Activate:

```text
ManualExtendRequest = TRUE
ManualRetractRequest = TRUE
```

### Expected Result

The device-control architecture shall prevent simultaneous final Extend and Retract commands.

The selected rejection or priority behaviour shall be documented.

---

## TC-020 — Manual Command Blocked by Fault

**Requirements:** FR-032, FR-049
**Type:** Fault Injection

### Preconditions

Manual mode active.

Activate a blocking fault.

### Procedure

Issue actuator movement request.

### Expected Result

No prohibited actuator command shall be issued.

---

# 15. Phase E — Automatic Start Permissives

## TC-021 — Valid Automatic Start

**Requirements:** FR-016, FR-020, FR-021
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = AUTOMATIC
WorkpiecePresent = TRUE
ActuatorRetracted = TRUE
NoBlockingFault = TRUE
AutoStartPermissive = TRUE
```

### Procedure

Issue Start request.

### Expected Result

```text
READY → RUNNING
```

Automatic sequence begins.

---

## TC-022 — Start Rejected in Manual Mode

**Requirements:** FR-016
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = MANUAL
```

Issue Start.

### Expected Result

Machine remains `READY`.

Automatic sequence remains inactive.

---

## TC-023 — Start Rejected Without Workpiece

**Requirements:** FR-022, FR-027, FR-030
**Type:** Functional Test

### Preconditions

```text
State = READY
Mode = AUTOMATIC
WorkpiecePresent = FALSE
```

### Procedure

Issue Start.

### Expected Result

* machine remains `READY`,
* sequence remains idle,
* no actuator command,
* Auto Start Permissive = FALSE,
* missing Workpiece permissive visible.

---

## TC-024 — Start Rejected with Blocking Fault

**Requirements:** FR-016, FR-029
**Type:** Functional Test

### Preconditions

Blocking fault active.

Issue Start.

### Expected Result

Automatic cycle shall not begin.

---

## TC-025 — Start Command Edge Handling

**Requirements:** ADR-024
**Type:** Functional Test

### Procedure

Hold Start input TRUE for multiple PLC scans.

### Expected Result

The system shall process one start event rather than repeatedly initiating the cycle.

---

# 16. Phase F — Normal Automatic Sequence

## TC-026 — Complete Automatic Cycle

**Requirements:** FR-036–FR-040
**Type:** Simulation

### Preconditions

All automatic start conditions valid.

### Procedure

Issue valid Start.

### Expected Sequence

```text
READY
→ RUNNING

SEQ_VERIFY_PART
→ SEQ_EXTEND
→ SEQ_PROCESS
→ SEQ_RETRACT
→ SEQ_COMPLETE

RUNNING
→ READY
```

### Acceptance Criteria

* each step occurs in correct order,
* required feedback gates progression,
* no unexpected fault occurs,
* sequence returns to `SEQ_IDLE`,
* machine returns to `READY`.

---

## TC-027 — Extend Requires Feedback

**Requirements:** FR-038, FR-045
**Type:** Simulation

### Procedure

Allow Extend command but delay Extended feedback.

### Expected Result

Sequence shall remain in `SEQ_EXTEND` until:

* Extended feedback arrives,
* or timeout fault occurs.

Command state alone shall not advance the sequence.

---

## TC-028 — Retract Requires Feedback

**Requirements:** FR-038, FR-045
**Type:** Simulation

Equivalent test for `SEQ_RETRACT`.

---

## TC-029 — Process Step Duration

**Requirements:** Functional Description
**Type:** Simulation

### Procedure

Observe simulated process step.

### Expected Result

`SEQ_PROCESS` shall remain active for the configured process duration before progressing.

---

## TC-030 — Cycle Counter

**Requirements:** FR-041
**Type:** Functional Test

### Preconditions

Record initial cycle count.

### Procedure

Complete one valid automatic cycle.

### Expected Result

Cycle count increments exactly once.

---

## TC-031 — Failed Cycle Does Not Increment Counter

**Requirements:** FR-042
**Type:** Fault Injection

### Procedure

Trigger fault during the automatic cycle before successful completion.

### Expected Result

Cycle count shall remain unchanged.

---

# 17. Phase G — Controlled Stop

## TC-032 — Stop During Extend Step

**Requirements:** FR-023, FR-024, FR-056, FR-057
**Type:** Functional Test

### Preconditions

Automatic cycle active in `SEQ_EXTEND`.

### Procedure

Issue Stop.

### Expected Result

```text
RUNNING → STOPPING
```

The system shall attempt controlled return to the defined retracted condition.

After confirmed retraction:

```text
STOPPING → READY
```

Sequence returns to `SEQ_IDLE`.

---

## TC-033 — Stop During Process Step

**Requirements:** FR-024
**Type:** Functional Test

Equivalent stop test while:

```text
SEQ_PROCESS
```

is active.

---

## TC-034 — Stop During Retract Step

**Requirements:** FR-024
**Type:** Functional Test

Equivalent stop test while actuator is already returning.

System shall still reach deterministic `READY`.

---

## TC-035 — Stop While READY

**Requirements:** FR-057
**Type:** Functional Test

### Preconditions

```text
State = READY
```

### Procedure

Issue Stop.

### Expected Result

No fault shall occur.

Machine remains in a valid non-running state.

---

## TC-036 — Stop Recovery Failure

**Requirements:** FR-024, FR-046
**Type:** Fault Injection

### Preconditions

Machine enters `STOPPING`.

Inject:

```text
xSimFailRetract = TRUE
```

### Expected Result

* retraction timeout occurs,
* blocking fault registered,
* machine enters `FAULT`.

---

# 18. Phase H — Actuator Failure Testing

## TC-037 — Extend Timeout

**Requirements:** FR-046, VER-004
**Type:** Fault Injection

### Fault Injection

```text
xSimFailExtend = TRUE
```

### Expected Result

* Extend command occurs,
* Extended feedback does not occur,
* movement timeout expires,
* Extend Timeout fault latches,
* `RUNNING → FAULT`,
* cycle does not complete.

---

## TC-038 — Retract Timeout

**Requirements:** FR-046
**Type:** Fault Injection

Equivalent test for Retract.

---

## TC-039 — Contradictory Feedback During READY

**Requirements:** FR-047, VER-005
**Type:** Fault Injection

### Fault Injection

```text
ExtendedFeedback = TRUE
RetractedFeedback = TRUE
```

### Expected Result

* conflict detected,
* blocking fault registered,
* machine transitions to `FAULT` if applicable.

---

## TC-040 — Intermediate Feedback Is Not Immediate Conflict

**Requirements:** ADR-032
**Type:** Functional Test

### Condition

```text
ExtendedFeedback = FALSE
RetractedFeedback = FALSE
```

during valid actuator motion.

### Expected Result

No immediate Feedback Conflict fault shall occur.

A timeout may occur later if commanded position is not reached.

---

## TC-041 — Simultaneous Device Fault Conditions

**Requirements:** FR-052
**Type:** Fault Injection

### Procedure

Create more than one fault condition where practical.

### Expected Result

* blocking-fault summary remains valid,
* individual fault causes remain diagnostically observable,
* primary fault-code behaviour matches documented priority.

---

# 19. Phase I — Workpiece / Process Faults

## TC-042 — Workpiece Missing Before Start

Covered primarily by TC-023.

Verify that missing workpiece is a permissive failure rather than automatically a blocking fault while idle.

---

## TC-043 — Workpiece Lost During Active Process

**Requirements:** FR-031, ADR-034
**Type:** Fault Injection

### Preconditions

Cycle has progressed into a committed active process condition.

### Procedure

Set:

```text
WorkpiecePresent = FALSE
```

### Expected Result

A defined process fault shall be generated and automatic operation interrupted.

Expected state:

```text
RUNNING → FAULT
```

---

## TC-044 — Workpiece Restored Without Reset

**Type:** Fault Injection

### Preconditions

Workpiece-loss fault is latched.

Restore:

```text
WorkpiecePresent = TRUE
```

without Reset.

### Expected Result

Fault shall remain latched and machine remain in `FAULT`.

---

# 20. Phase J — Reset and Recovery

## TC-045 — Reset While Fault Cause Active

**Requirements:** FR-026, FR-053, VER-006
**Type:** Fault Injection

### Preconditions

Blocking fault active and underlying fault cause still present.

### Procedure

Issue Reset.

### Expected Result

* reset rejected,
* fault remains latched,
* state remains `FAULT`.

---

## TC-046 — Successful Reset After Fault Clearance

**Requirements:** FR-053, VER-007
**Type:** Functional Test

### Preconditions

Machine in `FAULT`.

Remove underlying fault condition.

Issue Reset.

### Expected Result

```text
FAULT → INITIALIZING
```

After successful initialization:

```text
INITIALIZING → READY
```

---

## TC-047 — Failed Reinitialization After Reset

**Requirements:** FR-058
**Type:** Fault Injection

### Preconditions

Fault cleared sufficiently for reset acceptance, but machine cannot re-establish required initial position.

### Expected Result

Machine shall not incorrectly enter `READY`.

A new fault or failed initialization condition shall occur.

---

## TC-048 — No Automatic Restart

**Requirements:** FR-054, VER-008
**Type:** Functional Test

### Preconditions

Before fault:

```text
Mode = AUTOMATIC
State = RUNNING
```

Recover fault successfully.

### Expected Result

```text
State = READY
Mode = AUTOMATIC
```

Automatic sequence remains inactive.

A new Start request is required.

---

## TC-049 — Mode Retained After Recovery

**Requirements:** ADR-038
**Type:** Functional Test

Verify that selected mode remains as designed after successful recovery.

---

# 21. Phase K — Disable Behaviour

## TC-050 — Disable from READY

**Type:** Functional Test

### Preconditions

```text
State = READY
```

Issue Disable request if implemented.

### Expected Result

```text
READY → DISABLED
```

Outputs inactive.

Sequence idle.

Mode may return to `NONE` according to implemented architecture.

---

## TC-051 — Re-Enable Requires Initialization

**Type:** Functional Test

### Preconditions

Machine in `DISABLED`.

Issue Enable.

### Expected Result

Machine shall pass through:

```text
INITIALIZING
```

before returning to `READY`.

---

# 22. Phase L — Diagnostics Verification

## TC-052 — Machine State Diagnostic

**Requirements:** FR-060
**Type:** Functional Test

Verify that state diagnostic matches actual machine behaviour during all major states.

---

## TC-053 — Mode Diagnostic

**Requirements:** FR-061
**Type:** Functional Test

Verify active mode is observable and correct.

---

## TC-054 — Sequence-Step Diagnostic

**Requirements:** FR-062
**Type:** Functional Test

Verify that current automatic step is visible and updates correctly.

---

## TC-055 — Permissive Diagnostics

**Requirements:** FR-063
**Type:** Functional Test

Deliberately remove individual permissives.

Verify each individual condition and combined Auto Start Permissive are observable.

---

## TC-056 — Interlock Diagnostics

**Requirements:** FR-064
**Type:** Functional Test

Create at least one blocked device request.

Verify relevant blocking interlock can be identified.

---

## TC-057 — Fault Diagnostic

**Requirements:** FR-065
**Type:** Fault Injection

Generate each implemented fault individually.

Verify:

* overall fault status,
* specific fault condition,
* primary fault identifier.

---

## TC-058 — Device Diagnostic

**Requirements:** FR-066
**Type:** Functional Test

Verify visibility of:

* device request,
* final command,
* feedback,
* movement status,
* availability.

---

# 23. Phase M — Invalid Transition Testing

## TC-059 — Start While DISABLED

**Requirements:** FR-004, VER-010
**Type:** Functional Test

Issue Start while:

```text
State = DISABLED
```

### Expected Result

No transition to RUNNING.

---

## TC-060 — Start While INITIALIZING

Issue Start.

Expected:

No unintended transition.

---

## TC-061 — Start While FAULT

Issue Start while faulted.

Expected:

No automatic cycle.

---

## TC-062 — Manual Request While RUNNING

Attempt manual actuator command during automatic cycle.

Expected:

Manual command has no conflicting effect.

---

## TC-063 — Reset Outside FAULT

Issue Reset while machine is healthy.

Expected:

No unintended state change or sequence start.

---

## TC-064 — Enable Request While RUNNING

If Enable is implemented as a specific event, verify repeated Enable commands do not corrupt machine state.

---

# 24. Phase N — Regression Testing

After significant software changes, a reduced regression set shall be repeated.

Minimum regression suite:

```text
TC-006  Startup
TC-008  Initialization
TC-012  Manual mode
TC-013  Automatic mode
TC-021  Valid start
TC-023  Missing permissive
TC-026  Normal cycle
TC-032  Controlled stop
TC-037  Extend timeout
TC-039  Feedback conflict
TC-045  Invalid reset
TC-046  Successful recovery
TC-048  No automatic restart
TC-052  State diagnostics
```

The regression suite may expand as the project matures.

---

# 25. Preliminary Requirement Traceability Matrix

The following matrix is intentionally high-level.

A complete final matrix shall be updated after implementation.

| Requirement | Primary Test(s)              |
| ----------- | ---------------------------- |
| FR-001      | TC-001, TC-052               |
| FR-002      | TC-001                       |
| FR-003      | TC-006–TC-015, TC-059–TC-064 |
| FR-004      | TC-059–TC-064                |
| FR-005      | TC-006                       |
| FR-006      | TC-007–TC-009                |
| FR-007      | TC-010, TC-011               |
| FR-008      | TC-008, TC-009               |
| FR-009      | TC-037–TC-043                |
| FR-010      | TC-046–TC-048                |
| FR-011      | TC-002, TC-053               |
| FR-012      | TC-012                       |
| FR-013      | TC-013                       |
| FR-014      | TC-002                       |
| FR-015      | TC-012–TC-014                |
| FR-016      | TC-021–TC-024                |
| FR-017      | TC-016–TC-020                |
| FR-018      | TC-022                       |
| FR-019      | TC-014, TC-062               |
| FR-020      | TC-021                       |
| FR-021      | TC-021                       |
| FR-022      | TC-023, TC-024               |
| FR-023      | TC-032–TC-036                |
| FR-024      | TC-032–TC-036                |
| FR-025      | TC-045–TC-049                |
| FR-026      | TC-045                       |
| FR-027      | TC-021–TC-024                |
| FR-028      | TC-055                       |
| FR-029      | TC-021–TC-024, TC-055        |
| FR-030      | TC-023                       |
| FR-031      | TC-043                       |
| FR-032      | TC-016–TC-020                |
| FR-033      | TC-019                       |
| FR-034      | TC-056                       |
| FR-035      | TC-019, TC-020               |
| FR-036      | TC-026                       |
| FR-037      | TC-026, TC-054               |
| FR-038      | TC-027, TC-028               |
| FR-039      | TC-054                       |
| FR-040      | TC-026                       |
| FR-041      | TC-030                       |
| FR-042      | TC-031                       |
| FR-043      | TC-016–TC-018, TC-026        |
| FR-044      | TC-027, TC-028, TC-058       |
| FR-045      | TC-027, TC-028               |
| FR-046      | TC-037, TC-038               |
| FR-047      | TC-011, TC-039, TC-040       |
| FR-048      | TC-057                       |
| FR-049      | TC-020, TC-037–TC-043        |
| FR-050      | TC-044, TC-045               |
| FR-051      | TC-057                       |
| FR-052      | TC-041                       |
| FR-053      | TC-045, TC-046               |
| FR-054      | TC-048                       |
| FR-055      | TC-037–TC-043                |
| FR-056      | TC-032–TC-036                |
| FR-057      | TC-032–TC-035                |
| FR-058      | TC-046, TC-047               |
| FR-059      | TC-047                       |
| FR-060      | TC-052                       |
| FR-061      | TC-053                       |
| FR-062      | TC-054                       |
| FR-063      | TC-055                       |
| FR-064      | TC-056                       |
| FR-065      | TC-057                       |
| FR-066      | TC-058                       |

---

# 26. Design Requirement Verification

| Requirement                        | Verification                   |
| ---------------------------------- | ------------------------------ |
| DR-001 Structured Architecture     | TC-005                         |
| DR-002 Separation of Concerns      | TC-005                         |
| DR-003 Reusable Device Logic       | Code inspection + device tests |
| DR-004 Descriptive Naming          | Inspection                     |
| DR-005 Symbolic States             | TC-001 + inspection            |
| DR-006 Named Parameters            | Inspection                     |
| DR-007 Controlled Output Ownership | TC-003                         |
| DR-008 Simulation Separation       | TC-004                         |

---

# 27. Documentation Verification

Before Portfolio Verified status, confirm:

| Document                      | Required |
| ----------------------------- | :------: |
| README                        |    Yes   |
| Problem Definition            |    Yes   |
| Requirements                  |    Yes   |
| System Architecture           |    Yes   |
| Functional Description        |    Yes   |
| Design Decisions              |    Yes   |
| Verification Plan             |    Yes   |
| Test Results                  |    Yes   |
| Lessons Learned               |    Yes   |
| State Diagram                 |    Yes   |
| State Transition Table        |    Yes   |
| Software Architecture Diagram |    Yes   |
| Signal / I/O List             |    Yes   |

---

# 28. Verification Evidence Storage

Suggested repository structure:

```text
tests/
│
├── test-cases/
├── results/
└── logs/

media/
└── screenshots/
    ├── normal-operation/
    ├── fault-tests/
    └── recovery/
```

The exact structure may be simplified if manual documentation remains clearer.

---

# 29. Test Result File

Formal results shall eventually be summarized in:

```text
docs/07-test-results.md
```

Each executed test should contain:

```text
Test ID
Date
Software Version
Result
Observed Behaviour
Evidence Reference
Deviation / Notes
```

---

# 30. Failed Test Handling

A failed test shall not be removed from the record merely because the software is later corrected.

Recommended workflow:

```text
Test fails
    ↓
Record result
    ↓
Investigate cause
    ↓
Create corrective change
    ↓
Commit change
    ↓
Repeat test
    ↓
Record new result
```

This provides useful evidence of:

* troubleshooting,
* root-cause analysis,
* controlled iteration.

---

# 31. Verification Version Control

Each meaningful verification milestone should correspond to a defined project revision.

Suggested lifecycle:

```text
v0.1 Requirements baseline
v0.2 Architecture baseline
v0.3 Initial PLC implementation
v0.4 Normal-operation implementation
v0.5 Fault-handling implementation
v0.6 Verification candidate
v0.7 Verification corrections
v0.9 Portfolio review
v1.0 Portfolio Verified
```

The exact version sequence may change as required.

---

# 32. Minimum Acceptance Set for v1.0

LAB-001 shall not reach `VERIFIED` status unless at minimum the following tests pass:

```text
TC-001  Machine State Ownership
TC-002  Mode Separation
TC-003  Final Output Ownership
TC-006  Deterministic Startup
TC-008  Successful Initialization
TC-010  Initialization Failure
TC-012  Manual Mode
TC-013  Automatic Mode
TC-014  Invalid Mode Change
TC-016  Manual Extend
TC-018  Manual Rejection Outside Mode
TC-019  Conflicting Commands
TC-021  Valid Automatic Start
TC-023  Missing-Permissive Rejection
TC-026  Complete Automatic Cycle
TC-027  Feedback-Based Progression
TC-030  Cycle Counter
TC-031  Failed Cycle Counter
TC-032  Controlled Stop
TC-036  Stop Recovery Failure
TC-037  Extend Timeout
TC-038  Retract Timeout
TC-039  Feedback Conflict
TC-040  Valid Intermediate Position
TC-043  Workpiece Lost During Process
TC-045  Invalid Fault Reset
TC-046  Successful Fault Recovery
TC-048  No Automatic Restart
TC-052  Machine-State Diagnostics
TC-055  Permissive Diagnostics
TC-057  Fault Diagnostics
TC-059  Invalid Start from DISABLED
TC-061  Invalid Start from FAULT
```

Tests outside this minimum set should still be completed where practical.

---

# 33. Portfolio Verification Gate

After functional verification, the project shall undergo an additional portfolio review.

The project may reach **Portfolio Verified** only when:

* required tests are complete,
* unresolved failures are explained,
* source structure is clean,
* requirements are traceable,
* key test evidence is retained,
* screenshots/videos accurately represent behaviour,
* limitations are documented,
* README reflects actual implementation,
* lessons learned are complete,
* no unsupported claims remain.

---

# 34. Verification Philosophy

LAB-001 shall intentionally demonstrate not only that:

> **the machine works when everything is correct**

but also that:

> **the machine behaves predictably when conditions are incorrect.**

The portfolio value of LAB-001 depends heavily on this distinction.

Normal operation demonstrates implementation.

Abnormal-condition testing demonstrates engineering maturity.

---

# 35. Verification Readiness

The project is considered ready to begin PLC implementation when:

* requirements baseline exists,
* architecture baseline exists,
* functional description exists,
* major design decisions are recorded,
* verification plan exists.

LAB-001 has therefore reached the intended documentation gate for initial PLC implementation.
