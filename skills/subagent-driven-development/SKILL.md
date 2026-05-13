---
name: subagent-driven-development
description: "Execute plans via delegate_task subagents with active orchestrator oversight — progress tracking, result review, and intervention."
version: 2.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [delegation, subagent, implementation, workflow, parallel, orchestration]
    related_skills: [writing-plans, requesting-code-review, test-driven-development]
---

# Subagent-Driven Development

## Overview

Execute implementation plans by dispatching fresh subagents per task. The main model acts as an **active orchestrator** — not just a task router, but a supervisor who tracks every subagent's progress, diagnoses problems, reviews all results personally, and ensures quality end-to-end.

**Core principle:** Subagents do the work. The main model ensures it's done right — by tracking progress, intervening when subagents get stuck, reviewing every result, and doing the final integration check.

## When to Use

Use this skill when:
- You have an implementation plan (from writing-plans skill or user requirements)
- Tasks are mostly independent
- Quality and spec compliance are important
- You want the main model to actively supervise execution

**vs. fire-and-forget delegation:**
- Main model tracks each subagent's progress in real time
- Main model reviews outputs personally (not just delegating review to other subagents)
- Problems are diagnosed and fixed immediately, not discovered later
- Final quality gate is the main model's own review

## The Orchestrator's Three Responsibilities

As the main model (orchestrator), you have three active responsibilities beyond dispatching subagents:

### Responsibility 1: Progress Tracking & Intervention

During subagent execution:
- Read subagent output carefully as it arrives — do NOT treat it as a black box
- Watch for signs the subagent is stuck: repeated errors, circular reasoning, giving up, producing vague output
- When a subagent has problems, diagnose the root cause and provide specific guidance
- Re-dispatch with corrected context — don't let a stuck subagent waste iterations

### Responsibility 2: Per-Task Result Review

After each subagent completes a task:
- Review the output against the original task spec — did it deliver what was asked?
- Check quality: code correctness, error handling, edge cases, test coverage
- If results are poor: identify root cause, create a fix plan, re-dispatch subagent to fix
- Only mark complete when YOU are satisfied — not when the subagent claims it's done

### Responsibility 3: Final Integration Review

After ALL tasks complete:
- Review the entire implementation holistically
- Check cross-task consistency: do components work together? naming conventions match? no duplicated code?
- Run the full test suite yourself
- Verify against original requirements
- Fix any remaining issues before declaring done

## The Process

### Phase 1: Prepare

#### Step 1: Read and Parse Plan

Read the plan file ONCE. Extract ALL tasks with their full text and context upfront. Create a todo list:

```
# Read the plan
read_file("docs/plans/feature-plan.md")

# Create todo list with all tasks
todo([
    {"id": "task-1", "content": "Create User model with email field", "status": "pending"},
    {"id": "task-2", "content": "Add password hashing utility", "status": "pending"},
    {"id": "task-3", "content": "Create login endpoint", "status": "pending"},
])
```

**Key:** Read the plan ONCE. Provide full task text in subagent context — don't make subagents read the plan file.

### Phase 2: Execute Tasks (one at a time)

For EACH task, follow this cycle:

#### Step 1: Dispatch Implementer Subagent

Use `delegate_task` with complete context. The subagent should have everything it needs — task spec, file paths, exact commands, expected output, project conventions.

```
delegate_task(
    goal="Implement Task 1: Create User model...",
    context="""
    TASK FROM PLAN:
    [full task text from plan, not a reference to the plan file]

    REQUIREMENTS:
    - Exact files to create/modify
    - Expected behavior
    - Testing requirements

    PROJECT CONTEXT:
    - Language, framework, conventions
    - Existing code structure
    - How to verify success
    """,
    toolsets=['terminal', 'file']
)
```

#### Step 2: Main Model Reviews Subagent Output

**This is your most important step.** Do NOT skip it. Do NOT delegate it to another subagent.

When the subagent returns, read its output carefully:

1. **Spec Compliance Check**
   - Did it create/modify all the files listed in the task spec?
   - Does the implementation match the expected behavior?
   - Any scope creep (added things not in the spec)?
   - Any missing requirements?

2. **Quality Check**
   - Code looks correct and follows project conventions?
   - Proper error handling?
   - Tests exist and are meaningful?
   - No obvious bugs?

3. **Evidence Check**
   - Did the subagent actually run tests? What were the results?
   - Did it verify the output works?
   - Are there any warnings or errors in its output?

If the subagent's output is vague or missing details, use `read_file` to inspect the actual files it created. Trust but verify.

#### Step 3: Diagnose and Fix Issues (if needed)

**If the result is good:** Go to Step 4.

**If the result has issues:** Do NOT just re-dispatch with "try again." Diagnose first:

1. **Identify the root cause:**
   - Did the subagent misunderstand the task? → Clarify the spec in re-dispatch context
   - Did it hit a technical obstacle? → Provide the solution directly
   - Did it skip steps? → Tell it explicitly what steps it missed
   - Is the task spec itself wrong? → Fix the spec first, then re-dispatch

2. **Create a fix plan** (3-5 bullet points max):
   ```
   Fix plan for Task 3:
   - The login endpoint returns 200 instead of 401 for invalid credentials → fix response code
   - Missing rate limiting as specified → add rate limiter
   - Test only covers happy path → add test for invalid credentials
   ```

3. **Re-dispatch with specific fix instructions:**
   ```
   delegate_task(
       goal="Fix Task 3: Login endpoint has 3 issues to resolve",
       context="""
       PREVIOUS ATTEMPT HAD THESE ISSUES:
       1. Response code: returns 200 instead of 401 for bad credentials
          FIX: Change return to 401 with {"error": "Invalid credentials"}
       2. Missing rate limiting (spec requires 5 attempts/min)
          FIX: Add Flask-Limiter decorator to /login route
       3. Missing negative test case
          FIX: Add test_login_invalid_credentials in tests/test_auth.py

       ORIGINAL TASK CONTEXT:
       [include original task context again]

       After fixing, run: pytest tests/test_auth.py -v
       Expected: 5 passed
       """,
       toolsets=['terminal', 'file']
   )
   ```

4. **After re-dispatch, review again** (back to Step 2). Repeat until satisfied.

**Important:** If a subagent fails the same task 3 times, escalate to the user. Don't loop forever.

#### Step 4: Mark Task Complete

Only when YOU are satisfied with the result:
```
todo([{"id": "task-1", "content": "...", "status": "completed"}], merge=True)
```

### Phase 3: Final Integration Review

After ALL tasks are marked complete, the main model does a comprehensive review:

#### 1. Holistic Code Review

Read through all changed files as a whole:
- Do components fit together correctly?
- Consistent naming conventions across tasks?
- No duplicated code between tasks?
- Imports and dependencies are correct?

```
# Review all changes
search_files(pattern=".*", target="files", path="src/", file_glob="*.py")
read_file("src/new_module.py")  # spot-check key files
```

#### 2. Full Test Suite

Run the complete test suite yourself:
```
pytest tests/ -v
```

If tests fail, diagnose and fix — don't delegate. By this point, failures are usually integration issues that need your holistic understanding.

#### 3. Requirements Verification

Go back to the original requirements/plan and verify each one:
- [ ] Feature A works as described
- [ ] Feature B handles edge cases
- [ ] All acceptance criteria met

#### 4. Final Cleanup

```
# Review diff
git diff --stat
git diff  # spot-check key changes

# Commit if all good
git add -A && git commit -m "feat: complete [feature] implementation"
```

#### 5. Report to User

Summarize what was built, any issues encountered and resolved, and any decisions made. Include:
- Tasks completed
- Any deviations from the plan (and why)
- Test results
- Known limitations (if any)

### Phase 4: Progress Tracking During Batch Execution

When dispatching multiple subagents in parallel (batch mode with `tasks` array):

```
delegate_task(
    tasks=[
        {"goal": "Task 1: ...", "context": "...", "toolsets": [...]},
        {"goal": "Task 2: ...", "context": "...", "toolsets": [...]},
    ]
)
```

After results come back:
1. Review each subagent's output individually (Phase 2, Step 2)
2. Identify which tasks passed and which need fixes
3. Fix failed tasks one at a time
4. Note: batch results are summaries — use `read_file` to verify actual file contents

## Detecting and Handling Subagent Problems

### Signs a Subagent Is Stuck

Watch for these patterns in subagent output:
- **Circular reasoning:** Repeating the same analysis without making progress
- **Tool call loops:** Calling the same tool with the same parameters repeatedly
- **Giving up:** "I cannot complete this task" without a specific technical reason
- **Vague output:** "Implemented the feature" with no file paths, no test results
- **Partial completion:** Claims done but task spec lists 5 requirements and only 2 are addressed
- **Silent failure:** Returns success but files don't exist or tests fail

### How to Intervene

When you detect a stuck subagent:

1. **Stop and diagnose:** Don't let it continue. Read its output to understand what went wrong.

2. **Identify the specific blocker:**
   - Missing information? → Provide it in the re-dispatch context
   - Wrong approach? → Give a clear direction: "Instead of X, try Y because Z"
   - Technical error? → Provide the exact fix: "Add `import X` at line 3"
   - Tool limitation? → Work around it: mention alternative tools or approaches

3. **Re-dispatch with corrected context:**
   ```
   delegate_task(
       goal="Retry Task 3 with corrected approach",
       context="""
       PREVIOUS ISSUE: [specific description of what went wrong]
       ROOT CAUSE: [why it happened]
       SOLUTION: [what to do differently]

       [Original task context]
       """,
       toolsets=[...]
   )
   ```

4. **If the same subagent fails 3 times on the same task:** Escalate to user with:
   - What the task is
   - What's been tried
   - What the blocker is
   - Options: change approach, split task, skip for now

### Specific Problem Patterns and Solutions

| Problem | Root Cause | Solution |
|---------|-----------|----------|
| Subagent claims done but no files created | It reasoned about code but never wrote it | Re-dispatch: "Write the files using write_file tool. Do not just describe the code." |
| Subagent writes code but doesn't test | It skipped the testing step | Re-dispatch: "Run the tests before reporting done. Include test output in your response." |
| Subagent gets wrong error and gives up | It misread the error message | Point out the actual error: "The error says X, which means Y. Fix by doing Z." |
| Subagent produces working but ugly code | It prioritized speed over quality | Re-dispatch: "Refactor for readability. Apply [specific conventions]." |
| Subagent misses edge cases | Task spec wasn't explicit enough | Add edge cases to re-dispatch context: "Also handle: empty input, null values, max length" |

## Task Granularity

**Each task = 2-5 minutes of focused work.**

**Too big:**
- "Implement user authentication system"

**Right size:**
- "Create User model with email and password fields"
- "Add password hashing function"
- "Create login endpoint"
- "Add JWT token generation"
- "Create registration endpoint"

## Red Flags — Never Do These

- Start implementation without a plan
- Skip your review of subagent output (Step 2 is NOT optional)
- Delegate review to another subagent instead of reviewing yourself
- Proceed with unfixed critical issues
- Dispatch multiple implementation subagents for tasks that touch the same files
- Make subagent read the plan file (provide full text in context instead)
- Ignore subagent questions (answer before letting them proceed)
- Accept "close enough" on spec compliance
- Let subagent self-report substitute for your own verification
- Move to next task while current task has open issues
- Skip the final integration review
- Re-dispatch a failing subagent without diagnosing why it failed first
- Loop more than 3 times on the same task without escalating
- Set delegation.provider to a built-in provider and rely on delegation.api_key (built-in providers read env vars, not config api_key — use custom_providers instead; see references/delegation-config.md)
- Change delegation config and skip gateway restart (config changes need gateway restart to take effect)

## Integration with Other Skills

### With writing-plans

This skill EXECUTES plans created by the writing-plans skill:
1. User requirements → writing-plans → implementation plan
2. Implementation plan → subagent-driven-development → reviewed, working code

### With test-driven-development

Implementer subagents should follow TDD:
1. Write failing test first
2. Implement minimal code
3. Verify test passes
4. Commit

Include TDD instructions in every implementer context. When reviewing, verify the TDD cycle was followed.

### With systematic-debugging

If a subagent encounters bugs during implementation, and your diagnosis needs deeper analysis:
1. Apply systematic-debugging principles to find root cause
2. Provide the root cause and fix to the subagent in re-dispatch context
3. Don't expect subagents to do systematic debugging themselves — you do the diagnosis

## Efficiency Notes

**Why fresh subagent per task:**
- Prevents context pollution from accumulated state
- Each subagent gets clean, focused context
- No confusion from prior tasks' code or reasoning

**Why main model reviews (not subagent reviewers):**
- You have the full picture (original requirements, plan, inter-task dependencies)
- You can catch integration issues that a per-file reviewer would miss
- You're accountable for final quality
- Subagent reviewers add latency without adding value when you're already reading outputs

**Cost trade-off:**
- Main model reads subagent outputs → more tokens for orchestrator
- But catches issues early → fewer re-dispatches overall
- Better to spend tokens on review than on fixing compounded problems

**Context budget awareness:**
- When context is tight (DEGRADING or POOR tier), prioritize: check spec compliance and test results. Defer deep code quality review to final integration phase.
- See `references/context-budget-discipline.md` for tier thresholds.

## Example Workflow

```
[Read plan: docs/plans/auth-feature.md]
[Create todo list with 3 tasks]

━━━ Task 1: Create User model ━━━

[Dispatch implementer subagent]
  Subagent returns: "Created src/models/user.py and tests/models/test_user.py. 3/3 tests passing."

[MAIN MODEL REVIEWS]
  Read: src/models/user.py — looks good, fields match spec
  Read: tests/models/test_user.py — tests cover basic cases
  Verdict: ✅ PASS

[Mark Task 1 complete]

━━━ Task 2: Password hashing ━━━

[Dispatch implementer subagent]
  Subagent returns: "Added hash_password() in src/utils/auth.py. Tests in tests/utils/test_auth.py."

[MAIN MODEL REVIEWS]
  Read: src/utils/auth.py — wait, it uses MD5, spec says bcrypt
  Read: tests/utils/test_auth.py — only 1 test, no edge cases
  Verdict: ❌ Issues found

[DIAGNOSE]
  Root cause: Subagent defaulted to MD5 instead of using bcrypt as specified.
  Also missed edge case tests.

[Re-dispatch with fix instructions]
  delegate_task(
      goal="Fix password hashing: use bcrypt + add edge case tests",
      context="""
      ISSUES TO FIX:
      1. Replace MD5 with bcrypt — spec explicitly requires bcrypt
         - Import bcrypt, use bcrypt.hashpw() and bcrypt.checkpw()
      2. Add test cases: empty password, unicode password, long password
         - Add to tests/utils/test_auth.py

      ORIGINAL TASK: [full context]
      """
  )

  Subagent returns: "Fixed. Now using bcrypt. 6/6 tests passing."

[MAIN MODEL REVIEWS]
  Read: src/utils/auth.py — uses bcrypt correctly ✅
  Read: tests/utils/test_auth.py — edge cases covered ✅
  Verdict: ✅ PASS

[Mark Task 2 complete]

━━━ Task 3: Login endpoint ━━━

[Dispatch implementer subagent]
  Subagent returns: "Created /login endpoint in src/routes/auth.py. 4/4 tests passing."

[MAIN MODEL REVIEWS]
  Read: src/routes/auth.py — correct, clean
  Run: pytest tests/routes/test_auth.py -v — all pass
  Verdict: ✅ PASS

[Mark Task 3 complete]

━━━ FINAL INTEGRATION REVIEW ━━━

[MAIN MODEL REVIEWS ALL WORK]
  - All 3 tasks' code fits together: User model → hash utility → login endpoint ✅
  - Naming consistent across files ✅
  - No duplicated code ✅
  - Full test suite: pytest tests/ -v → 13 passed ✅
  - Requirements: all acceptance criteria met ✅

[Commit]
  git add -A && git commit -m "feat: complete auth system"

[Report to user]
  "Auth system complete. 3 tasks, 13 tests.
   Task 2 needed one fix (MD5→bcrypt). All passing."
```

## Remember

```
Main model tracks progress — don't fire-and-forget
Main model reviews every result — don't delegate review
Diagnose before re-dispatching — don't say "try again"
Fix root cause, not symptoms
Three strikes and escalate
Final review is YOUR responsibility
Quality is not an accident — it's the result of active supervision
```

## Further reading (load when relevant)

- **`references/delegation-config.md`** — How to configure subagent model, provider, and API keys. Built-in vs custom provider pitfalls, env var requirements, gateway restart.
- **`references/context-budget-discipline.md`** — Four-tier context degradation model, read-depth rules, early warning signs. Load when orchestrating large multi-phase plans.
- **`references/gates-taxonomy.md`** — Four canonical gate types (Pre-flight, Revision, Escalation, Abort). Use this vocabulary when designing review checkpoints.
