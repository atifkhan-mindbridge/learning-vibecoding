# Step 11: Building Full Applications with AI

**Level:** Advanced
**Time:** ~40 minutes

In [Step 10](10-Code-Review-and-Quality-Assurance.md), you learned how to ensure quality through MRPs and multi-agent verification. This step applies those patterns at scale, showing how to build complete applications with AI while maintaining quality across multiple sessions and context windows.

## Learning Objectives

By the end of this step, you will be able to:

1. Apply phased PRD approach (150-200 instructions per phase)
2. Use initializer agent + coding agent pattern
3. Implement feature tracking with JSON files
4. Handle multi-context-window workflows

---

## The Full Application Challenge

Building complete applications with AI presents unique challenges:

1. **Context limits:** No single context window can hold an entire application
2. **State persistence:** Progress must survive across sessions
3. **Coherence:** Multiple generations must fit together
4. **Completion tracking:** Knowing what's done vs. what remains

**Solution:** Structure work into phases with explicit state management.

---

## Long-Running Agent Patterns

### Long-Running Agent Lifecycle

The following diagram illustrates the complete lifecycle of a long-running agent across multiple sessions:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    LONG-RUNNING AGENT LIFECYCLE                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SESSION 1                   SESSION 2                   SESSION N          │
│  ─────────                   ─────────                   ─────────          │
│                                                                              │
│  ┌─────────────┐            ┌─────────────┐            ┌─────────────┐     │
│  │   START     │            │   START     │            │   START     │     │
│  │ • Init env  │            │ • Read state│            │ • Read state│     │
│  │ • Create    │            │ • Restore   │            │ • Verify    │     │
│  │   tracking  │            │   context   │            │ • Continue  │     │
│  └──────┬──────┘            └──────┬──────┘            └──────┬──────┘     │
│         │                          │                          │             │
│         ▼                          ▼                          ▼             │
│  ┌─────────────┐            ┌─────────────┐            ┌─────────────┐     │
│  │    WORK     │            │    WORK     │            │    WORK     │     │
│  │ • Feature A │            │ • Feature D │            │ • Feature G │     │
│  │ • Feature B │ ──────────►│ • Feature E │ ──────────►│ • Feature H │     │
│  │ • Feature C │  (persist) │ • Feature F │  (persist) │ • Polish    │     │
│  └──────┬──────┘            └──────┬──────┘            └──────┬──────┘     │
│         │                          │                          │             │
│         ▼                          ▼                          ▼             │
│  ┌─────────────┐            ┌─────────────┐            ┌─────────────┐     │
│  │    END      │            │    END      │            │    END      │     │
│  │ • Tests pass│            │ • Tests pass│            │ • All done! │     │
│  │ • Commit    │            │ • Commit    │            │ • Final     │     │
│  │ • Update    │            │ • Update    │            │   commit    │     │
│  │   progress  │            │   progress  │            │             │     │
│  └─────────────┘            └─────────────┘            └─────────────┘     │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  PERSISTENT STATE (survives between sessions):                               │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐       │
│  │ feature_list │ │  progress    │ │    VCRs      │ │ Git commits  │       │
│  │    .json     │ │    .txt      │ │  (decisions) │ │ (code state) │       │
│  └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘       │
│                                                                              │
│  KEY: Context is reconstructed, not remembered                               │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Session Lifecycle

Building full applications requires multiple sessions with consistent state. Each session follows a lifecycle:

**Session Start Protocol:**
1. Verify working directory and environment
2. Read progress tracking files
3. Check current state (tests passing, build working)
4. Identify what to work on next
5. Confirm context is correct before making changes

**Active Work Protocol:**
1. Work on ONE feature at a time
2. Verify each change before moving to next
3. Update tracking files as progress is made
4. Commit atomically after each feature

**Session End Protocol:**
1. Ensure all tests pass
2. Update progress tracking with session summary
3. Document any known issues or blockers
4. Create checkpoint commit
5. List explicit next steps for next session

### State Persistence Strategies

For agents to maintain continuity across sessions, state must be persisted explicitly:

| State Type | Storage Method | Update Frequency |
|------------|---------------|------------------|
| Feature completion | feature_list.json | Per feature |
| Session history | progress.txt | Per session |
| Decisions made | VCRs or CLAUDE.md | When decisions occur |
| Known issues | progress.txt or issues.md | As discovered |
| Code state | Git commits | After each feature |

### Handoff Protocols

When starting a new session, the agent needs to reconstruct context:

**Context Reconstruction Prompt:**
```
This is a CONTINUATION of an ongoing project. Before doing anything:

1. Read progress.txt to understand:
   - What was done in previous sessions
   - Known issues and blockers
   - What was planned for this session

2. Read feature_list.json to see:
   - Which features are complete (passes: true)
   - Which features remain (passes: false)
   - Priority order

3. Run `git log --oneline -10` to see recent commits

4. Run tests to verify the project is in working state

5. Report your understanding of the current state and what you
   plan to work on. DO NOT start implementation until confirmed.
```

### Preventing Context Drift

Over multiple sessions, understanding can drift. Prevent this:

**1. Canonical Sources of Truth**
- feature_list.json is the definitive list of what to build
- progress.txt records what actually happened
- Git history shows the actual changes made

**2. Verification Before Action**
- Always verify assumptions by reading files
- Don't assume memory from previous sessions
- If uncertain, ask before proceeding

**3. Regular Reconciliation**
- Periodically verify that progress.txt matches reality
- Ensure feature_list.json reflects actual completion status
- Check that documented decisions are still being followed

---

## The Phased PRD Approach

### Why Phases Matter

Research from HumanLayer (2025) [^1] found that:
- AI agents work best with **150-200 instructions** at a time
- Larger scopes lead to incomplete implementations
- Phased work allows verification between stages

[^1]: HumanLayer. "Advanced Context Engineering for Coding Agents." (2025). https://github.com/humanlayer/advanced-context-engineering-for-coding-agents

### The 5-Phase Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    Full Application Build                    │
│                                                             │
│  Phase 1: Foundation    ████████████████░░░░░░░░░░░░░░░░   │
│  Phase 2: Core Features ░░░░░░░░░░░░░░░░████████████████░  │
│  Phase 3: Integration   ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░████ │
│  Phase 4: Polish        ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│  Phase 5: Production    ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Phase Details

**Phase 1: Foundation (30-50 requirements)**
- Project scaffolding
- Core data models
- Database schema
- Authentication skeleton
- Basic routing

**Phase 2: Core Features (50-70 requirements)**
- Primary user flows
- Main functionality
- Essential API endpoints
- Core business logic

**Phase 3: Integration (30-50 requirements)**
- External service connections
- Third-party APIs
- Webhooks and events
- Email/notification services

**Phase 4: Polish (30-40 requirements)**
- UX improvements
- Edge case handling
- Error states and messages
- Loading states
- Responsive design fixes

**Phase 5: Production (20-30 requirements)**
- Security hardening
- Performance optimization
- Monitoring and logging
- Documentation
- Deployment configuration

---

## Phased PRD Template

```markdown
# PRD: [Application Name]

## Overview
[One paragraph describing the application and its purpose]

## Technology Stack
- Frontend: [framework, libraries]
- Backend: [framework, database]
- Infrastructure: [hosting, services]

## Success Metrics
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

---

## Phase 1: Foundation
### Timeline: [Estimated sessions/days]

### Project Setup
1. Initialize Next.js 14 project with TypeScript
2. Configure ESLint and Prettier
3. Set up Tailwind CSS
4. Initialize Prisma with PostgreSQL

### Data Models
5. Create User model: id, email, password_hash, created_at, updated_at
6. Create Profile model: id, user_id, display_name, avatar_url
7. Create [Entity] model: [fields]
8. Set up relationships between models
9. Create initial migration

### Authentication
10. Install and configure NextAuth.js
11. Create sign-up page with email/password
12. Create login page
13. Create password reset flow
14. Add session management

### Verification Checkpoint
- [ ] Can create new user account
- [ ] Can log in with credentials
- [ ] Session persists across page loads
- [ ] Can reset password

---

## Phase 2: Core Features
### Timeline: [Estimated sessions/days]

### [Feature 1 Name]
15. Create [feature] list page
16. Implement [feature] creation form
17. Add [feature] detail view
18. Implement [feature] editing
19. Add [feature] deletion with confirmation

[Continue numbering sequentially...]

### Verification Checkpoint
- [ ] [Feature 1] CRUD operations work
- [ ] [Feature 2] complete
- [ ] All tests pass

---

## Phase 3: Integration
[...]

## Phase 4: Polish
[...]

## Phase 5: Production
[...]
```

---

## The Initializer + Coding Agent Pattern

Anthropic's research identified that long-running projects need two distinct agent modes:

### Initializer Agent

Runs once at the start to set up the project:

```
You are the INITIALIZER AGENT for a new project.

Your responsibilities:
1. Create the initial project structure
2. Set up the feature_list.json with all features marked "failing"
3. Create init.sh script for starting the dev server
4. Create progress.txt for tracking work
5. Make an initial git commit

Do NOT implement features. Only set up the scaffolding for the
coding agent to work with.

Project spec:
[Include Phase 1 requirements here]
```

### Coding Agent

Runs in subsequent sessions to make incremental progress:

```
You are the CODING AGENT for an ongoing project.

At the start of each session:
1. Run `pwd` to see your working directory
2. Read progress.txt to understand recent work
3. Read feature_list.json to see what needs to be done
4. Check git log for recent changes
5. Run init.sh to start the dev server
6. Run a basic test to verify the app isn't broken

Then:
1. Pick the highest-priority incomplete feature
2. Implement it fully
3. Test it end-to-end
4. Update feature_list.json to mark it "passing"
5. Update progress.txt with what you did
6. Commit your changes

Work on ONE feature at a time. Leave the codebase in a clean state.
```

---

## Feature Tracking with JSON

### feature_list.json Structure

```json
{
  "features": [
    {
      "id": "auth-001",
      "category": "authentication",
      "description": "User can create account with email and password",
      "priority": 1,
      "steps": [
        "Navigate to /signup",
        "Enter email and password",
        "Click create account",
        "Verify redirected to dashboard",
        "Verify user exists in database"
      ],
      "passes": false
    },
    {
      "id": "auth-002",
      "category": "authentication",
      "description": "User can log in with existing account",
      "priority": 2,
      "steps": [
        "Navigate to /login",
        "Enter valid credentials",
        "Click login",
        "Verify redirected to dashboard",
        "Verify session is active"
      ],
      "passes": false
    }
  ]
}
```

### Why JSON Instead of Markdown?

The agent is less likely to accidentally modify JSON structure compared to Markdown:
- Strict format enforces discipline
- Easy to parse and validate
- Clear pass/fail status
- Harder to "fudge" completion

### Feature List Rules

Include in your agent prompt:

```
IMPORTANT RULES for feature_list.json:
1. ONLY change the "passes" field from false to true
2. NEVER remove features
3. NEVER edit feature descriptions or steps
4. NEVER mark a feature as passing unless you've tested it end-to-end
5. It is UNACCEPTABLE to modify tests or requirements to make them pass
```

---

## Progress Tracking

### progress.txt Structure

```
# Project Progress

## Current Status
Last session: 2026-02-08 14:30 UTC
Features completed: 8/23
Current phase: Phase 2 (Core Features)

## Recent Work

### Session 2026-02-08
- Implemented user profile page
- Added avatar upload functionality
- Fixed bug in authentication middleware
- Committed: "feat: add user profile with avatar upload"

### Session 2026-02-07
- Completed login flow
- Added session persistence
- Committed: "feat: complete authentication system"

## Known Issues
- Profile image takes too long to load (optimization needed in Phase 4)
- Mobile nav menu needs work (deferred to Phase 4)

## Next Steps
1. Implement project creation (auth-003)
2. Add project list view (core-001)
3. Build project detail page (core-002)
```

### Updating Progress

```
At the end of each session:
1. Update progress.txt with:
   - What you accomplished
   - Any issues discovered
   - What should be done next
2. Commit progress.txt along with code changes
```

---

## Multi-Context-Window Workflows

### The Context Window Challenge

Complex applications require more work than fits in one context window. Solutions:

### Approach 1: Session Handoff

Different prompts for first vs. subsequent sessions:

**First session:**
```
This is a NEW project. Run the initializer setup:
1. Create project structure
2. Set up feature_list.json
3. Create progress.txt
4. Make initial commit
```

**Subsequent sessions:**
```
This is an ONGOING project. Continue from where the last session ended:
1. Read progress.txt
2. Check feature_list.json
3. Continue implementing the next incomplete feature
```

### Approach 2: Checkpoint Commits

Create explicit checkpoints that future sessions can resume from:

```
Before ending this session:
1. Ensure all changes are committed
2. Update progress.txt with detailed state
3. Create a git tag: checkpoint-YYYY-MM-DD
4. List exact next steps for the next session
```

### Approach 3: Subagent Delegation

Break large features into subagent tasks:

```
The dashboard feature is large. Delegate to subagents:

SUBAGENT 1: Implement the dashboard data API
SUBAGENT 2: Create the dashboard UI components
SUBAGENT 3: Add real-time update functionality

Orchestrate the subagents, then integrate their work.
```

---

## Common Failure Modes

### Failure Mode Detection Diagram

The following diagram helps identify and classify common failure modes in long-running AI projects:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FAILURE MODE DETECTION FLOWCHART                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                          ┌─────────────────┐                                │
│                          │  PROJECT STUCK  │                                │
│                          └────────┬────────┘                                │
│                                   │                                          │
│                                   ▼                                          │
│                    ┌──────────────────────────────┐                         │
│                    │  Are tests passing?           │                         │
│                    └──────────────┬───────────────┘                         │
│                           ┌───────┴───────┐                                 │
│                           ▼               ▼                                  │
│                        [NO]            [YES]                                 │
│                         │               │                                    │
│                         ▼               ▼                                    │
│           ┌─────────────────────┐  ┌─────────────────────┐                  │
│           │ Does it build?      │  │ Does progress.txt   │                  │
│           └──────────┬──────────┘  │ match reality?      │                  │
│              ┌───────┴───────┐     └──────────┬──────────┘                  │
│              ▼               ▼            ┌───┴───┐                         │
│           [NO]            [YES]           ▼       ▼                          │
│            │               │           [NO]    [YES]                         │
│            ▼               ▼             │       │                           │
│  ┌─────────────────┐ ┌─────────────────┐ │       ▼                          │
│  │ FAILURE MODE:   │ │ FAILURE MODE:   │ │  ┌─────────────────┐             │
│  │ BUILD BROKEN    │ │ TESTS FAILING   │ │  │ Is agent trying │             │
│  │                 │ │                 │ │  │ to do too much? │             │
│  │ Action: Revert  │ │ Action: Fix or  │ │  └────────┬────────┘             │
│  │ to last good    │ │ selective revert│ │     ┌────┴────┐                  │
│  │ commit          │ │                 │ │     ▼         ▼                  │
│  └─────────────────┘ └─────────────────┘ │  [YES]      [NO]                 │
│                                          │    │         │                   │
│                                          │    ▼         ▼                   │
│                                          │  ┌────────┐ ┌────────┐           │
│                         ┌────────────────┘  │ONE-SHOT│ │PREMATURE│          │
│                         ▼                   │FAILURE │ │VICTORY │           │
│            ┌─────────────────┐              │        │ │        │           │
│            │ FAILURE MODE:   │              │Action: │ │Action: │           │
│            │ CONTEXT         │              │Phase & │ │Verify  │           │
│            │ CONFUSION       │              │checkpoint│ │claims  │           │
│            │                 │              └────────┘ └────────┘           │
│            │ Action: Resync  │                                              │
│            │ state files     │                                              │
│            └─────────────────┘                                              │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  RECOVERY PRINCIPLE: Diagnose before fixing, verify before continuing       │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Failure Mode 1: One-Shotting

**Problem:** Agent tries to build entire application in one go

**Symptom:** Incomplete implementation, context exhausted, feature_list.json not being used

**Signal:** Huge commits, skipped phases, no incremental progress tracking

**Solution:** Enforce phased approach with explicit checkpoints

### Failure Mode 2: Premature Victory

**Problem:** Agent declares completion when features don't work

**Symptom:** feature_list.json marked "passing" but manual testing fails

**Signal:** Features marked complete without visible testing, optimistic progress reports

**Solution:** Require end-to-end testing before marking complete; add visual verification

### Failure Mode 3: Context Confusion

**Problem:** Agent loses track of what was done vs. what remains

**Symptom:** Re-implementing completed features, missing incomplete ones

**Signal:** Agent seems to forget progress, repeats work, contradicts previous sessions

**Solution:** Always read progress.txt and feature_list.json first; use /clear if context becomes corrupted

### Failure Mode 4: Broken State

**Problem:** Agent leaves codebase with failing tests or errors

**Symptom:** Next session must fix problems before progressing

**Signal:** Red tests, build failures, uncommitted changes

**Solution:** Always verify basic functionality before ending session

---

## Failure Recovery Protocol

When a project becomes stuck, follow this recovery process:

### Step 1: Assess the Damage

Before attempting fixes, understand the current state:

```
ASSESSMENT PROMPT:
Read progress.txt and feature_list.json. Then:
1. What was the last successfully completed feature?
2. What is currently broken?
3. What error messages are appearing?
4. When did the project last work correctly?

Do NOT try to fix anything yet. Report findings only.
```

### Step 2: Isolate the Problem

Identify when and where things went wrong:

```
ISOLATION PROMPT:
Run the test suite and analyze:
1. Which tests are failing?
2. What was the last passing commit? (use git bisect if needed)
3. What changed between the last green commit and now?

Report findings without making changes.
```

### Step 3: Create Recovery Plan

Decide on the recovery approach:

```
PLANNING PROMPT:
Based on the assessment, create a recovery plan:

Option A: REVERT - Roll back to last known good state
  - Best when: Recent regression, limited good work to preserve
  - Risk: Lose work done after last good commit

Option B: FIX FORWARD - Repair the current state
  - Best when: Problem is isolated and understood
  - Risk: May introduce new issues

Option C: SELECTIVE REVERT - Revert specific changes
  - Best when: Problem traced to specific commits
  - Risk: May leave inconsistent state

Recommend an approach with justification. Do NOT execute yet.
```

### Step 4: Execute Recovery

With plan approved, execute carefully:

```
EXECUTION PROMPT:
Execute the recovery plan step by step.
After EACH step:
1. Run tests to verify improvement
2. Report current state
3. Stop and report if things get worse
```

### Step 5: Document and Prevent

After recovery, prevent recurrence:

```
DOCUMENTATION PROMPT:
Update project documentation:
1. Add lessons learned to progress.txt
2. Add any new constraints to CLAUDE.md
3. Create a commit: "fix: recover from [issue description]"
4. Consider if failure mode should be added to this prompt
```

### Common Recovery Scenarios

**Scenario: Tests Failing, Don't Know Why**
1. Use `git bisect` to find the breaking commit
2. Review that commit carefully
3. Either revert or fix the specific issue

**Scenario: Feature Works Locally, Fails in Tests**
1. Check for environment differences
2. Look for timing or race conditions
3. Verify test expectations match reality

**Scenario: Multiple Intertwined Issues**
1. Revert to last known good state
2. Re-apply changes one at a time
3. Test after each change to catch the problem

**Scenario: Context Seems Corrupted**
1. Use /clear to reset agent context
2. Start fresh with progress files as source of truth
3. Verify understanding before resuming work

### Complete Failure Recovery Script

Use this script when projects become stuck:

```python
#!/usr/bin/env python3
"""
recover_project.py - Automated Project Recovery Tool

This script helps recover from common failure states in AI-assisted projects.
It diagnoses issues, suggests recovery approaches, and can execute automated
recovery when safe to do so.

Usage:
    python recover_project.py diagnose     # Analyze current state
    python recover_project.py plan         # Create recovery plan
    python recover_project.py execute      # Execute recovery (with confirmation)
    python recover_project.py --auto       # Attempt automatic recovery
"""

import argparse
import json
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime
from pathlib import Path
from typing import Optional
from enum import Enum


class FailureType(Enum):
    TESTS_FAILING = "tests_failing"
    BUILD_BROKEN = "build_broken"
    CONTEXT_DRIFT = "context_drift"
    STUCK_LOOP = "stuck_loop"
    STATE_DESYNC = "state_desync"
    UNCOMMITTED_MESS = "uncommitted_mess"
    UNKNOWN = "unknown"


class RecoveryAction(Enum):
    REVERT = "revert"
    FIX_FORWARD = "fix_forward"
    SELECTIVE_REVERT = "selective_revert"
    RESYNC_STATE = "resync_state"
    CLEAN_RESTART = "clean_restart"


@dataclass
class DiagnosisResult:
    timestamp: datetime
    failure_types: list[FailureType]
    last_good_commit: Optional[str]
    broken_commit: Optional[str]
    failing_tests: list[str]
    uncommitted_changes: list[str]
    state_issues: list[str]
    severity: str  # "low", "medium", "high", "critical"
    details: dict = field(default_factory=dict)


@dataclass
class RecoveryPlan:
    diagnosis: DiagnosisResult
    recommended_action: RecoveryAction
    steps: list[str]
    risks: list[str]
    estimated_effort: str
    requires_human: bool


class ProjectRecovery:
    """Diagnose and recover from common AI project failures."""

    def __init__(self, project_dir: Path = Path(".")):
        self.project_dir = project_dir
        self.feature_list_path = project_dir / "feature_list.json"
        self.progress_path = project_dir / "progress.txt"

    def run_command(self, cmd: list[str], check: bool = False) -> tuple[int, str, str]:
        """Run a command and return (exit_code, stdout, stderr)."""
        try:
            result = subprocess.run(
                cmd,
                cwd=self.project_dir,
                capture_output=True,
                text=True,
                timeout=120
            )
            return result.returncode, result.stdout, result.stderr
        except subprocess.TimeoutExpired:
            return 1, "", "Command timed out"
        except Exception as e:
            return 1, "", str(e)

    def get_git_log(self, count: int = 20) -> list[dict]:
        """Get recent git commits."""
        code, stdout, _ = self.run_command([
            "git", "log", f"-{count}", "--format=%H|%s|%ci"
        ])
        if code != 0:
            return []

        commits = []
        for line in stdout.strip().split('\n'):
            if '|' in line:
                parts = line.split('|')
                commits.append({
                    "hash": parts[0][:8],
                    "full_hash": parts[0],
                    "message": parts[1],
                    "date": parts[2] if len(parts) > 2 else ""
                })
        return commits

    def find_last_good_commit(self) -> Optional[str]:
        """Use git bisect logic to find last commit where tests passed."""
        commits = self.get_git_log(20)

        for commit in commits:
            # Check out commit
            code, _, _ = self.run_command(["git", "stash"])
            code, _, _ = self.run_command(["git", "checkout", commit["full_hash"]])

            # Run tests
            test_code, _, _ = self.run_command(["npm", "test"])

            # Return to original branch
            self.run_command(["git", "checkout", "-"])
            self.run_command(["git", "stash", "pop"])

            if test_code == 0:
                return commit["hash"]

        return None

    def check_tests(self) -> tuple[bool, list[str]]:
        """Run tests and return (passed, failing_test_names)."""
        code, stdout, stderr = self.run_command(["npm", "test", "--", "--json"])

        failing = []
        if code != 0:
            # Parse test output for failing tests
            for line in (stdout + stderr).split('\n'):
                if 'FAIL' in line or 'failed' in line.lower():
                    failing.append(line.strip()[:100])

        return code == 0, failing[:10]  # Limit to 10 failures

    def check_build(self) -> bool:
        """Check if project builds."""
        code, _, _ = self.run_command(["npm", "run", "build"])
        return code == 0

    def check_uncommitted_changes(self) -> list[str]:
        """Get list of uncommitted changes."""
        code, stdout, _ = self.run_command(["git", "status", "--porcelain"])
        if code == 0 and stdout.strip():
            return [line.strip() for line in stdout.strip().split('\n')]
        return []

    def check_state_sync(self) -> list[str]:
        """Check if feature_list.json, progress.txt, and git are in sync."""
        issues = []

        # Load feature list
        if not self.feature_list_path.exists():
            issues.append("feature_list.json missing")
            return issues

        try:
            with open(self.feature_list_path) as f:
                data = json.load(f)

            # Check for features marked passing that might not actually work
            passing_features = [f for f in data.get("features", []) if f.get("passes")]

            # Run tests to see if they actually pass
            tests_pass, _ = self.check_tests()
            if not tests_pass and passing_features:
                issues.append(f"{len(passing_features)} features marked passing but tests fail")

        except json.JSONDecodeError:
            issues.append("feature_list.json is invalid JSON")

        # Check progress.txt
        if not self.progress_path.exists():
            issues.append("progress.txt missing")
        else:
            content = self.progress_path.read_text()
            today = datetime.now().strftime("%Y-%m-%d")
            yesterday = (datetime.now().replace(day=datetime.now().day-1)).strftime("%Y-%m-%d")
            if today not in content and yesterday not in content:
                issues.append("progress.txt may be stale (no recent dates)")

        return issues

    def diagnose(self) -> DiagnosisResult:
        """Comprehensive diagnosis of project state."""
        print("Diagnosing project state...")

        failure_types = []
        details = {}

        # Check tests
        print("  Checking tests...")
        tests_pass, failing_tests = self.check_tests()
        if not tests_pass:
            failure_types.append(FailureType.TESTS_FAILING)
            details["failing_tests"] = failing_tests

        # Check build
        print("  Checking build...")
        if not self.check_build():
            failure_types.append(FailureType.BUILD_BROKEN)

        # Check uncommitted changes
        print("  Checking git status...")
        uncommitted = self.check_uncommitted_changes()
        if len(uncommitted) > 10:
            failure_types.append(FailureType.UNCOMMITTED_MESS)

        # Check state synchronization
        print("  Checking state sync...")
        state_issues = self.check_state_sync()
        if state_issues:
            failure_types.append(FailureType.STATE_DESYNC)

        # Find last good commit if tests are failing
        last_good = None
        broken_commit = None
        if FailureType.TESTS_FAILING in failure_types:
            print("  Finding last good commit (this may take a while)...")
            last_good = self.find_last_good_commit()
            if last_good:
                commits = self.get_git_log(20)
                for i, commit in enumerate(commits):
                    if commit["hash"] == last_good and i > 0:
                        broken_commit = commits[i-1]["hash"]
                        break

        # Determine severity
        if FailureType.BUILD_BROKEN in failure_types:
            severity = "critical"
        elif FailureType.TESTS_FAILING in failure_types:
            severity = "high"
        elif FailureType.STATE_DESYNC in failure_types:
            severity = "medium"
        else:
            severity = "low"

        if not failure_types:
            failure_types.append(FailureType.UNKNOWN)

        return DiagnosisResult(
            timestamp=datetime.now(),
            failure_types=failure_types,
            last_good_commit=last_good,
            broken_commit=broken_commit,
            failing_tests=failing_tests if not tests_pass else [],
            uncommitted_changes=uncommitted,
            state_issues=state_issues,
            severity=severity,
            details=details
        )

    def create_plan(self, diagnosis: DiagnosisResult) -> RecoveryPlan:
        """Create a recovery plan based on diagnosis."""

        # Determine best action based on failure types
        if FailureType.BUILD_BROKEN in diagnosis.failure_types:
            if diagnosis.last_good_commit:
                action = RecoveryAction.REVERT
                steps = [
                    f"1. Stash any uncommitted changes: git stash",
                    f"2. Revert to last good commit: git revert --no-commit {diagnosis.broken_commit}..HEAD",
                    f"3. Review reverted changes",
                    f"4. Commit revert: git commit -m 'revert: recover from build failure'",
                    f"5. Pop stash if needed: git stash pop",
                    f"6. Re-apply changes incrementally with testing",
                ]
                risks = ["May lose some work that needs to be re-implemented"]
                effort = "30-60 minutes"
                requires_human = True
            else:
                action = RecoveryAction.FIX_FORWARD
                steps = [
                    "1. Read build error output carefully",
                    "2. Fix the specific build errors",
                    "3. Run build after each fix to verify",
                    "4. Once building, run tests",
                ]
                risks = ["May introduce new issues while fixing"]
                effort = "1-2 hours"
                requires_human = True

        elif FailureType.TESTS_FAILING in diagnosis.failure_types:
            if diagnosis.last_good_commit and len(diagnosis.failing_tests) > 3:
                action = RecoveryAction.SELECTIVE_REVERT
                steps = [
                    f"1. Review commits since {diagnosis.last_good_commit}",
                    f"2. Identify which commit(s) broke tests",
                    f"3. Revert specific problematic commits",
                    f"4. Verify tests pass after each revert",
                    f"5. Re-implement reverted features correctly",
                ]
                risks = ["Need to carefully identify problematic commits"]
                effort = "1-2 hours"
                requires_human = True
            else:
                action = RecoveryAction.FIX_FORWARD
                steps = [
                    "1. Read failing test output",
                    "2. Identify root cause of each failure",
                    "3. Fix tests one at a time",
                    "4. Run full suite after each fix",
                ]
                risks = ["Fixes may mask underlying issues"]
                effort = "30-60 minutes"
                requires_human = False

        elif FailureType.STATE_DESYNC in diagnosis.failure_types:
            action = RecoveryAction.RESYNC_STATE
            steps = [
                "1. Run all tests to get true pass/fail status",
                "2. Update feature_list.json to match reality",
                "3. Update progress.txt with current state",
                "4. Commit state files: git add feature_list.json progress.txt",
                "5. Commit: git commit -m 'chore: resync project state'",
            ]
            risks = ["May reveal more incomplete work than expected"]
            effort = "15-30 minutes"
            requires_human = False

        elif FailureType.UNCOMMITTED_MESS in diagnosis.failure_types:
            action = RecoveryAction.CLEAN_RESTART
            steps = [
                "1. Review uncommitted changes: git diff",
                "2. Decide what to keep vs discard",
                "3. Stage changes to keep: git add <files>",
                "4. Commit in logical chunks",
                "5. Discard unwanted changes: git checkout -- <files>",
                "6. Update progress.txt with what was done",
            ]
            risks = ["May accidentally discard needed work"]
            effort = "30-60 minutes"
            requires_human = True

        else:
            action = RecoveryAction.FIX_FORWARD
            steps = [
                "1. Manual investigation required",
                "2. Review recent git history",
                "3. Check for obvious issues",
                "4. Consult with team if needed",
            ]
            risks = ["Unknown state may have hidden issues"]
            effort = "Unknown"
            requires_human = True

        return RecoveryPlan(
            diagnosis=diagnosis,
            recommended_action=action,
            steps=steps,
            risks=risks,
            estimated_effort=effort,
            requires_human=requires_human
        )

    def execute_auto_recovery(self, plan: RecoveryPlan) -> bool:
        """Execute automatic recovery steps where safe."""
        if plan.requires_human:
            print("This recovery plan requires human intervention.")
            return False

        print(f"\nExecuting automatic recovery: {plan.recommended_action.value}")

        if plan.recommended_action == RecoveryAction.RESYNC_STATE:
            # Run tests and update feature_list.json
            print("  Running tests...")
            tests_pass, failing = self.check_tests()

            print("  Updating feature_list.json...")
            if self.feature_list_path.exists():
                with open(self.feature_list_path) as f:
                    data = json.load(f)

                # Mark features as not passing if tests fail
                if not tests_pass:
                    for feature in data.get("features", []):
                        # Conservative: if tests fail, mark all as needs verification
                        if feature.get("passes"):
                            feature["passes"] = False
                            feature["needs_verification"] = True

                with open(self.feature_list_path, 'w') as f:
                    json.dump(data, f, indent=2)

            # Update progress.txt
            print("  Updating progress.txt...")
            progress_content = f"""# Project Progress

## Recovery Session - {datetime.now().strftime('%Y-%m-%d %H:%M')}

### Automated Recovery Performed
- Ran tests: {'PASS' if tests_pass else 'FAIL'}
- Updated feature_list.json to reflect actual status
- State resynchronized

### Current Status
- Tests: {'Passing' if tests_pass else f'Failing ({len(failing)} failures)'}
- Build: {'Working' if self.check_build() else 'Broken'}

### Next Steps
1. Review feature_list.json for features marked needs_verification
2. Re-test features manually if needed
3. Continue development from verified state

"""
            if self.progress_path.exists():
                existing = self.progress_path.read_text()
                progress_content += f"\n---\n\n## Previous Progress\n\n{existing}"

            self.progress_path.write_text(progress_content)

            print("  Recovery complete!")
            return True

        return False


def format_diagnosis(diagnosis: DiagnosisResult) -> str:
    """Format diagnosis as readable output."""
    lines = [
        "=" * 60,
        "PROJECT DIAGNOSIS REPORT",
        "=" * 60,
        f"Timestamp: {diagnosis.timestamp.isoformat()}",
        f"Severity: {diagnosis.severity.upper()}",
        "",
        "Failure Types Detected:",
    ]

    for ft in diagnosis.failure_types:
        lines.append(f"  - {ft.value}")

    if diagnosis.last_good_commit:
        lines.append(f"\nLast Good Commit: {diagnosis.last_good_commit}")
    if diagnosis.broken_commit:
        lines.append(f"Breaking Commit: {diagnosis.broken_commit}")

    if diagnosis.failing_tests:
        lines.append("\nFailing Tests:")
        for test in diagnosis.failing_tests[:5]:
            lines.append(f"  - {test}")
        if len(diagnosis.failing_tests) > 5:
            lines.append(f"  ... and {len(diagnosis.failing_tests) - 5} more")

    if diagnosis.state_issues:
        lines.append("\nState Issues:")
        for issue in diagnosis.state_issues:
            lines.append(f"  - {issue}")

    if diagnosis.uncommitted_changes:
        lines.append(f"\nUncommitted Changes: {len(diagnosis.uncommitted_changes)} files")

    lines.append("=" * 60)
    return '\n'.join(lines)


def format_plan(plan: RecoveryPlan) -> str:
    """Format recovery plan as readable output."""
    lines = [
        "=" * 60,
        "RECOVERY PLAN",
        "=" * 60,
        f"Recommended Action: {plan.recommended_action.value}",
        f"Estimated Effort: {plan.estimated_effort}",
        f"Requires Human: {'Yes' if plan.requires_human else 'No (can auto-execute)'}",
        "",
        "Steps:",
    ]

    for step in plan.steps:
        lines.append(f"  {step}")

    lines.append("\nRisks:")
    for risk in plan.risks:
        lines.append(f"  - {risk}")

    lines.append("=" * 60)
    return '\n'.join(lines)


def main():
    parser = argparse.ArgumentParser(description="Project Recovery Tool")
    parser.add_argument("action", nargs="?", default="diagnose",
                       choices=["diagnose", "plan", "execute"],
                       help="Action to perform")
    parser.add_argument("--auto", action="store_true",
                       help="Attempt automatic recovery")
    parser.add_argument("--project-dir", "-d", default=".",
                       help="Project directory")

    args = parser.parse_args()

    recovery = ProjectRecovery(Path(args.project_dir))

    if args.action == "diagnose":
        diagnosis = recovery.diagnose()
        print(format_diagnosis(diagnosis))

    elif args.action == "plan":
        diagnosis = recovery.diagnose()
        print(format_diagnosis(diagnosis))
        print()
        plan = recovery.create_plan(diagnosis)
        print(format_plan(plan))

    elif args.action == "execute" or args.auto:
        diagnosis = recovery.diagnose()
        print(format_diagnosis(diagnosis))
        print()
        plan = recovery.create_plan(diagnosis)
        print(format_plan(plan))
        print()

        if args.auto and not plan.requires_human:
            success = recovery.execute_auto_recovery(plan)
            sys.exit(0 if success else 1)
        elif plan.requires_human:
            print("Manual recovery required. Follow the steps above.")
            sys.exit(1)
        else:
            confirm = input("Execute automatic recovery? [y/N]: ")
            if confirm.lower() == 'y':
                success = recovery.execute_auto_recovery(plan)
                sys.exit(0 if success else 1)


if __name__ == "__main__":
    main()
```

**Usage:**

```bash
# Diagnose current project state
python recover_project.py diagnose

# Get diagnosis and recovery plan
python recover_project.py plan

# Execute recovery (with confirmation)
python recover_project.py execute

# Automatic recovery for CI/CD
python recover_project.py --auto
```

---

## State Synchronization

### State Synchronization Diagram

The following diagram shows how the three sources of truth must stay synchronized:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    STATE SYNCHRONIZATION ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│         ┌──────────────────────────────────────────────────────┐            │
│         │                 SYNCHRONIZATION HUB                   │            │
│         │                                                       │            │
│         │   All three sources must agree on project state:      │            │
│         │   • What features exist                               │            │
│         │   • Which are complete                                │            │
│         │   • What was done when                                │            │
│         └──────────────────────────────────────────────────────┘            │
│                            │                                                 │
│            ┌───────────────┼───────────────┐                                │
│            ▼               ▼               ▼                                 │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐               │
│  │ feature_list.   │ │  progress.txt   │ │  Git Commits    │               │
│  │     json        │ │                 │ │                 │               │
│  │                 │ │                 │ │                 │               │
│  │ PURPOSE:        │ │ PURPOSE:        │ │ PURPOSE:        │               │
│  │ What to build   │ │ What happened   │ │ Actual code     │               │
│  │                 │ │                 │ │                 │               │
│  │ CONTAINS:       │ │ CONTAINS:       │ │ CONTAINS:       │               │
│  │ • Feature list  │ │ • Session logs  │ │ • Code changes  │               │
│  │ • Pass/fail     │ │ • Issues found  │ │ • Change history│               │
│  │ • Priority      │ │ • Next steps    │ │ • Authorship    │               │
│  │                 │ │                 │ │                 │               │
│  │ UPDATE WHEN:    │ │ UPDATE WHEN:    │ │ UPDATE WHEN:    │               │
│  │ Feature passes  │ │ End of session  │ │ After each      │               │
│  │ all tests       │ │                 │ │ feature         │               │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘               │
│            │               │               │                                │
│            └───────────────┼───────────────┘                                │
│                            ▼                                                 │
│                   ┌─────────────────┐                                       │
│                   │   DESYNC?       │                                       │
│                   └────────┬────────┘                                       │
│                     ┌──────┴──────┐                                         │
│                     ▼             ▼                                          │
│                  [YES]          [NO]                                         │
│                    │             │                                           │
│                    ▼             ▼                                           │
│         ┌─────────────────┐  ┌─────────────────┐                           │
│         │ RECONCILIATION  │  │   CONTINUE      │                           │
│         │ REQUIRED        │  │   WORK          │                           │
│         │                 │  │                 │                           │
│         │ 1. Trust Git    │  │   State is      │                           │
│         │    as reality   │  │   consistent    │                           │
│         │ 2. Update       │  │                 │                           │
│         │    feature_list │  │                 │                           │
│         │ 3. Update       │  │                 │                           │
│         │    progress.txt │  │                 │                           │
│         └─────────────────┘  └─────────────────┘                           │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  GOLDEN RULE: Git is the ultimate source of truth for code state            │
│  If feature_list.json claims "passing" but tests fail, trust the tests      │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Three Sources of Truth

For long-running projects, three artifacts must stay synchronized:

**1. feature_list.json**
- Definitive list of what needs to be built
- Tracks completion status
- Should only be modified to mark features as passing
- Never remove or rename features mid-project

**2. progress.txt**
- Human-readable history of what happened
- Session-by-session log of work done
- Documents known issues and next steps
- Updated at end of each session

**3. Git Commits**
- Actual code changes
- Provides rollback points
- Should match feature completion
- Atomic commits per feature

### Synchronization Rules

**Rule 1: feature_list.json leads**
- A feature is not "done" until marked passing in feature_list.json
- The passes field is only set to true after end-to-end verification

**Rule 2: progress.txt reflects reality**
- What progress.txt says happened should match git history
- Include honest assessment of issues and blockers
- Don't overstate progress

**Rule 3: Commits match features**
- Each feature completion should have a corresponding commit
- Commit messages should reference feature IDs
- Never commit broken code to main branch

### Detecting Synchronization Problems

Watch for these warning signs:
- feature_list.json says complete but tests fail
- progress.txt claims work that's not in git history
- Git shows changes not mentioned in progress.txt
- Features marked incomplete that clearly exist in code

When detected, stop and reconcile before continuing.

---

## The init.sh Pattern

Create a script that quickly gets the project running:

```bash
#!/bin/bash
# init.sh - Start development environment

echo "Starting development server..."

# Kill any existing processes on our ports
lsof -ti:3000 | xargs kill -9 2>/dev/null
lsof -ti:5432 | xargs kill -9 2>/dev/null

# Start database (if local)
docker-compose up -d db

# Wait for database
sleep 2

# Run migrations
npx prisma migrate dev

# Start development server
npm run dev &

# Wait for server to be ready
echo "Waiting for server..."
while ! curl -s http://localhost:3000 > /dev/null; do
    sleep 1
done

echo "Server ready at http://localhost:3000"
```

---

## Visual Verification with Puppeteer

For web applications, add visual verification to catch issues that tests might miss.

### Why Visual Verification Matters

Tests verify that code executes correctly, but they may miss:
- CSS/layout issues
- Responsive design problems
- Visual regressions
- User experience issues
- Elements that render but are unusable

### Integration Approaches

**Approach 1: MCP-Based (Real-time)**

Use Puppeteer MCP during development:
```
After implementing a feature:
1. Start the dev server
2. Use Puppeteer MCP to navigate to the feature
3. Take a screenshot
4. Verify the feature works as expected
5. Only mark as "passing" if visual verification succeeds
```

**Approach 2: Automated Visual Tests (CI/CD)**

Add visual regression tests to your pipeline:
- Capture baseline screenshots for each feature
- Compare new screenshots against baselines
- Fail if differences exceed threshold
- Human review for intentional changes

**Approach 3: Hybrid (Recommended)**

Combine both approaches:
- Real-time verification during development
- Automated visual tests catch regressions in CI
- Baseline updates reviewed by humans

### Example Visual Test Protocol

```
Test the user profile feature:
1. Navigate to /login
2. Log in with test credentials
3. Navigate to /profile
4. Verify the profile page loads (check for key elements)
5. Upload a test avatar
6. Verify avatar displays correctly
7. Check responsive behavior at different viewport sizes
8. Take screenshots at each state
9. Mark feature as passing only if all visual checks succeed
```

### Screenshot Comparison Strategy

**What to Compare:**
- Key page states (loaded, loading, error)
- Interactive element states (hover, active, disabled)
- Different viewport sizes (mobile, tablet, desktop)
- Different data states (empty, populated, error)

**Handling Intentional Changes:**
- Keep baselines in version control
- Update baselines as part of feature PRs
- Review baseline changes in code review

### Visual Verification Checklist

Before marking a UI feature as complete:

- [ ] Page renders without errors
- [ ] Key elements are visible and accessible
- [ ] Interactive elements respond correctly
- [ ] Responsive behavior is correct at all breakpoints
- [ ] Loading and error states display properly
- [ ] Screenshots captured for documentation/baselines

### Complete Visual Verification Script with Puppeteer

Use this script for automated visual verification of UI features:

```javascript
/**
 * visual-verify.js - Automated Visual Verification for AI-Generated UIs
 *
 * This script uses Puppeteer to verify that UI features work as expected,
 * capturing screenshots as evidence. Integrates with feature_list.json to
 * mark features as passing only after visual confirmation.
 *
 * Usage:
 *   node visual-verify.js                    # Run all visual tests
 *   node visual-verify.js --feature auth-002 # Run specific feature
 *   node visual-verify.js --update-baselines # Update baseline screenshots
 */

const puppeteer = require('puppeteer');
const fs = require('fs').promises;
const path = require('path');
const { PNG } = require('pngjs');
const pixelmatch = require('pixelmatch');

// Configuration
const CONFIG = {
  baseUrl: process.env.BASE_URL || 'http://localhost:3000',
  screenshotDir: './screenshots',
  baselineDir: './screenshots/baselines',
  diffDir: './screenshots/diffs',
  featureListPath: './feature_list.json',
  viewports: [
    { name: 'mobile', width: 375, height: 667 },
    { name: 'tablet', width: 768, height: 1024 },
    { name: 'desktop', width: 1280, height: 800 },
  ],
  diffThreshold: 0.1, // 10% pixel difference allowed
  timeout: 30000,
};

/**
 * Feature verification step definition
 */
class VerificationStep {
  constructor(description, action, expectation) {
    this.description = description;
    this.action = action;       // async function(page) => void
    this.expectation = expectation; // async function(page) => boolean
  }
}

/**
 * Feature test definition
 */
class FeatureTest {
  constructor(id, name, steps) {
    this.id = id;
    this.name = name;
    this.steps = steps;
  }
}

/**
 * Visual verification engine
 */
class VisualVerifier {
  constructor(config = CONFIG) {
    this.config = config;
    this.browser = null;
    this.results = [];
  }

  async initialize() {
    // Ensure directories exist
    await fs.mkdir(this.config.screenshotDir, { recursive: true });
    await fs.mkdir(this.config.baselineDir, { recursive: true });
    await fs.mkdir(this.config.diffDir, { recursive: true });

    // Launch browser
    this.browser = await puppeteer.launch({
      headless: 'new',
      args: ['--no-sandbox', '--disable-setuid-sandbox'],
    });
  }

  async cleanup() {
    if (this.browser) {
      await this.browser.close();
    }
  }

  async createPage(viewport = CONFIG.viewports[2]) {
    const page = await this.browser.newPage();
    await page.setViewport(viewport);
    page.setDefaultTimeout(this.config.timeout);
    return page;
  }

  async captureScreenshot(page, name) {
    const screenshotPath = path.join(this.config.screenshotDir, `${name}.png`);
    await page.screenshot({ path: screenshotPath, fullPage: true });
    return screenshotPath;
  }

  async compareWithBaseline(screenshotPath, baselineName) {
    const baselinePath = path.join(this.config.baselineDir, `${baselineName}.png`);

    try {
      const [screenshot, baseline] = await Promise.all([
        fs.readFile(screenshotPath),
        fs.readFile(baselinePath),
      ]);

      const img1 = PNG.sync.read(screenshot);
      const img2 = PNG.sync.read(baseline);

      if (img1.width !== img2.width || img1.height !== img2.height) {
        return { match: false, reason: 'Size mismatch' };
      }

      const diff = new PNG({ width: img1.width, height: img1.height });
      const numDiffPixels = pixelmatch(
        img1.data,
        img2.data,
        diff.data,
        img1.width,
        img1.height,
        { threshold: 0.1 }
      );

      const diffPercent = numDiffPixels / (img1.width * img1.height);

      if (diffPercent > this.config.diffThreshold) {
        // Save diff image
        const diffPath = path.join(this.config.diffDir, `${baselineName}-diff.png`);
        await fs.writeFile(diffPath, PNG.sync.write(diff));

        return {
          match: false,
          reason: `${(diffPercent * 100).toFixed(2)}% pixels different`,
          diffPath,
        };
      }

      return { match: true, diffPercent };
    } catch (error) {
      if (error.code === 'ENOENT') {
        return { match: false, reason: 'No baseline exists' };
      }
      throw error;
    }
  }

  async verifyFeature(feature, updateBaselines = false) {
    console.log(`\nVerifying feature: ${feature.id} - ${feature.name}`);
    console.log('='.repeat(50));

    const result = {
      featureId: feature.id,
      featureName: feature.name,
      timestamp: new Date().toISOString(),
      steps: [],
      passed: true,
      screenshots: [],
    };

    const page = await this.createPage();

    try {
      for (let i = 0; i < feature.steps.length; i++) {
        const step = feature.steps[i];
        console.log(`  Step ${i + 1}: ${step.description}`);

        const stepResult = {
          index: i + 1,
          description: step.description,
          passed: false,
          screenshot: null,
          error: null,
        };

        try {
          // Execute action
          await step.action(page);

          // Small delay for rendering
          await page.waitForTimeout(500);

          // Capture screenshot
          const screenshotName = `${feature.id}-step-${i + 1}`;
          const screenshotPath = await this.captureScreenshot(page, screenshotName);
          stepResult.screenshot = screenshotPath;
          result.screenshots.push(screenshotPath);

          // Update baseline if requested
          if (updateBaselines) {
            const baselinePath = path.join(this.config.baselineDir, `${screenshotName}.png`);
            await fs.copyFile(screenshotPath, baselinePath);
            console.log(`    Updated baseline: ${baselinePath}`);
          }

          // Compare with baseline
          const comparison = await this.compareWithBaseline(screenshotPath, screenshotName);
          if (!comparison.match && !updateBaselines) {
            console.log(`    Visual diff: ${comparison.reason}`);
          }

          // Verify expectation
          if (step.expectation) {
            const expectationMet = await step.expectation(page);
            if (!expectationMet) {
              throw new Error('Expectation not met');
            }
          }

          stepResult.passed = true;
          console.log(`    [PASS]`);

        } catch (error) {
          stepResult.passed = false;
          stepResult.error = error.message;
          result.passed = false;
          console.log(`    [FAIL] ${error.message}`);

          // Capture failure screenshot
          try {
            const failScreenshot = `${feature.id}-step-${i + 1}-FAILED`;
            await this.captureScreenshot(page, failScreenshot);
          } catch (e) {
            // Ignore screenshot errors
          }
        }

        result.steps.push(stepResult);
      }

    } finally {
      await page.close();
    }

    this.results.push(result);
    return result;
  }

  async updateFeatureList(featureId, passed) {
    try {
      const content = await fs.readFile(this.config.featureListPath, 'utf-8');
      const data = JSON.parse(content);

      const feature = data.features.find(f => f.id === featureId);
      if (feature) {
        feature.passes = passed;
        feature.lastVerified = new Date().toISOString();
        await fs.writeFile(
          this.config.featureListPath,
          JSON.stringify(data, null, 2)
        );
        console.log(`  Updated feature_list.json: ${featureId} = ${passed}`);
      }
    } catch (error) {
      console.error(`  Failed to update feature_list.json: ${error.message}`);
    }
  }

  generateReport() {
    const passed = this.results.filter(r => r.passed).length;
    const total = this.results.length;

    let report = `# Visual Verification Report\n\n`;
    report += `**Generated:** ${new Date().toISOString()}\n`;
    report += `**Results:** ${passed}/${total} features passed\n\n`;

    for (const result of this.results) {
      const status = result.passed ? '[PASS]' : '[FAIL]';
      report += `## ${status} ${result.featureId}: ${result.featureName}\n\n`;

      for (const step of result.steps) {
        const stepStatus = step.passed ? '+' : '-';
        report += `${stepStatus} Step ${step.index}: ${step.description}\n`;
        if (step.error) {
          report += `  Error: ${step.error}\n`;
        }
        if (step.screenshot) {
          report += `  Screenshot: ${step.screenshot}\n`;
        }
      }
      report += '\n';
    }

    return report;
  }
}

// ============================================================
// Example Feature Tests
// ============================================================

const featureTests = [
  new FeatureTest('auth-001', 'User Registration', [
    new VerificationStep(
      'Navigate to registration page',
      async (page) => {
        await page.goto(`${CONFIG.baseUrl}/register`);
        await page.waitForSelector('form');
      },
      async (page) => {
        return await page.$('input[name="email"]') !== null;
      }
    ),
    new VerificationStep(
      'Fill registration form',
      async (page) => {
        await page.type('input[name="email"]', 'test@example.com');
        await page.type('input[name="password"]', 'SecurePassword123!');
        await page.type('input[name="confirmPassword"]', 'SecurePassword123!');
      },
      async (page) => {
        const email = await page.$eval('input[name="email"]', el => el.value);
        return email === 'test@example.com';
      }
    ),
    new VerificationStep(
      'Submit form and verify success',
      async (page) => {
        await page.click('button[type="submit"]');
        await page.waitForNavigation({ waitUntil: 'networkidle0' });
      },
      async (page) => {
        const url = page.url();
        return url.includes('/dashboard') || url.includes('/welcome');
      }
    ),
  ]),

  new FeatureTest('auth-002', 'User Login', [
    new VerificationStep(
      'Navigate to login page',
      async (page) => {
        await page.goto(`${CONFIG.baseUrl}/login`);
        await page.waitForSelector('form');
      },
      async (page) => {
        return await page.$('input[name="email"]') !== null;
      }
    ),
    new VerificationStep(
      'Enter credentials',
      async (page) => {
        await page.type('input[name="email"]', 'test@example.com');
        await page.type('input[name="password"]', 'SecurePassword123!');
      },
      null
    ),
    new VerificationStep(
      'Submit and verify dashboard loads',
      async (page) => {
        await page.click('button[type="submit"]');
        await page.waitForSelector('#dashboard', { timeout: 10000 });
      },
      async (page) => {
        const dashboard = await page.$('#dashboard');
        return dashboard !== null;
      }
    ),
    new VerificationStep(
      'Verify user menu appears',
      async (page) => {
        await page.waitForSelector('[data-testid="user-menu"]');
      },
      async (page) => {
        const userMenu = await page.$('[data-testid="user-menu"]');
        return userMenu !== null;
      }
    ),
  ]),

  new FeatureTest('ui-001', 'Responsive Navigation', [
    new VerificationStep(
      'Desktop: Navigation visible',
      async (page) => {
        await page.setViewport({ width: 1280, height: 800 });
        await page.goto(`${CONFIG.baseUrl}/`);
        await page.waitForSelector('nav');
      },
      async (page) => {
        const nav = await page.$('nav');
        const box = await nav.boundingBox();
        return box && box.width > 0;
      }
    ),
    new VerificationStep(
      'Mobile: Hamburger menu visible',
      async (page) => {
        await page.setViewport({ width: 375, height: 667 });
        await page.reload();
        await page.waitForSelector('[data-testid="mobile-menu-button"]');
      },
      async (page) => {
        const menuButton = await page.$('[data-testid="mobile-menu-button"]');
        return menuButton !== null;
      }
    ),
    new VerificationStep(
      'Mobile: Menu opens on click',
      async (page) => {
        await page.click('[data-testid="mobile-menu-button"]');
        await page.waitForSelector('[data-testid="mobile-menu"]', { visible: true });
      },
      async (page) => {
        const menu = await page.$('[data-testid="mobile-menu"]');
        const isVisible = await page.evaluate(el => {
          const style = window.getComputedStyle(el);
          return style.display !== 'none' && style.visibility !== 'hidden';
        }, menu);
        return isVisible;
      }
    ),
  ]),
];

// ============================================================
// CLI Interface
// ============================================================

async function main() {
  const args = process.argv.slice(2);
  const updateBaselines = args.includes('--update-baselines');
  const featureFilter = args.find(a => a.startsWith('--feature='))?.split('=')[1];

  const verifier = new VisualVerifier();

  try {
    await verifier.initialize();

    // Filter features if specified
    let testsToRun = featureTests;
    if (featureFilter) {
      testsToRun = featureTests.filter(t => t.id === featureFilter);
      if (testsToRun.length === 0) {
        console.error(`Feature '${featureFilter}' not found`);
        process.exit(1);
      }
    }

    // Run tests
    for (const test of testsToRun) {
      const result = await verifier.verifyFeature(test, updateBaselines);
      await verifier.updateFeatureList(test.id, result.passed);
    }

    // Generate report
    const report = verifier.generateReport();
    await fs.writeFile('visual-verification-report.md', report);
    console.log('\nReport written to visual-verification-report.md');

    // Summary
    const passed = verifier.results.filter(r => r.passed).length;
    const total = verifier.results.length;
    console.log(`\n${'='.repeat(50)}`);
    console.log(`SUMMARY: ${passed}/${total} features passed visual verification`);

    process.exit(passed === total ? 0 : 1);

  } finally {
    await verifier.cleanup();
  }
}

main().catch(error => {
  console.error('Fatal error:', error);
  process.exit(1);
});
```

**Installation and Usage:**

```bash
# Install dependencies
npm install puppeteer pngjs pixelmatch

# Run all visual tests
node visual-verify.js

# Run specific feature
node visual-verify.js --feature=auth-002

# Update baseline screenshots (after intentional UI changes)
node visual-verify.js --update-baselines
```

**CI/CD Integration:**

```yaml
# .github/workflows/visual-tests.yml
visual-verification:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4

    - name: Install dependencies
      run: npm ci

    - name: Start dev server
      run: npm run dev &

    - name: Wait for server
      run: npx wait-on http://localhost:3000

    - name: Run visual verification
      run: node visual-verify.js

    - name: Upload screenshots
      if: failure()
      uses: actions/upload-artifact@v3
      with:
        name: visual-verification-screenshots
        path: screenshots/
```

---

## Checkpoint Evidence

### What Makes a Good Checkpoint

At phase boundaries, create evidence that the phase is truly complete:

**Checkpoint Evidence Bundle:**
1. **Feature Status** - All features in phase marked passing
2. **Test Results** - All tests pass
3. **Build Verification** - Application builds successfully
4. **Visual Evidence** - Screenshots showing features work
5. **Known Issues** - Any issues deferred to later phases

### Phase Transition Criteria

Only proceed to the next phase when:
- All features in current phase pass
- No critical issues remain
- Build and tests are green
- Human has reviewed and approved transition

### Complete Checkpoint Validation Script

Use this script to validate phase completion before proceeding:

```python
#!/usr/bin/env python3
"""
validate_checkpoint.py - Validate phase completion before proceeding

This script ensures all criteria are met before moving to the next phase.
It generates evidence that can be included in checkpoint commits.

Usage:
    python validate_checkpoint.py foundation
    python validate_checkpoint.py core-features --strict
    python validate_checkpoint.py --phase 2 --output checkpoint-report.md
"""

import argparse
import json
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime
from pathlib import Path
from typing import Optional
from enum import Enum


class CheckStatus(Enum):
    PASS = "pass"
    FAIL = "fail"
    WARN = "warn"
    SKIP = "skip"


@dataclass
class CheckResult:
    name: str
    status: CheckStatus
    message: str
    details: Optional[dict] = None


@dataclass
class PhaseReport:
    phase_name: str
    timestamp: datetime
    checks: list[CheckResult] = field(default_factory=list)
    overall_passed: bool = False
    blocking_issues: list[str] = field(default_factory=list)
    warnings: list[str] = field(default_factory=list)


class CheckpointValidator:
    """Validates that a phase is ready for completion."""

    def __init__(self, project_dir: Path = Path(".")):
        self.project_dir = project_dir
        self.feature_list_path = project_dir / "feature_list.json"
        self.progress_path = project_dir / "progress.txt"

    def load_feature_list(self) -> dict:
        """Load and parse feature_list.json."""
        if not self.feature_list_path.exists():
            raise FileNotFoundError("feature_list.json not found")

        with open(self.feature_list_path) as f:
            return json.load(f)

    def validate_feature_completion(self, phase_name: str) -> CheckResult:
        """Check that all features in the phase are marked as passing."""
        try:
            data = self.load_feature_list()
            features = data.get("features", [])

            phase_features = [
                f for f in features
                if f.get("phase", "").lower() == phase_name.lower()
            ]

            if not phase_features:
                return CheckResult(
                    name="Feature Completion",
                    status=CheckStatus.WARN,
                    message=f"No features found for phase '{phase_name}'",
                    details={"available_phases": list(set(f.get("phase") for f in features))}
                )

            incomplete = [f for f in phase_features if not f.get("passes", False)]

            if incomplete:
                return CheckResult(
                    name="Feature Completion",
                    status=CheckStatus.FAIL,
                    message=f"{len(incomplete)}/{len(phase_features)} features incomplete",
                    details={
                        "incomplete": [
                            {"id": f["id"], "description": f.get("description", "")}
                            for f in incomplete
                        ]
                    }
                )

            return CheckResult(
                name="Feature Completion",
                status=CheckStatus.PASS,
                message=f"All {len(phase_features)} features complete",
                details={"completed_features": [f["id"] for f in phase_features]}
            )

        except FileNotFoundError as e:
            return CheckResult(
                name="Feature Completion",
                status=CheckStatus.FAIL,
                message=str(e)
            )
        except json.JSONDecodeError as e:
            return CheckResult(
                name="Feature Completion",
                status=CheckStatus.FAIL,
                message=f"Invalid JSON in feature_list.json: {e}"
            )

    def validate_tests(self) -> CheckResult:
        """Run test suite and verify all tests pass."""
        try:
            # Detect test framework
            if (self.project_dir / "package.json").exists():
                cmd = ["npm", "test"]
            elif (self.project_dir / "pytest.ini").exists() or \
                 (self.project_dir / "pyproject.toml").exists():
                cmd = ["pytest", "--tb=short", "-q"]
            else:
                cmd = ["pytest", "--tb=short", "-q"]  # Default to pytest

            result = subprocess.run(
                cmd,
                cwd=self.project_dir,
                capture_output=True,
                text=True,
                timeout=300
            )

            if result.returncode == 0:
                # Parse test count from output
                test_count = "unknown"
                for line in result.stdout.split('\n'):
                    if 'passed' in line.lower():
                        test_count = line.strip()
                        break

                return CheckResult(
                    name="Test Suite",
                    status=CheckStatus.PASS,
                    message=f"All tests pass: {test_count}",
                    details={"output": result.stdout[-1000:]}
                )
            else:
                return CheckResult(
                    name="Test Suite",
                    status=CheckStatus.FAIL,
                    message="Tests failing",
                    details={
                        "stdout": result.stdout[-500:],
                        "stderr": result.stderr[-500:]
                    }
                )

        except subprocess.TimeoutExpired:
            return CheckResult(
                name="Test Suite",
                status=CheckStatus.FAIL,
                message="Tests timed out after 5 minutes"
            )
        except FileNotFoundError as e:
            return CheckResult(
                name="Test Suite",
                status=CheckStatus.WARN,
                message=f"Test command not found: {e}"
            )

    def validate_build(self) -> CheckResult:
        """Verify the project builds successfully."""
        try:
            # Detect build command
            if (self.project_dir / "package.json").exists():
                cmd = ["npm", "run", "build"]
            elif (self.project_dir / "Cargo.toml").exists():
                cmd = ["cargo", "build", "--release"]
            elif (self.project_dir / "go.mod").exists():
                cmd = ["go", "build", "./..."]
            else:
                # Python projects might not have a build step
                return CheckResult(
                    name="Build",
                    status=CheckStatus.SKIP,
                    message="No build step detected for this project type"
                )

            result = subprocess.run(
                cmd,
                cwd=self.project_dir,
                capture_output=True,
                text=True,
                timeout=300
            )

            if result.returncode == 0:
                return CheckResult(
                    name="Build",
                    status=CheckStatus.PASS,
                    message="Build successful"
                )
            else:
                return CheckResult(
                    name="Build",
                    status=CheckStatus.FAIL,
                    message="Build failed",
                    details={"stderr": result.stderr[-500:]}
                )

        except subprocess.TimeoutExpired:
            return CheckResult(
                name="Build",
                status=CheckStatus.FAIL,
                message="Build timed out"
            )

    def validate_linting(self) -> CheckResult:
        """Run linting checks."""
        try:
            # Detect linter
            if (self.project_dir / "package.json").exists():
                cmd = ["npm", "run", "lint"]
            else:
                cmd = ["ruff", "check", "."]

            result = subprocess.run(
                cmd,
                cwd=self.project_dir,
                capture_output=True,
                text=True,
                timeout=60
            )

            if result.returncode == 0:
                return CheckResult(
                    name="Linting",
                    status=CheckStatus.PASS,
                    message="No linting issues"
                )
            else:
                issue_count = result.stdout.count('\n')
                return CheckResult(
                    name="Linting",
                    status=CheckStatus.WARN,  # Warnings don't block
                    message=f"{issue_count} linting issues",
                    details={"issues": result.stdout[-500:]}
                )

        except FileNotFoundError:
            return CheckResult(
                name="Linting",
                status=CheckStatus.SKIP,
                message="Linter not found"
            )

    def validate_no_uncommitted_changes(self) -> CheckResult:
        """Ensure all changes are committed."""
        result = subprocess.run(
            ["git", "status", "--porcelain"],
            cwd=self.project_dir,
            capture_output=True,
            text=True
        )

        if result.stdout.strip():
            changed_files = result.stdout.strip().split('\n')
            return CheckResult(
                name="Git Status",
                status=CheckStatus.WARN,
                message=f"{len(changed_files)} uncommitted changes",
                details={"files": changed_files[:10]}  # First 10 files
            )

        return CheckResult(
            name="Git Status",
            status=CheckStatus.PASS,
            message="All changes committed"
        )

    def validate_progress_updated(self) -> CheckResult:
        """Check that progress.txt is up to date."""
        if not self.progress_path.exists():
            return CheckResult(
                name="Progress Tracking",
                status=CheckStatus.WARN,
                message="progress.txt not found"
            )

        content = self.progress_path.read_text()

        # Check for recent update (today's date should appear)
        today = datetime.now().strftime("%Y-%m-%d")
        if today in content:
            return CheckResult(
                name="Progress Tracking",
                status=CheckStatus.PASS,
                message="Progress updated today"
            )

        return CheckResult(
            name="Progress Tracking",
            status=CheckStatus.WARN,
            message="Progress may not be current (no today's date found)"
        )

    def validate_phase(self, phase_name: str, strict: bool = False) -> PhaseReport:
        """Run all validation checks for a phase."""
        report = PhaseReport(
            phase_name=phase_name,
            timestamp=datetime.now()
        )

        # Required checks (must pass)
        required_checks = [
            self.validate_feature_completion(phase_name),
            self.validate_tests(),
            self.validate_build(),
        ]

        # Advisory checks (warnings don't block)
        advisory_checks = [
            self.validate_linting(),
            self.validate_no_uncommitted_changes(),
            self.validate_progress_updated(),
        ]

        report.checks = required_checks + advisory_checks

        # Determine overall status
        for check in required_checks:
            if check.status == CheckStatus.FAIL:
                report.blocking_issues.append(f"{check.name}: {check.message}")

        for check in advisory_checks:
            if check.status == CheckStatus.WARN:
                report.warnings.append(f"{check.name}: {check.message}")
            elif check.status == CheckStatus.FAIL and strict:
                report.blocking_issues.append(f"{check.name}: {check.message}")

        report.overall_passed = len(report.blocking_issues) == 0

        return report


def generate_report_markdown(report: PhaseReport) -> str:
    """Generate a Markdown report."""
    status_emoji = {
        CheckStatus.PASS: "[PASS]",
        CheckStatus.FAIL: "[FAIL]",
        CheckStatus.WARN: "[WARN]",
        CheckStatus.SKIP: "[SKIP]",
    }

    lines = [
        f"# Checkpoint Validation Report",
        f"",
        f"**Phase:** {report.phase_name}",
        f"**Timestamp:** {report.timestamp.isoformat()}",
        f"**Status:** {'PASSED' if report.overall_passed else 'BLOCKED'}",
        f"",
        "## Check Results",
        "",
    ]

    for check in report.checks:
        emoji = status_emoji[check.status]
        lines.append(f"### {emoji} {check.name}")
        lines.append(f"")
        lines.append(f"{check.message}")
        if check.details:
            lines.append(f"")
            lines.append(f"<details>")
            lines.append(f"<summary>Details</summary>")
            lines.append(f"")
            lines.append(f"```json")
            lines.append(json.dumps(check.details, indent=2)[:1000])
            lines.append(f"```")
            lines.append(f"</details>")
        lines.append(f"")

    if report.blocking_issues:
        lines.extend([
            "## Blocking Issues",
            "",
            "The following must be resolved before proceeding:",
            "",
        ])
        for issue in report.blocking_issues:
            lines.append(f"- {issue}")
        lines.append("")

    if report.warnings:
        lines.extend([
            "## Warnings",
            "",
            "These should be addressed but don't block progress:",
            "",
        ])
        for warning in report.warnings:
            lines.append(f"- {warning}")
        lines.append("")

    if report.overall_passed:
        lines.extend([
            "## Next Steps",
            "",
            f"Phase '{report.phase_name}' validation passed. Ready to proceed to next phase.",
            "",
            "Recommended actions:",
            "1. Create checkpoint commit with this report",
            "2. Tag the commit: `git tag checkpoint-{phase_name}-$(date +%Y%m%d)`",
            "3. Update progress.txt with phase completion",
            "4. Begin next phase",
        ])

    return '\n'.join(lines)


def main():
    parser = argparse.ArgumentParser(description="Validate phase checkpoint")
    parser.add_argument("phase", nargs="?", default="foundation",
                       help="Phase name to validate")
    parser.add_argument("--strict", action="store_true",
                       help="Treat warnings as failures")
    parser.add_argument("--output", "-o", help="Output report to file")
    parser.add_argument("--json", action="store_true",
                       help="Output as JSON")
    parser.add_argument("--project-dir", "-d", default=".",
                       help="Project directory")

    args = parser.parse_args()

    validator = CheckpointValidator(Path(args.project_dir))

    print(f"Validating checkpoint for phase: {args.phase}")
    print("=" * 50)

    report = validator.validate_phase(args.phase, strict=args.strict)

    # Print summary to console
    for check in report.checks:
        status = check.status.value.upper()
        print(f"  [{status}] {check.name}: {check.message}")

    print("=" * 50)

    if report.overall_passed:
        print(f"CHECKPOINT VALIDATED - Phase '{args.phase}' ready to complete")
    else:
        print(f"CHECKPOINT BLOCKED - {len(report.blocking_issues)} issue(s) must be resolved")
        for issue in report.blocking_issues:
            print(f"  - {issue}")

    # Generate output
    if args.json:
        from dataclasses import asdict
        output = json.dumps(asdict(report), default=str, indent=2)
    else:
        output = generate_report_markdown(report)

    if args.output:
        Path(args.output).write_text(output)
        print(f"\nReport written to {args.output}")
    elif args.json:
        print(output)

    sys.exit(0 if report.overall_passed else 1)


if __name__ == "__main__":
    main()
```

**Usage Examples:**

```bash
# Validate foundation phase
python validate_checkpoint.py foundation

# Strict mode (warnings become failures)
python validate_checkpoint.py core-features --strict

# Generate report for commit message
python validate_checkpoint.py phase-2 -o checkpoint-report.md

# Get JSON for CI/CD
python validate_checkpoint.py --json > checkpoint.json
```

**Integration with Git:**

```bash
# After validation passes, create checkpoint commit
python validate_checkpoint.py foundation -o .checkpoint-report.md && \
git add -A && \
git commit -m "checkpoint: complete Phase 1 (Foundation)

$(cat .checkpoint-report.md | head -30)"
```

### Checkpoint Commit Convention

Use consistent commit messages for checkpoints:
```
checkpoint: complete Phase 1 (Foundation)

Features completed: 8/8
Tests passing: 47/47
Known issues: 0 blocking, 2 deferred to Phase 4

See checkpoint-evidence/phase-1.md for full report.
```

---

## Key Takeaways

1. **Phase your work** — 150-200 requirements per phase maximum
2. **Follow the session lifecycle** — Start, work, and end protocols ensure continuity
3. **Use initializer + coding agents** — Setup vs. implementation are different modes
4. **Track features in JSON** — Harder to accidentally corrupt than Markdown
5. **Maintain progress.txt** — Essential for multi-session continuity
6. **Keep three sources synchronized** — feature_list.json, progress.txt, and git commits
7. **Have a recovery protocol** — Know how to get unstuck when things go wrong
8. **Verify end-to-end** — Never mark complete without testing
9. **Add visual verification** — Catch what unit tests miss
10. **Create checkpoints with evidence** — Not just commits, but proof of completion
11. **Test at session start** — Verify the app works before adding features
12. **Leave clean state** — Next session shouldn't need to fix your mess

---

## Practical Exercise: Build a Mini Application

Build a small but complete application using these patterns:

### The Application
A simple task manager with:
- User authentication (signup, login, logout)
- Create, read, update, delete tasks
- Mark tasks as complete
- Filter tasks by status

### Your Process

**Step 1: Create Phased PRD**
- Phase 1: Auth + data models
- Phase 2: Task CRUD
- Phase 3: Filtering and polish

**Step 2: Run Initializer Agent**
- Set up project structure
- Create feature_list.json
- Create progress.txt
- Initial commit

**Step 3: Run Coding Agent (multiple sessions)**
- Follow the incremental progress pattern
- Track completion in feature_list.json
- Update progress.txt after each feature

**Step 4: Reflect**
- How did phased approach help?
- What failure modes did you encounter?
- What would you do differently?

---

## Next Step

[Step 12: Production Deployment and Operations](12-production-deployment-and-operations.md) — Learn to deploy and operate AI-assisted applications in production.

---

## Sources

### Primary References

1. **Anthropic Engineering Blog**
   - "Effective Harnesses for Long-Running Agents" (2025) - https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
   - "Claude 4 Prompting Guide - Multi-Context Workflows"
   - State persistence strategies for agentic workflows

2. **Academic Papers**
   - Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)
     - Checkpoint evidence and phase transition patterns
   - "Lifecycle-Aware Code Generation." arXiv:2510.24019 (2025)

3. **Industry Research**
   - HumanLayer. "Advanced Context Engineering for Coding Agents" (2025)
     - 150-200 instruction limit per phase finding
     - https://github.com/humanlayer/advanced-context-engineering-for-coding-agents

4. **Stanford CS146S: The Modern Software Developer**
   - Week 8: Automated UI and App Building
   - Multi-session agent patterns

5. **Tool Documentation**
   - Vercel AI: Visual verification patterns for AI-generated UIs

---

## Further Reading

1. **"Effective Harnesses for Long-Running Agents"** - Anthropic's definitive guide to managing agents across sessions, including state persistence and recovery.
   https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents

2. **HumanLayer Context Engineering Research** - Practical research on instruction limits and multi-session workflows.
   https://github.com/humanlayer/advanced-context-engineering-for-coding-agents

3. **SASE Paper: Structured Agentic Software Engineering** (arXiv:2509.06216v2) - Introduces checkpoint evidence and phase transition patterns for large-scale AI development.
   https://arxiv.org/abs/2509.06216

4. **"Specs Are the New Source Code"** - Ravi Mehta's article on treating PRDs as executable specifications for AI agents.
   https://blog.ravi-mehta.com/p/specs-are-the-new-source-code

5. **Puppeteer MCP Server** - Tool for visual verification through automated screenshots, useful for UI validation.
   https://github.com/anthropics/puppeteer-mcp-server
