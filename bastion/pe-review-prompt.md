# PE Gate Review Prompt Template

Use this as a self-prompt when you (the PE/controller) reach a gate and need to review
the Feature Agent's work. This is NOT dispatched as a subagent — it's a checklist for
YOU to follow.

## Gate Review Checklist

### 1. Read Artifacts

```
Read all files in .bastion/<session-id>/<feature>/<current-phase>/
Read .bastion/<session-id>/<feature>/state.md
Read .bastion/<session-id>/<feature>/handoff/context.md
```

For component teams, also read:
```
Read .bastion/<session-id>/<feature>/shared/coordination/contracts/
Read .bastion/<session-id>/<feature>/shared/coordination/dependencies.md
Read each component's handoff/consolidation.md
```

Understand what the Feature Agent did, what decisions it made, and why.

### 2. Verify Against Scope

Compare work against `.bastion/<session-id>/<feature>/agent-card.md` (or `feature-brief.md` for component teams):

- [ ] Does the work match the feature scope? (no scope creep)
- [ ] Are acceptance criteria being addressed?
- [ ] Are constraints being respected?
- [ ] Did the agent stay within the agreed approach?

### 3. Three-Tier Validation Check (Component Teams)

For component teams, verify the validation chain was followed:

```
Sub-agent output → Feature Agent reviewed → PE validates
```

- [ ] Sub-agent work is in `reviewed/` directories (not just `output/`)
- [ ] Each sub-agent task has `notes.md` documenting review
- [ ] Feature Agent's `consolidation.md` shows how sub-agent work was integrated
- [ ] No raw sub-agent output presented as final work
- [ ] Feature Agent fixed issues found during review (documented in notes.md)

**Quick check:** If `reviewed/` is empty but `output/` has files, the Feature Agent skipped review. **Automatic REVISE.**

### 4. Contract Verification (Component Teams)

Verify implementations match agreed interfaces:

- [ ] All contracts in `shared/coordination/contracts/` are current
- [ ] Each component's implementation matches contract specifications:
  - API endpoints match agreed routes, methods, request/response shapes
  - Database schema matches agreed data models
  - Frontend calls match API contract
- [ ] No unilateral contract changes (check announcements for breaking-change notices)
- [ ] Dependencies in `shared/coordination/dependencies.md` are all resolved or tracked

**Verification method:**
```bash
# For each contract, grep the implementation for expected interface shapes
# Example: If api-contract.md specifies POST /auth/login with { email, password }
grep -rn "auth/login\|/login" <api-implementation-files>
# Verify the handler signature matches
```

### 5. Phase-Specific Checks

#### Gate 1: Design Review
- [ ] Exploration was thorough — relevant files identified
- [ ] Dependencies mapped correctly
- [ ] Approach is sound and justified
- [ ] Alternatives were genuinely considered (not strawmen)
- [ ] Gotchas identified make sense
- [ ] Design doesn't over-engineer (YAGNI)
- [ ] **Component teams:** Contracts proposed and agreed between components

#### Gate 2: Plan Review
- [ ] Plan follows writing-plans format (bite-sized TDD steps)
- [ ] Every task has specific validation criteria
- [ ] Exact file paths specified
- [ ] Code in plan is complete (not "add validation here")
- [ ] Tasks are ordered correctly (dependencies respected)
- [ ] Plan covers ALL acceptance criteria
- [ ] No unnecessary tasks (YAGNI)
- [ ] **Component teams:** Plans account for cross-component dependency ordering

#### Gate 3: Completion Review (and any implementation gates)
- [ ] All tasks marked complete in `implementation/progress.md`
- [ ] All validation results documented and passing

**Anti-Stub Scan — run these commands on changed files:**

```bash
# Scan for fake implementations
grep -rn "TODO\|FIXME\|not implemented\|placeholder" <changed-files>

# Scan for stubbed returns
grep -rn "return true\|return false\|return null\|return \[\]\|return {}" <changed-files>

# Scan for skipped tests
grep -rn "\.skip\|\.only\|xit\|xdescribe\|@Ignore\|pending\|test\.todo" <test-files>

# Scan for weak assertions
grep -rn "expect(true)\|expect(1)\|toBeTruthy()\|assert True" <test-files>

# Scan for broad mocks hiding real logic
grep -rn "jest\.mock\|vi\.mock\|mock\.\|stub\.\|sinon\." <test-files>
```

- [ ] No TODO/FIXME markers in final code
- [ ] No hardcoded return values standing in for real logic
- [ ] No skipped or pending tests
- [ ] No weak assertions that would pass regardless
- [ ] Mocks are limited to true external boundaries (network, DB), not internal logic

**Code Quality:**
- [ ] Functions have real logic (not one-liners returning constants)
- [ ] Error handling present and meaningful
- [ ] Edge cases handled
- [ ] Tests would FAIL if implementation were wrong
- [ ] Tests exercise real code paths (not just mocked wrappers)

### 6. Run Tests Independently

```bash
# Run the project's test suite — don't trust the agent's report
npm test / pytest / cargo test / go test ./...
```

- [ ] Tests pass when YOU run them
- [ ] No flaky or timing-dependent tests

### 7. Check for Cross-Feature Conflicts (Concurrent Mode Only)

If running multiple Feature Agents:

```bash
# Compare files modified by this feature vs other active features
# Check .bastion/<session-id>/*/implementation/progress.md for file lists
```

- [ ] No overlapping file modifications across active features
- [ ] If overlap found: flag to user, decide priority

### 8. Render Verdict

Based on the above:

**Approve** — All checks pass. Use `./gate-resume-prompt.md` to continue.

**Revise** — Issues found. Use `./gate-revise-prompt.md` with:
- Specific issues (file:line references)
- Required changes
- Re-gate criteria

**Redirect** — Approach is wrong. Provide new direction, agent re-does phase.

**Abort** — Feature should be abandoned. Cleanup `.bastion/<session-id>/<feature>/`.

## Structured Revision Feedback

```markdown
## Gate Review: REVISE

### Issues Found
1. `file:line` — description
2. Sub-agent review skipped for [subtask-id] — output/ has files but reviewed/ is empty
3. Contract violation — [component] implementation doesn't match [contract-file]

### Required Changes
- Specific fix required
- Review and fix sub-agent output for [subtask-id]
- Update implementation to match contract OR update contract with team agreement

### Re-gate When
All issues resolved, tests pass, no new stubs, contracts verified
```

## Quick Reference

| Check | Gate 1 | Gate 2 | Gate 3 |
|-------|--------|--------|--------|
| Scope compliance | Yes | Yes | Yes |
| Exploration quality | Yes | -- | -- |
| Design soundness | Yes | -- | -- |
| Contract agreement | Yes | Yes | Yes |
| Plan completeness | -- | Yes | -- |
| Sub-agent review chain | -- | -- | Yes |
| Anti-stub scan | -- | -- | Yes |
| Test independence | -- | -- | Yes |
| Code quality | -- | -- | Yes |
| Contract verification | -- | -- | Yes |
| Cross-feature conflicts | Yes | Yes | Yes |
| Run tests yourself | -- | -- | Yes |

## Common PE Mistakes

**Auto-approving:** Reading the report summary without checking actual artifacts.
Fix: Always read the actual files, not just the report.

**Trusting test claims:** Agent says "all tests pass" but they don't.
Fix: Run tests yourself at Gate 3.

**Missing stubs:** Agent wrote clean code everywhere except one stubbed function.
Fix: Run the anti-stub scan commands — they catch what eyes miss.

**Scope creep tolerance:** Agent added "nice to have" features not in scope.
Fix: Compare strictly against agent-card.md acceptance criteria.

**Skipping sub-agent review check:** Feature Agent says "all sub-agent work reviewed" but `reviewed/` directories are empty.
Fix: Always check the `sub-agents/*/reviewed/` vs `sub-agents/*/output/` directories.

**Ignoring contract mismatches:** Components implement different versions of the same interface.
Fix: Verify each component's implementation against `shared/coordination/contracts/`.
