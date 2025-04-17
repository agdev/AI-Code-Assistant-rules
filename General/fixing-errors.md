# Debugging Workflow: Fixing Runtime Errors

This document outlines a systematic approach to debugging runtime errors encountered during manual testing or reported by users.

## 1. Define the Problem

- **Symptom:** Clearly describe the incorrect behavior observed (e.g., "Data doesn't persist after refresh", "Error message 'X' appears when performing action Y", "UI hangs during operation Z").
- **Context:** Note the specific steps, user inputs, and application state required to reliably reproduce the error.

## 2. Formulate Hypotheses

- Based on the symptom and application architecture, list potential root causes. Consider common areas:
    - **Logic Errors:** Flaws in component, function, or service logic.
    - **State Management:** Incorrect state updates, race conditions, stale state.
    - **Data Handling:** Issues with data fetching, persistence (storage, API), validation, or mapping/transformation.
    - **Asynchronous Operations:** Unhandled promises, incorrect `async/await` usage, race conditions.
    - **Argument Mismatches:** Functions called with incorrect argument types or structures.
    - **Environment Issues:** Browser quirks, dependency conflicts, build problems.
    - **Third-Party Libraries:** Unexpected behavior or conflicts with external libraries.

## 3. Gather Evidence with Logging & Tools

- **Targeted Logging:** Add `console.log` (or equivalent) statements at critical points in the suspected code path.
    - Log function entry/exit points.
    - Log arguments received by functions (including `typeof` checks).
    - Log data before and after transformations or persistence operations.
    - Log relevant state variables before and after updates.
    - Log data immediately before it's used in a failing operation.
- **Browser DevTools:** Utilize the browser's developer tools extensively:
    - **Console:** Examine error messages, stack traces, and custom logs.
    - **Network Tab:** Inspect API request/response payloads and statuses.
    - **Application/Storage Tab:** Check `localStorage`, `sessionStorage`, cookies, IndexedDB for expected data.
    - **Debugger:** Set breakpoints to step through code execution and inspect variable values.
- **Reproduce & Analyze:** Systematically reproduce the error while observing logs and DevTools to pinpoint where the actual behavior deviates from the expected behavior.

## 4. Inspect Code Logic

- Use the evidence gathered to identify the specific functions, components, or modules involved.
- Use `read_file` or IDE tools to examine the relevant code implementations.
- **Verify:**
    - Does the code logic align with its intended purpose?
    - Are variables, arguments, and state handled correctly (types, null checks, structure)?
    - Are asynchronous operations managed correctly?
    - Are external services or APIs being called with the correct parameters?
    - Are data transformations/mappings accurate?
    - Are component lifecycle methods or hooks (`useEffect`, etc.) behaving as expected (check dependency arrays)?

## 5. Check Data Integrity (If Applicable)

- If the issue involves persisted data:
    - Directly inspect the storage mechanism (browser storage, database) *before* and *after* the operation that triggers the error.
    - Look for missing fields, incorrect data types, or corrupted entries.
    - **Consider clearing relevant storage** as a diagnostic step to rule out issues caused by stale or invalid data.

## 6. Test-Driven Fix (TDD Approach)

- **Write/Enhance Failing Test:** Before applying a code fix, write a new unit/integration test (or modify an existing one) that specifically reproduces the bug.
    - Ensure mocks accurately represent dependencies.
    - Set up the specific initial state/props required to trigger the bug.
    - Assert the expected outcome (e.g., correct function arguments, correct final state, absence of errors).
- **Verify Failure:** Run the test and confirm that it fails for the expected reason.
- **Apply Fix:** Implement the code change intended to resolve the bug.
- **Verify Success:** Run the test again and confirm it now passes. Ensure other related tests still pass.

## 7. Manual Verification

- After unit/integration tests pass, perform the original manual test case again in the application (clearing storage/cache if necessary) to confirm the symptom is resolved.
- Test related functionality to check for any unintended regressions.

## 8. Cleanup

- Once the fix is confirmed via manual testing, remove the temporary `console.log` statements and any other diagnostic code added during debugging.
- Commit the fix with a clear message describing the bug and the solution.