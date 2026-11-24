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