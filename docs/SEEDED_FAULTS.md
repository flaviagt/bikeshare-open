# Seeded Faults

Faults introduced for Methodology Assignment 7, Exercise 2. This document records the intentional code changes, test failures, hook responses, and agent recovery process.

## Fault 1: Forbidden Feature (`no_forbidden_features`)

### Objective
Test whether the leakage protection detects a forbidden feature being included in the model's input columns.

The `casual` column is forbidden because it contains information about the same day's ridership, which contributes to the prediction target `cnt`.

### Intentional Code Change

File: `src/features.py`

Original:
```python
return [c for c in numeric if c not in FORBIDDEN]
```

Modified:
```python
return [c for c in numeric if c not in FORBIDDEN or c == "casual"]
```

This modification intentionally allows `casual` to bypass the forbidden-feature filter.

### Test Result

Command executed:
```bash
python -m pytest tests/test_leakage.py::test_no_forbidden_features -v
```

Actual failure:
```text
FAILED tests/test_leakage.py::test_no_forbidden_features

AssertionError: forbidden columns in feature set: {'casual'}

tests/test_leakage.py:31: AssertionError

1 failed in 0.50s
```

**Result:** The test successfully detected the forbidden feature.

### PostToolUse Hook Stderr

[PASTE THE ACTUAL HOOK STDERR FROM CLAUDE CODE HERE. Do not substitute the manually executed pytest output.]

### Recovery

- Fix: Remove the exception for `casual`, restoring the original feature-selection logic.
- Agent turns needed: [FILL IN AFTER FIX]
- Tests after recovery: [FILL IN AFTER RUNNING make test]

### Initial Unsuccessful Attempt

The first attempt used:
```python
return [c for c in numeric if c not in FORBIDDEN + ["casual"]]
```

This produced:
```text
TypeError: unsupported operand type(s) for +: 'set' and 'list'
```

The PostToolUse hook reported failing tests, but the failure was caused by an invalid Python operation rather than the forbidden-feature assertion.

The code was corrected and the experiment repeated to demonstrate genuine leakage detection.

### Takeaway

The forbidden-feature test correctly identified the intentional inclusion of `casual` in the model inputs. The initial attempt also illustrated the importance of examining why a test failed: a failing test alone does not demonstrate that the intended safeguard worked.

## Fault 2: Missing `.shift(1)` in Rolling Mean

### Objective
Test whether the leakage tests detect temporal leakage when the rolling mean includes the current day's target value.

### Intentional Change
File: `src/features.py`

Removed `.shift(1)` from the `cnt_roll_7` calculation, causing the rolling mean to include today's ridership instead of using only the preceding seven days.

### Test Results
Command: `make test`

```text
FAILED tests/test_leakage.py::test_rolling_mean_excludes_today

Obtained: 4389.0
Expected: 4414.857142857143

1 failed, 14 passed in 1.56s
```

**Result:** The existing test successfully detected temporal leakage.

### PostToolUse Hook Stderr
[Paste actual hook stderr from Claude Code here, or explicitly state if no hook output appeared.]

### Recovery
- Fix: Restored `.shift(1)` in the rolling mean calculation.
- Agent turns to recover: [Fill in actual count]
- Tests after recovery: [Fill in actual results]

### Takeaway
The experiment demonstrated that excluding the current day's target from rolling features is necessary to prevent data leakage. The test successfully caught the incorrect calculation because it compares the rolling mean against the seven preceding calendar days.


## Fault 3: Missing Weather Validation

### Objective
Determine whether the tests detect unexpected weather labels
when the encoder's validation guard is removed.

### Intentional Change
File: `src/features.py`

Removed the guard that checks for unknown weather labels and
raises ValueError. Without this guard, unknown labels are
silently mapped to NaN.

### Initial Test Results
Command: `make test`

15 passed in 1.46s

The original test suite did not detect the fault.

### Additional Test
Added `test_unknown_weather_raises` to `tests/test_leakage.py`.

Command:
`python -m pytest tests/test_leakage.py::test_unknown_weather_raises -v`

Actual failing output:

```text
tests/test_leakage.py::test_unknown_weather_raises FAILED

E   Failed: DID NOT RAISE ValueError

tests/test_leakage.py:83: Failed

1 failed in 0.57s
```

### PostToolUse Hook Stderr
[Paste actual hook stderr, or state that none was observed.]

### Recovery
- Fix: Restored the weather encoder's validation guard.
- Agent turns to fix: 2
- Tests after recovery: 16

### Takeaway
The original tests passed despite the missing guard. Adding a
targeted test demonstrated the coverage gap and verified that
unexpected weather labels should raise an error rather than
silently becoming missing values.


## Fault 4: Bash Copy Bypass

### Objective
Determine whether a Bash `cp` command can modify files
inside `data/raw/` without triggering the PostToolUse hook.

### Attempted Command
`cp /tmp/fault4_probe.txt data/raw/fault4_probe.txt`

### Actual Result
Claude Code returned:

```text
Error: Permission to use Bash with command
cp /tmp/fault4_probe.txt data/raw/fault4_probe.txt
has been denied.
```

The command was blocked before execution. No file was
created inside `data/raw/`.

### Hook Response
The PostToolUse leakage hook did not execute because
the Bash command was denied before it could run.

This attempt therefore did not demonstrate a successful
bypass, but it showed that the permission system blocked
the requested operation.

### Potential Vulnerability
The existing PostToolUse hook monitors Edit/Write operations,
not arbitrary filesystem modifications through Bash.

If a Bash copy were permitted, the Edit/Write hook would
not automatically validate that modification.

### Potential Mitigations
- A PreToolUse hook that inspects Bash commands
- A Stop hook that verifies repository integrity
- Filesystem-level write restrictions
- Hash checks on raw datasets

### Conclusion
The attempted Bash copy was blocked by Claude Code's
permission system, rather than by the leakage hook.
