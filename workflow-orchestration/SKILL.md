---
name: workflow-orchestration
description: Orchestrates structured task execution with planning, progress tracking, self-improvement, and verification gates. Use at the start of any coding session, when given a non-trivial task (3+ steps or architectural decisions), when corrected by the user, when fixing bugs, or when the user mentions "plan", "todo", "track progress", "verify", "lessons learned", "fix this bug", or "task management". Applies to all coding workflows where structured methodology improves quality and reduces mistakes.
metadata:
  author: Martin Clasen
  version: 1.0.0
  category: workflow
  tags: [planning, task-management, self-improvement, verification, bug-fixing]
---

# Workflow Orchestration

Skill for structured task execution across coding sessions. Ensures consistent planning, progress tracking, quality verification, and continuous self-improvement.

## How It Works

Every non-trivial task follows a lifecycle: **Plan → Execute → Verify → Learn**. Trivial tasks (1-2 obvious steps) skip planning overhead and execute directly.

The two persistent files that track state across sessions:
- `tasks/todo.md` — current plan with checkable items and review notes
- `tasks/lessons.md` — accumulated patterns from past corrections

## Task Lifecycle

### Step 1: Assess Complexity

Before starting any task, determine if it is trivial or non-trivial.

**Trivial** (execute directly): single-file rename, add a comment, run a command, fix a typo.

**Non-trivial** (follow full lifecycle): 3+ steps, architectural decisions, multi-file changes, ambiguous requirements, or anything where a wrong first step wastes significant effort.

When in doubt, treat it as non-trivial. The overhead of a quick plan is far less than the cost of reworking a bad approach.

### Step 2: Plan (non-trivial tasks)

Enter plan mode and write a structured plan to `tasks/todo.md` before writing any code.

A good plan contains:
- Checkable items (`- [ ]`) at a granularity where each item is independently verifiable
- Dependencies between items made explicit
- Acceptance criteria for the overall task

```markdown
# Task: Add user authentication

## Plan
- [ ] Choose auth strategy (JWT vs session) — document decision
- [ ] Add auth middleware
- [ ] Create login/register endpoints
- [ ] Protect existing routes
- [ ] Add tests for auth flow
- [ ] Verify end-to-end with manual test

## Acceptance criteria
- Unauthenticated requests to protected routes return 401
- Login returns valid token
- Token expires after configured TTL
```

Present the plan to the user before starting implementation. This catches misunderstandings early when they are cheap to fix.

### Step 3: Execute with Progress Tracking

Work through the plan item by item. Mark each item complete in `tasks/todo.md` as you go. Provide a high-level summary of what changed at each step — enough for the user to follow without reading diffs.

If something goes sideways during execution, stop and re-plan immediately. Continuing down a broken path wastes effort and pollutes the codebase. Update `tasks/todo.md` with the revised approach.

### Step 4: Verify Before Done

Never mark a task complete without proving it works. Verification methods depend on the task:

- **Code changes**: run tests, check linter, demonstrate the behavior
- **Bug fixes**: reproduce the original bug, show it is fixed, confirm no regressions
- **Refactors**: diff behavior between before and after — output should be identical

The quality bar: "Would a staff engineer approve this?" If the answer is uncertain, the task is not done.

Add a review section to `tasks/todo.md` documenting what was verified and how.

```markdown
## Review
- All tests pass (12/12)
- Manual test: login returns JWT, protected route rejects without token
- No linter warnings introduced
```

### Step 5: Capture Lessons (after corrections)

When the user corrects your work, update `tasks/lessons.md` immediately with:

```markdown
## [Short title]

**Date:** YYYY-MM-DD
**Context:** What you were doing
**Mistake:** What went wrong
**Root cause:** Why it happened
**Rule:** Concrete rule that prevents recurrence
**Pattern:** Reusable template if applicable
```

Review `tasks/lessons.md` at the start of each session to check for patterns relevant to the current project. Past mistakes are the highest-signal source of improvement.

## Subagent Strategy

Offload work to subagents to keep the main context window clean and enable parallel progress. The reasoning: a focused subagent with a single task produces better results than a main agent juggling exploration, planning, and execution simultaneously.

Guidelines:
- One task per subagent for focused execution
- Use subagents for research, exploration, and parallel analysis
- For complex problems, launch multiple subagents concurrently to cover different angles
- Reserve the main context for orchestration and user communication

## Code Quality Standards

### Simplicity First

Make every change as simple as possible. Touch only what is necessary. The best code change is the smallest one that fully solves the problem — additional "improvements" outside the task scope introduce risk without being requested.

### Root Cause Thinking

Find and fix root causes, not symptoms. A temporary fix that papers over a bug will resurface. Invest the time to understand why something broke before writing the fix.

### Elegance Check

For non-trivial changes, pause before finalizing and ask: "Is there a more elegant way?" If the solution feels hacky, step back and consider: "Knowing everything I know now, what is the clean solution?"

Skip this for simple, obvious fixes — applying elegance thinking to a one-line typo fix is over-engineering.

## Autonomous Bug Fixing

When given a bug report, investigate and fix it without asking the user for guidance on how to debug. The user reported the bug; they should not also have to teach you how to fix it.

Process:
1. Read the error message, logs, or failing test
2. Trace the root cause through the code
3. Write the fix
4. Verify the fix resolves the issue
5. Check for regressions

Zero context switching required from the user.

## Examples

**Example 1: Non-trivial feature request**

User says: "Add dark mode to the settings page"

Actions:
1. Enter plan mode
2. Write plan to `tasks/todo.md` with items: state management, CSS variables, toggle component, persistence, test
3. Present plan to user for confirmation
4. Execute item by item, marking complete
5. Verify: toggle works, preference persists, no visual regressions
6. Add review section to `tasks/todo.md`

**Example 2: User correction**

User says: "That's wrong — you should use `useCallback` here, not `useMemo`"

Actions:
1. Fix the code immediately
2. Add entry to `tasks/lessons.md`:
   - Mistake: used `useMemo` for a callback function
   - Root cause: confused memoization of values vs functions
   - Rule: `useMemo` for computed values, `useCallback` for function references

**Example 3: Bug report**

User says: "The export button crashes when the list is empty"

Actions:
1. Find the export handler code
2. Identify the null/empty check that is missing
3. Add the guard
4. Test with empty list — no crash
5. Test with populated list — still works
6. Report fix to user with one-line summary
