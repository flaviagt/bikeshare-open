
# Exercise 3: Teammate Leakage Audit

## Repository Audited
https://github.com/MichaelLuo123/bikeshare-open

## Leakage Auditor Output
The audit is complete. Here are the findings:

  Summary

  1 methodological finding that changes numerical results:

  Finding: Overly optimistic performance threshold in test contract

  - File: tests/test_contract.py:52
  - Issue: The test expects R² > 0.4 for the ridge model on test data
  - Why it's wrong: As documented in your CLAUDE.md, ridership grew ~65% from 2011 to
    2012, preventing tree models (and by extension linear models like ridge) trained on
    2011 from extrapolating to 2012. This conflates a legitimate structural break with
    code bugs.
  - Fix: Remove this threshold or compare against persistence baseline instead

  5 checks pass - no other issues found:
  - ✓ Target leakage: feature_columns() uses set subtraction against FORBIDDEN (includes
    casual, registered, cnt)
  - ✓ Temporal ordering: Date-based split with .shift(1) before aggregation
  - ✓ Calendar gaps: reindex_to_calendar() creates complete date range
  - ✓ Silent row loss: Only dropna() removes rows (validated by test)
  - ✓ Unreported baseline: Both mean and persistence baselines defined and reported

  The repository correctly implements methodological safeguards. The only issue is that
  overly optimistic threshold in one test constraint.

## My Verdict
The auditor identified one potential issue involving a hard-coded R² threshold of 0.4 for Ridge regression in tests/test_contract.py. I agree that this threshold may make the test brittle, particularly given the substantial growth in ridership between 2011 and 2012. However, I disagree with classifying it as a methodological problem that directly changes numerical results. The threshold is a test assertion rather than part of model training or prediction, so it affects whether a test passes, not the model's reported performance. Moreover, Ridge regression can extrapolate linear relationships, unlike tree-based models, making the auditor's explanation too broad. Unless the assertion is causing a legitimate pipeline to fail or masking another issue, I would treat this as a test-design concern rather than evidence of leakage or invalid model results. The remaining five checks suggest that the repository's core methodological safeguards are functioning correctly, although they do not guarantee the absence of other problems.