# Gate Resume Prompt Template

Use this template when the PE **approves** a gate and the Feature Agent should continue to the next phase. Resume the same subagent session using `task_id`.

```
Task tool (general-purpose):
  task_id: [PREVIOUS_TASK_ID — resume the Feature Agent session]
  description: "Resume Feature: [FEATURE_NAME] — Gate [N] Approved"
  prompt: |
    ## Gate [N] Approved

    Your Gate [N] ([GATE_NAME]) has been reviewed and **approved** by the
    Principal Engineer.

    **Session:** [SESSION_ID]
    **Feature:** [FEATURE_NAME]

    ### PE Notes

    [PE_NOTES — any guidance, observations, or minor suggestions from the PE.
    If none, write "No additional notes. Proceed as planned."]

    ### What Was Approved

    [BRIEF_SUMMARY — 1-2 sentences confirming what the PE reviewed and accepted]

    ### Next Phase

    You are now cleared to proceed to **Phase [NEXT_PHASE_NUMBER]: [NEXT_PHASE_NAME]**.

    **Reminder:**
    - Update `state.md` immediately: phase=[NEXT_PHASE], status=in-progress
    - Continue writing to `.bastion/[SESSION_ID]/[FEATURE_SLUG]/` throughout
    - Your next gate is **Gate [NEXT_GATE_NUMBER]: [NEXT_GATE_NAME]**
    - Stop at that gate and report — do not proceed past it without approval
    - Poll `shared/coordination/` before cross-component decisions (component teams)

    [IF NEXT_PHASE IS PLAN]
    **REQUIRED:** Use the writing-plans skill to create your implementation plan.
    Save to `.bastion/[SESSION_ID]/[FEATURE_SLUG]/plan/implementation.md`.
    [END IF]

    [IF NEXT_PHASE IS IMPLEMENT]
    **REQUIRED:** Use test-driven-development for each task.
    Every task must have executed validation before marking complete.
    No mocks/stubs as final code.
    Log all issues to `implementation/debugging-log.md`.
    Commit after each task.
    If you spawn sub-agents, review ALL their output before integrating.
    [END IF]

    [IF NEXT_PHASE IS COMPLETE]
    Run full test suite, self-review, and write comprehensive handoff docs.
    For component teams: write consolidation.md showing how sub-agent work was integrated.
    This is your final gate — make the handoff thorough.
    [END IF]

    Proceed.
```

## Usage

The PE fills in the template values and dispatches using the **same `task_id`** from the
original Feature Agent dispatch. This resumes the persistent session, preserving all
context the Feature Agent has accumulated.

**Example:**

```
Task tool:
  task_id: "ses_abc123"
  description: "Resume Feature: Dark Mode — Gate 1 Approved"
  prompt: |
    ## Gate 1 Approved

    Your Gate 1 (Design Review) has been reviewed and **approved**.

    **Session:** 2026-02-10-dark-mode-001
    **Feature:** Dark Mode

    ### PE Notes
    Good approach. Consider using CSS custom properties for theme variables
    instead of a context-based approach — it'll be simpler for the existing
    component structure.

    ### What Was Approved
    Your exploration was thorough and the design approach using CSS custom
    properties with a ThemeProvider wrapper is accepted.

    ### Next Phase
    Proceed to Phase 3: Plan.
    Use the writing-plans skill. Your next gate is Gate 2: Plan Review.

    Proceed.
```
