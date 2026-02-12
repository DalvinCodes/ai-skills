# Gate Revise Prompt Template

Use this template when the PE finds issues at a gate and needs the Feature Agent to revise before re-submitting for review. Resume the same subagent session using `task_id`.

```
Task tool (general-purpose):
  task_id: [PREVIOUS_TASK_ID — resume the Feature Agent session]
  description: "Revise Feature: [FEATURE_NAME] — Gate [N] Revision Required"
  prompt: |
    ## Gate [N] Review: REVISE

    Your Gate [N] ([GATE_NAME]) has been reviewed by the Principal Engineer.
    **Revisions are required** before you can proceed.

    **Session:** [SESSION_ID]
    **Feature:** [FEATURE_NAME]

    ### Issues Found

    [ISSUES — numbered list with specific file:line references where applicable]

    1. [ISSUE_1 — e.g., "`src/auth/handler.ts:45` — `validateToken` returns
       hardcoded `true` instead of real validation logic"]
    2. [ISSUE_2 — e.g., "`tests/auth.test.ts:23` — Test mocks the entire auth
       service, never exercises real validation path"]
    3. [ISSUE_3 — e.g., "Missing: No error handling for expired tokens
       (acceptance criterion #3)"]
    4. [ISSUE_4 — e.g., "Sub-agent review skipped: `sub-agents/test-gen-01/reviewed/`
       is empty but `output/` has files"]
    5. [ISSUE_5 — e.g., "Contract violation: API implementation returns `{ user_id }`
       but `contracts/api-contract.md` specifies `{ userId }`"]

    ### Required Changes

    [CHANGES — specific, actionable items the Feature Agent must do]

    - [CHANGE_1]
    - [CHANGE_2]
    - [CHANGE_3]

    ### Re-Gate Criteria

    You may re-submit Gate [N] when ALL of the following are true:

    [CRITERIA — what must be true before the agent reports back]

    - [CRITERION_1 — e.g., "All issues above are resolved"]
    - [CRITERION_2 — e.g., "Tests pass with real implementations, no new stubs"]
    - [CRITERION_3 — e.g., "No new TODO/FIXME markers introduced"]
    - [CRITERION_4 — e.g., "All sub-agent output reviewed and in reviewed/ dirs"]
    - [CRITERION_5 — e.g., "Implementation matches contracts in shared/coordination/"]

    ### Instructions

    1. Update `state.md`: phase=[CURRENT_PHASE], status=revising
    2. Address each issue listed above
    3. Update relevant `.bastion/[SESSION_ID]/[FEATURE_SLUG]/` artifacts with revision notes
    4. Log any issues encountered during revision to `debugging-log.md`
    5. When all re-gate criteria are met, submit Gate [N] report again
    6. **DO NOT proceed to the next phase** — you must pass this gate first

    ### What NOT to Do

    - Do not skip any issue — all must be addressed
    - Do not replace real implementations with different stubs
    - Do not mark issues as "won't fix" without PE approval
    - Do not proceed past this gate without re-approval
    - Do not present raw sub-agent output — review it first
    - Do not change contracts unilaterally — coordinate with other components
```

## Usage

The PE fills in specific issues, required changes, and re-gate criteria. This template
ensures the Feature Agent knows exactly what to fix and when it's allowed to re-submit.

**Example:**

```
Task tool:
  task_id: "ses_abc123"
  description: "Revise Feature: Dark Mode — Gate 3 Revision Required"
  prompt: |
    ## Gate 3 Review: REVISE

    Revisions required before completion.

    **Session:** 2026-02-10-dark-mode-001
    **Feature:** Dark Mode

    ### Issues Found

    1. `src/components/ThemeToggle.tsx:12` — Toggle component uses
       hardcoded `true` for dark mode check instead of reading from
       CSS custom property
    2. `tests/theme.test.ts:34-45` — Tests mock `window.matchMedia`
       but never test the actual CSS variable switching logic
    3. Missing: No persistence — theme preference resets on page reload
       (acceptance criterion #4)
    4. Sub-agent review gap: `sub-agents/icon-set-01/reviewed/` is empty
       — raw output was used directly

    ### Required Changes

    - Implement real CSS custom property reading in ThemeToggle
    - Add integration test that verifies CSS variables actually change
    - Add localStorage persistence for theme preference
    - Add test for persistence across page loads
    - Review sub-agent icon-set-01 output and store fixed version in reviewed/

    ### Re-Gate Criteria

    - All 4 issues resolved with real implementations
    - Tests pass — no .skip or mocked-out assertions
    - Theme persists across simulated page reload in tests
    - No new TODO/FIXME markers
    - Sub-agent reviewed/ directory populated with reviewed work

    Address these issues and re-submit Gate 3.
```
