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