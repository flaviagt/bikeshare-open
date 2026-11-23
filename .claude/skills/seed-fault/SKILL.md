# Seeded Faults: $ARGUMENTS

A reusable agent that introduces exactly one requested fault, captures evidence, and restores on approval. This is the core capability for Methodology Assignment 7, Exercise 2.

## Workflow

1. **Verify baseline** — Confirm `make test` passes before any fault.
2. **Introduce the fault** — Edit the relevant file to break the specified behavior.
3. **Capture evidence** — Record:
   - Which feature or line was modified
   - The exact code change (diff)
   - The hook stderr and failing test output
4. **Stop** — Leave the fault in place so you can inspect it, commit it, and write the report.
5. **Restore on approval** — After your explicit approval (`/ok continue`), revert to correct code and verify tests pass again.
6. **Record** — Append a single entry to `docs/SEEDED_FAULTS.md`.

## Available faults: $FAULTS

- **no_forbidden_features** — Remove `"cnt"` from the `FORBIDDEN` set in `src/config.py`, letting leakage reach the model.
- **test_no_forbidden_features_removed** — Comment out the `test_no_forbidden_features` test in `tests/test_leakage.py`.
- **lag_not_shifted** — Remove `.shift(2)` from `cnt_lag_2`, so it becomes today's count instead of two days ago.

## Usage

```text
/seed-fault <fault-name>
```

Example: `/seed-fault no_forbidden_features`

## What the skill does

- Runs `make test` once and confirms all tests pass before modifying anything.
- Uses the `Edit` tool to make the minimal change required to break the specified test.
- Runs `make test` again, captures both the hook stderr and failing test output.
- Writes the fault summary, code change, and failure evidence to `docs/SEEDED_FAULTS.md`.
- **Stops here.** The fault is committed in your working tree for you to review.

## Recovery (after your approval)

Once you confirm the fault captures the intended behaviour, say `/ok continue` and the skill will:

1. Revert the code change using `Edit` or `git checkout`.
2. Run `make test` again to confirm recovery.
3. Verify that only one entry was added to `docs/SEEDED_FAULTS.md`.

## What this is not

- It does not fix faults automatically without your approval.
- It does not introduce more than one fault at a time.
- It does not fabricate outputs — any missing value (like today's count for a lag) will cause the test to fail naturally, and that failure is recorded as-is.
- It preserves existing features like `cnt_lag_2` and never deletes them.

## Output artifact

The agent leaves these artifacts in your working tree:

| Artifact | Purpose |
|----------|---------|
| Modified source file(s) | The fault, ready for you to inspect and commit |
| `docs/SEEDED_FAULTS.md` (new entry) | One-line summary + code diff + test failure output |
| Failing pytest stdout/stderr | Captured in the new `SEEDED_FAULTS.md` entry |

## Example session

```text
$ /seed-fault no_forbidden_features

[Step 1] Baseline check — OK, all tests pass.

[Step 2] Introducing fault: removing `"cnt"` from `src/config.py`'s FORBIDDEN set.

[Step 3] Running tests...
FAILED tests/test_leakage.py::test_no_forbidden_features ... (hook captured stderr)

[Step 4] Writing entry to docs/SEEDED_FAULTS.md.

>>> Fault is in place. Inspect and commit. When ready, say "/ok continue" to restore.
```
