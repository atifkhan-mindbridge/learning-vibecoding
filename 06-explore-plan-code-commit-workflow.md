# Step 06: Explore-Plan-Code-Commit Workflow

**Level:** Intermediate
**Time:** ~30 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Master Anthropic's 4-phase workflow for AI-assisted development
2. Understand why skipping research degrades quality
3. Use subagents effectively for parallel exploration
4. Maintain context hygiene with `/clear` and `/compact`

---

With your context files from [Step 05](05-context-engineering-and-documentation.md) in place, you're ready to learn a structured workflow that ties everything together. This chapter introduces Anthropic's 4-phase workflow that consistently produces better results than ad-hoc prompting.

## The 4-Phase Workflow

Anthropic's internal teams developed a structured workflow that consistently produces better results than ad-hoc prompting [^anthropic]. It has four phases:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  EXPLORE │ → │   PLAN   │ → │   CODE   │ → │  COMMIT  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
  Research       Design          Implement       Finalize
  context        solution        code            changes
```

### Why This Order Matters

The most common failure mode in AI-assisted coding is **jumping straight to code**. When developers skip exploration and planning:

- AI makes incorrect assumptions about the codebase
- Generated code doesn't fit existing patterns
- Edge cases are missed
- Integration issues arise
- More time is spent debugging than was saved

The 4-phase workflow front-loads understanding, which dramatically improves first-attempt success rates.

---

## LoopScript: Workflow as Code

### What Is LoopScript?

**LoopScript** is a concept from the SASE (Structured Agentic Software Engineering) framework [^sase] that treats workflows as declarative specifications rather than informal patterns. Just as infrastructure-as-code made server configuration reproducible, LoopScript makes development workflows explicit and auditable.

### Why LoopScript Matters

The following diagram illustrates the difference between implicit workflows and explicit LoopScript specifications:

```
┌─────────────────────────────────────────────────────────────────┐
│              LOOPSCRIPT: WORKFLOW AS CODE                       │
│                                                                 │
│   Traditional Workflow:                                         │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  "Explore, then plan, then code, then commit"           │   │
│   │                                                         │   │
│   │  (Implicit, varies by developer, hard to audit)         │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   LoopScript Workflow:                                          │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  phases:                                                │   │
│   │    - explore: {steps: [...], checkpoint: {...}}         │   │
│   │    - plan: {thinking: "hard", checkpoint: {...}}        │   │
│   │    - code: {for_each: step, checkpoint: {...}}          │   │
│   │    - commit: {steps: [...], checkpoint: {...}}          │   │
│   │                                                         │   │
│   │  (Explicit, reproducible, auditable)                    │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Benefits:                                                     │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  • Consistent across team members                       │   │
│   │  • Checkpoints prevent drift from the plan              │   │
│   │  • Evidence artifacts for review                        │   │
│   │  • Can be partially automated                           │   │
│   │  • New developers learn structured approach             │   │
│   │  • Easier to identify which phases need improvement     │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Traditional workflow:**
- "Explore, then plan, then code, then commit"
- Implicit, varies by developer
- Hard to audit or improve
- Inconsistent across team

**LoopScript workflow:**
- Phases explicitly defined with checkpoints
- Each phase has evidence requirements
- Human review gates are specified
- Reproducible and teachable

### Core LoopScript Concepts

**Phases:** Named stages with defined inputs, steps, and outputs

**Checkpoints:** Gates between phases that require evidence before proceeding

**Evidence:** Artifacts that prove a phase was completed correctly (exploration summaries, test results, review approvals)

**Human Review Gates:** Points where human judgment is required before continuing

### Applying LoopScript to the 4-Phase Workflow

Think of each phase with explicit checkpoints:

**Explore Phase**
- Steps: Read relevant code, identify patterns, understand constraints
- Checkpoint evidence: Exploration summary documenting findings
- Human review: Optional (verify AI understood correctly)

**Plan Phase**
- Steps: Design solution, identify changes, assess risks
- Checkpoint evidence: Implementation plan document
- Human review: Required (approve approach before coding)

**Code Phase**
- Steps: Implement changes, run tests, verify each step
- Checkpoint evidence: Tests pass, type checks pass
- Human review: On failure (when tests don't pass after reasonable iteration)

**Commit Phase**
- Steps: Create commit, update documentation, prepare PR
- Checkpoint evidence: PR created, docs updated
- Human review: Standard code review process

### Benefits of the LoopScript Mindset

1. **Consistency:** Team members follow the same process
2. **Auditability:** Can trace how decisions were made
3. **Improvement:** Can identify which phases need refinement
4. **Training:** New team members learn structured approach
5. **Automation potential:** Well-defined workflows can be partially automated

### LoopScript Example: Feature Implementation

While LoopScript is a conceptual framework (not a literal file format you run), expressing workflows declaratively helps make them explicit and teachable:

```yaml
# search-feature.loopscript.yaml
# A LoopScript specification for implementing product search

workflow: implement-product-search
version: 1.0
created: 2024-01-15
author: team-lead

context_files:
  - AGENTS.md
  - CLAUDE.md
  - PRD.md#phase-2-search

phases:
  - name: explore
    description: "Understand existing code before making changes"
    duration_estimate: "10-15 minutes"
    steps:
      - action: read
        targets:
          - "src/features/catalog/"
          - "src/lib/data-fetching.ts"
          - "src/hooks/useProducts.ts"
        purpose: "Understand current data flow"

      - action: identify
        targets: "existing search utilities"
        look_in:
          - "src/lib/"
          - "src/utils/"

      - action: summarize
        output: "exploration-findings.md"
        include:
          - "Data fetching patterns used"
          - "State management approach"
          - "Existing utilities to reuse"
          - "Files that will need modification"

    checkpoint:
      evidence: "exploration-findings.md exists and covers all include items"
      human_review: optional
      gate_question: "Do we understand the codebase well enough to plan?"
      on_failure: "Continue exploring; ask clarifying questions"

  - name: plan
    description: "Design solution with implementation steps"
    duration_estimate: "15-20 minutes"
    thinking_mode: "think hard"
    steps:
      - action: design
        constraints:
          - "Must use existing data fetching patterns"
          - "Search must work with current product model"
          - "Must support debounced input (300ms)"
        requirements:
          - "Search by name and description"
          - "Filter by category"
          - "Filter by price range"

      - action: create_plan
        output: "implementation-plan.md"
        format:
          - "Step number and title"
          - "Files to create or modify"
          - "What to implement"
          - "How to verify"
          - "Dependencies on other steps"

    checkpoint:
      evidence: "implementation-plan.md with numbered steps"
      human_review: required
      gate_question: "Is this plan comprehensive and does it follow existing patterns?"
      approval_criteria:
        - "Plan addresses all requirements"
        - "Each step has verification method"
        - "No missing dependencies"
        - "Estimated effort is reasonable"
      on_rejection: "Revise plan based on feedback"

  - name: code
    description: "Implement solution step by step"
    iteration: "for each step in implementation-plan.md"
    steps:
      - action: implement
        scope: "current step only"
        constraints:
          - "Do NOT modify tests to make them pass"
          - "Follow patterns in codebase"
          - "Add TypeScript types for all new code"

      - action: verify
        commands:
          - "npm run typecheck"
          - "npm test -- --grep 'search'"
        success_criteria: "No type errors, relevant tests pass"

    checkpoint:
      evidence: "tests pass for current step"
      human_review: on_failure
      max_iterations: 3
      on_max_iterations: "Create CRP explaining blockers"

  - name: commit
    description: "Finalize and document changes"
    steps:
      - action: create_commit
        format: "conventional commits"
        message_template: "feat(search): {description}"

      - action: update_docs
        if_applicable:
          - "CHANGELOG.md - Add new feature"
          - "README.md - If setup changed"
          - "API docs - If endpoints added"

      - action: create_pr
        include:
          - "Summary of changes"
          - "How to test"
          - "Screenshots if UI changed"

    checkpoint:
      evidence: "PR created with all documentation"
      human_review: "standard code review process"

completion_criteria:
  - "All tests pass"
  - "PR approved"
  - "Documentation updated"
  - "Feature works as specified"
```

**Using this LoopScript:**

This YAML isn't executed directly—it's a specification you and Claude reference:

```
Let's implement product search following our LoopScript pattern.
We're starting the EXPLORE phase. Per the spec, I need to:
1. Read the catalog code in src/features/catalog/
2. Identify existing search utilities
3. Create an exploration-findings.md

Let's begin. Read the catalog directory and summarize what you find.
Don't write any code yet.
```

The explicit specification makes the workflow teachable, auditable, and consistent.

---

## Phase 1: Explore

### The Goal

Build understanding before writing code. Gather context about:
- How the codebase currently works
- What patterns exist
- What constraints apply
- What related code might be affected

### How to Execute

**Key principle:** Explicitly tell Claude NOT to write code yet.

**Example prompts:**

```
Read the authentication system in src/auth/ and explain how it works.
Don't write any code - just help me understand the current implementation.
```

```
I need to add a new API endpoint for user preferences. Before we implement:
1. Read the existing endpoints in src/api/
2. Understand the patterns they follow
3. Identify where preferences would fit
4. Note any shared utilities we should use

Don't write any implementation code yet.
```

### Using Subagents for Exploration

For complex problems, use subagents to explore in parallel:

```
Use subagents to research the following in parallel:
1. How error handling works in this codebase
2. What logging patterns are used
3. How similar features were tested

Summarize the findings for each, then we'll plan the implementation.
```

**Why subagents help:**
- Each subagent has its own context window
- They can explore different areas simultaneously
- They return only relevant findings (not full context)
- Preserves your main context for implementation

### Subagent Deep Dive

Understanding how subagents work helps you use them effectively:

```
┌─────────────────────────────────────────────────────────────────┐
│              SUBAGENT ARCHITECTURE                              │
│                                                                 │
│   Main Agent                                                    │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  "Use subagents to research in parallel:"               │   │
│   │                                                         │   │
│   │    ┌───────────┐   ┌───────────┐   ┌───────────┐       │   │
│   │    │ Subagent  │   │ Subagent  │   │ Subagent  │       │   │
│   │    │    1      │   │    2      │   │    3      │       │   │
│   │    │  (Auth)   │   │ (Logging) │   │ (Testing) │       │   │
│   │    └─────┬─────┘   └─────┬─────┘   └─────┬─────┘       │   │
│   │          │               │               │              │   │
│   │          ▼               ▼               ▼              │   │
│   │    [Findings]      [Findings]      [Findings]          │   │
│   │          │               │               │              │   │
│   │          └───────────────┼───────────────┘              │   │
│   │                          ▼                              │   │
│   │               [Synthesized Summary]                     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Key Properties:                                               │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  • Each subagent has its own context window             │   │
│   │  • Results are summarized, not full transcripts         │   │
│   │  • Subagents don't see main conversation history        │   │
│   │  • Main agent synthesizes findings                      │   │
│   │  • Parallel execution where possible                    │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Context Isolation**
Each subagent spawns with:
- The project's context files (AGENTS.md, CLAUDE.md)
- A focused task prompt you provide
- Fresh context window (no conversation history)

This isolation is both a strength (parallel exploration, focused work) and a limitation (subagents don't know what you discussed in the main conversation).

**Result Synthesis**
When subagents complete, the main agent receives:
- Summarized findings (not full exploration transcripts)
- Key facts and conclusions
- References to files examined

The main agent synthesizes these into a coherent picture.

**Effective Subagent Prompts**
Give subagents:
- Clear, focused scope ("How does authentication work" not "Tell me about the codebase")
- Specific questions to answer
- Format for reporting findings

**When to Use Subagents**
- Exploring multiple unrelated areas in parallel
- Research tasks that benefit from focused attention
- Independent verification of work
- When context window is becoming constrained

**When NOT to Use Subagents**
- Tasks requiring knowledge of current conversation state
- Tightly interdependent research questions
- Simple tasks that don't benefit from parallelization

### What to Look For

During exploration, have Claude identify:

| Aspect | Questions |
|--------|-----------|
| Patterns | What conventions does this codebase follow? |
| Dependencies | What existing code can we reuse? |
| Constraints | What must we avoid breaking? |
| Examples | What similar implementations can we reference? |
| Edge cases | What special handling exists? |

---

## Phase 2: Plan

### The Goal

Design the solution before implementing. Create a detailed plan that:
- Specifies what changes will be made
- Orders the changes logically
- Identifies risks and edge cases
- Can be reviewed before coding begins

### How to Execute

**Trigger extended thinking with thinking keywords:**

```
Based on what you learned about our auth system, think about how to add
OAuth support. Create a detailed plan including:
1. What files need to change
2. What new files to create
3. Order of implementation
4. Potential edge cases
5. Testing strategy

Don't implement yet - just create the plan.
```

For harder problems:

```
Think hard about the best approach to migrate our database schema.
Consider:
- Backward compatibility
- Zero-downtime deployment
- Data integrity
- Rollback strategy

Create a step-by-step migration plan with risk assessment.
```

### Plan Quality Checklist

A good plan includes:

- [ ] Clear list of files to modify
- [ ] Order of changes (what depends on what)
- [ ] New files to create
- [ ] Tests to write
- [ ] Edge cases to handle
- [ ] Risks and mitigations
- [ ] Verification steps

### Checkpoint Evidence for Plans

Following the LoopScript concept, the Plan phase should produce specific evidence:

```
┌─────────────────────────────────────────────────────────────────┐
│              CHECKPOINT EVIDENCE FLOW                           │
│                                                                 │
│   Explore ────► Checkpoint 1                                    │
│                 Evidence: exploration-summary.md                │
│                 Gate: "Do we understand the codebase?"          │
│                          │                                      │
│                          ▼ Yes                                  │
│                                                                 │
│   Plan    ────► Checkpoint 2                                    │
│                 Evidence: implementation-plan.md                │
│                 Gate: "Is the plan comprehensive?"              │
│                          │                                      │
│                          ▼ Yes                                  │
│                                                                 │
│   Code    ────► Checkpoint 3 (per step)                         │
│                 Evidence: tests-pass                            │
│                 Gate: "Does this step work?"                    │
│                          │                                      │
│                          ▼ Yes (all steps)                      │
│                                                                 │
│   Commit  ────► Checkpoint 4                                    │
│                 Evidence: PR-created, docs-updated              │
│                 Gate: "Is this ready for review?"               │
│                          │                                      │
│                          ▼ Yes                                  │
│                                                                 │
│                 [Ready for Human Code Review]                   │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Each checkpoint requires:                              │   │
│   │  • Specific evidence artifact                           │   │
│   │  • Clear gate question (yes/no)                         │   │
│   │  • Human review option (required/optional/on-failure)   │   │
│   │  • Action if gate fails (redo, revise, escalate)        │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Required evidence:**
- Implementation plan document (can be in PR description, GitHub issue, or local file)
- List of files to change with expected modifications
- Risk assessment for complex changes
- Defined acceptance criteria

**Human review gate:**
Before moving to Code phase, verify:
- Does the plan address the actual requirements?
- Are there approaches not considered?
- Are the risks acceptable?
- Is the scope appropriate?

This checkpoint prevents wasted effort implementing the wrong solution.

### Saving Plans

For complex features, save the plan before implementing:

**Option 1: GitHub Issue**
```
Create a GitHub issue with this implementation plan so we can track it.
```

**Option 2: Local File**
```
Save this plan to docs/plans/oauth-implementation.md so we can reference it.
```

**Why save plans?**
- Allows review before coding
- Creates documentation
- Can reset to this point if implementation goes wrong
- Helps with handoffs

---

## Phase 3: Code

### The Goal

Implement the solution according to the plan. Focus on:
- Following the planned approach
- Writing code that fits existing patterns
- Including verification at each step

### How to Execute

```
Now implement the plan we created. Start with step 1: creating the
OAuth configuration module.

After each step:
1. Verify the change works
2. Run relevant tests
3. Move to the next step only if verification passes
```

### Verification During Implementation

Include explicit verification instructions:

```
Implement the OAuth callback handler. After writing the code:
1. Run the type checker (pnpm typecheck)
2. Run the auth tests (pnpm test src/auth/)
3. Verify no existing tests broke

Only proceed if all verifications pass.
```

### Handling Problems

If implementation reveals issues:

**Option 1: Revise plan**
```
The plan assumed we could use the existing session store, but it doesn't
support OAuth tokens. Let's revise the plan to include session store changes.
```

**Option 2: Ask for alternatives**
```
This approach is causing circular dependency issues. Think hard about
alternative approaches that avoid this problem.
```

**Option 3: Create a CRP (Consultation Request Pack)**
When you're genuinely stuck, create a structured request for human decision-making.

---

## CRP: Consultation Request Pack

### When to Use a CRP

A **Consultation Request Pack (CRP)** is a structured document the AI creates when it encounters a decision that requires human judgment. Use CRPs when:

- Multiple valid approaches exist with significant tradeoffs
- The decision has business or architectural implications beyond the current task
- Technical constraints create unexpected limitations
- Requirements are ambiguous and assumptions are risky

### CRP Structure

A good CRP includes:

**Context:** What the AI is working on and how it got to this decision point

**Current State:** What's been accomplished, what's in progress, what's blocked

**Specific Question:** A clear, answerable question (not vague uncertainty)

**Tradeoffs Identified:** Analysis of options with pros and cons

**Recommendation:** The AI's suggested approach, if it has one

**Decision Needed:** What specific decision the human needs to make

### How CRPs Improve Collaboration

**Without CRPs:**
- AI gets stuck and produces vague "I'm not sure" responses
- Developer doesn't understand why progress stalled
- Time wasted on back-and-forth clarification

**With CRPs:**
- AI clearly articulates the decision point
- Developer sees options analyzed
- Decision can be made quickly with full context
- Work resumes immediately after decision

### Requesting a CRP

When the AI seems uncertain or you want structured options:

```
Create a CRP for the approach decision. Include:
- What you've determined so far
- The specific decision you need
- Options with tradeoffs
- Your recommendation if you have one
```

### Complete CRP Template Example

Here's a fully worked CRP that demonstrates the pattern:

```markdown
# Consultation Request Pack

## Context
**Task:** Implementing product search for the e-commerce dashboard
**Current Phase:** Code (Step 2 of 5)
**Session:** search-feature-2024-01-15

I'm implementing the search API endpoint and have hit a decision point
that requires human input due to significant architectural implications.

---

## Current State

**Completed:**
- [x] Step 1: Created SearchInput component with debouncing
- [x] Search UI connected to state

**In Progress:**
- [ ] Step 2: Search API endpoint - BLOCKED

**Not Started:**
- [ ] Step 3: Connect UI to API
- [ ] Step 4: Add filters
- [ ] Step 5: Tests and documentation

---

## Specific Question

**Should we use PostgreSQL full-text search or add Elasticsearch for product search?**

The product catalog currently has ~5,000 products and is expected to grow to
~50,000 in the next year. Search needs to query product names, descriptions,
and tags.

---

## Tradeoffs Identified

| Aspect | PostgreSQL FTS | Elasticsearch |
|--------|---------------|---------------|
| **Setup complexity** | None - already have Postgres | New service, Docker config, deployment |
| **Query performance** | Good for <100k docs | Excellent at any scale |
| **Features** | Basic text search, ranking | Fuzzy matching, synonyms, facets |
| **Operational cost** | $0 additional | ~$50-100/month managed, or ops burden |
| **Development time** | 2-3 hours | 1-2 days (including setup) |
| **Future scalability** | Will need migration at scale | Ready for growth |
| **Team expertise** | Team knows Postgres well | No ES experience on team |

### PostgreSQL Implementation Sketch
```sql
-- Add tsvector column
ALTER TABLE products ADD COLUMN search_vector tsvector;
CREATE INDEX products_search_idx ON products USING GIN(search_vector);

-- Query
SELECT * FROM products
WHERE search_vector @@ plainto_tsquery('english', $1)
ORDER BY ts_rank(search_vector, plainto_tsquery('english', $1)) DESC;
```

### Elasticsearch Implementation Sketch
```typescript
// New service to maintain
const client = new Client({ node: process.env.ELASTICSEARCH_URL });

// Index mapping, sync job, query builder needed
// ~200 lines of new infrastructure code
```

---

## Risk Assessment

**PostgreSQL Risks:**
- May need to migrate later if we outgrow it (cost: 2-3 days)
- Limited relevance tuning compared to ES
- No fuzzy matching for typos

**Elasticsearch Risks:**
- New operational complexity for the team
- Index sync issues if not carefully implemented
- Overkill for current scale

---

## Recommendation

**I recommend PostgreSQL full-text search for MVP.**

Reasoning:
1. Current scale (5k products) is well within Postgres FTS capabilities
2. Zero additional infrastructure or operational burden
3. 2-3 hours vs 1-2 days implementation time
4. If we outgrow it, migrating to ES is a known path
5. "Boring technology" principle - use what we know

**Proposed migration trigger:**
- If search latency exceeds 200ms at p95, OR
- If product count exceeds 100k, OR
- If we need advanced features (fuzzy, synonyms)

---

## Decision Needed

Please confirm one of:

**Option A:** Proceed with PostgreSQL FTS (my recommendation)
- I will implement basic full-text search
- We accept limitations for now
- Document migration trigger conditions

**Option B:** Invest in Elasticsearch now
- I will set up ES infrastructure first
- Timeline extends by 1-2 days
- We get advanced features from the start

**Option C:** Other direction
- Please specify alternative approach

---

## Supporting Information

**Files reviewed:**
- `prisma/schema.prisma` - Current product model
- `src/lib/db.ts` - Database configuration
- `package.json` - No ES dependencies currently

**Relevant PRD requirements:**
- "Search by product name and description" (no fuzzy mentioned)
- "Return results in under 500ms" (Postgres can meet this)

**Team context:**
- No Elasticsearch experience on team
- Preference for managed services over self-hosted
```

**What makes this CRP effective:**

1. **Clear context:** Reviewer knows exactly where we are
2. **Specific question:** Not "what should I do" but a precise decision point
3. **Thorough analysis:** Both options explored with evidence
4. **Recommendation with reasoning:** AI isn't just punting to human
5. **Clear options:** Easy for human to say "Option A" and move on
6. **Supporting info:** Human can verify reasoning if needed

---

## Phase 4: Commit

### The Goal

Finalize changes with proper documentation and version control:
- Create meaningful commits
- Update documentation
- Create pull requests
- Ensure changes are ready for review

### How to Execute

```
The OAuth implementation is complete and tests pass. Now:
1. Create a commit with a descriptive message
2. Update the CHANGELOG.md with this new feature
3. Create a pull request with a summary of changes
```

### Commit Message Best Practices

```
Create a commit for the OAuth changes. The message should:
- Start with type (feat/fix/refactor/docs)
- Summarize what changed in the subject
- Explain why in the body
- Reference any issues
```

Example output:
```
feat(auth): add OAuth support for Google and GitHub

- Implement OAuth flow with configurable providers
- Add token refresh handling
- Include rate limiting for auth endpoints
- Add comprehensive test coverage

Closes #123
```

### Documentation Updates

```
Update documentation to reflect the OAuth changes:
1. Update README.md with setup instructions
2. Add API documentation for new endpoints
3. Update the authentication guide in docs/
```

---

## Context Hygiene

### The Context Problem

Long conversations fill the context window with:
- Old code versions
- Abandoned approaches
- Resolved discussions
- Outdated information

This **context pollution** degrades AI performance over time.

### `/clear` - Reset Context

Use `/clear` between distinct tasks to start fresh:

```
> /clear
Context cleared. Starting fresh conversation.
```

**When to use `/clear`:**
- After completing a feature
- When switching to unrelated work
- When Claude seems confused or stuck
- After major debugging sessions

### `/compact` - Compress Context

Use `/compact` when you need to preserve some context but free up space:

```
> /compact
Compacting conversation... keeping key context while reducing size.
```

**When to use `/compact`:**
- Mid-feature when context is getting full
- When you want to preserve recent progress
- Before starting a complex implementation phase

### Context Hygiene Decision Tree

Use this decision tree to determine when to use `/clear` vs `/compact`:

```
┌─────────────────────────────────────────────────────────────────┐
│              WHEN TO USE /clear vs /compact                     │
│                                                                 │
│   Starting new task?                                            │
│   ├── Yes ──► /clear (fresh start)                              │
│   │                                                             │
│   └── No ──► Is context filling up?                             │
│              ├── No ──► Continue (no action needed)             │
│              │                                                  │
│              └── Yes ──► Is recent context important?           │
│                         ├── Yes ──► /compact (keep summary)     │
│                         │                                       │
│                         └── No ──► /clear (reset)               │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   Symptoms of context problems:                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  • Claude seems confused about current state            │   │
│   │  • Repeating already-discussed solutions                │   │
│   │  • Forgetting recent instructions                       │   │
│   │  • Responses becoming generic or unfocused              │   │
│   │  • Mixing up details from different parts of project    │   │
│   │  • Suggesting approaches already tried and rejected     │   │
│   └─────────────────────────────────────────────────────────┘   │
│                                                                 │
│   Emergency: /clear and restart with fresh, focused prompt      │
└─────────────────────────────────────────────────────────────────┘
```

### Context Hygiene Best Practices

| Scenario | Action |
|----------|--------|
| Starting new feature | `/clear` |
| Same feature, new day | `/compact` then summarize yesterday's progress |
| Debugging session resolved | `/clear` |
| Mid-feature context filling up | `/compact` |
| Claude giving confused responses | `/clear` and restart with fresh context |

---

## The Complete Workflow in Practice

### Example: Adding a Search Feature (Brief Version)

**Phase 1: Explore**
```
I need to add search functionality to our product catalog. Before we start:

1. Read the current catalog code in src/features/catalog/
2. Understand how data fetching works
3. Check if we have any existing search utilities
4. Look at how other features handle user input

Don't write any code yet.
```

**Phase 2: Plan**
```
Based on what you learned, think hard about implementing product search:

Requirements:
- Search by product name and description
- Filter by category and price range
- Debounced input (300ms)
- Show loading and empty states

Create a detailed implementation plan with file changes, order of work,
and testing approach.
```

**Phase 3: Code**
```
Implement the search feature following our plan. Start with the search
API endpoint (step 1).

After each step, run the relevant tests before proceeding.
```

**Phase 4: Commit**
```
Search feature is complete. Create a commit, update CHANGELOG, and
prepare a PR description summarizing:
- What was added
- How to test it
- Any configuration needed
```

---

## Complete Annotated Workflow Transcript

Here's a full, realistic transcript showing how the 4-phase workflow unfolds in practice. Annotations explain what's happening and why.

### Task: Add Password Reset Functionality

```
┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 1: EXPLORE                                                            │
└─────────────────────────────────────────────────────────────────────────────┘

USER:
I need to add password reset functionality to our app. Users should be able
to request a reset email and set a new password.

Before we implement anything, I need you to explore the codebase:
1. Read the current auth system in src/auth/
2. Understand how we send emails (if we have email capability)
3. Check what password validation rules exist
4. Look at the database schema for users

Don't write any code yet - just help me understand what we're working with.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
I'll explore each area. Let me start by reading the auth system.

[Reads src/auth/]

**Auth System Summary:**

The authentication is in `src/auth/` with these key files:
- `login.ts` - Handles POST /api/auth/login, validates email/password
- `register.ts` - Handles POST /api/auth/register, creates users
- `middleware.ts` - JWT validation, attaches user to request
- `tokens.ts` - JWT generation and validation utilities

Key observations:
- Passwords hashed with bcrypt (cost factor 12)
- JWT tokens expire in 7 days
- No existing password reset functionality

[Reads src/lib/email/]

**Email System:**

Found email configuration in `src/lib/email/`:
- Using Resend for email delivery
- `sendEmail()` function in `client.ts`
- Templates in `templates/` directory (currently: welcome.tsx, invoice.tsx)
- Environment variable: RESEND_API_KEY

[Reads prisma/schema.prisma]

**User Schema:**

```prisma
model User {
  id           String   @id @default(uuid())
  email        String   @unique
  passwordHash String
  name         String
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}
```

No password reset token storage currently exists.

[Reads src/auth/register.ts for validation]

**Password Validation:**

From registration:
- Minimum 8 characters
- At least one number required
- Zod schema in src/lib/validation.ts

**Summary of Exploration:**

1. ✅ We have an auth system with JWT
2. ✅ Email capability exists (Resend)
3. ✅ Password validation rules defined
4. ⚠️ No password reset tokens in database schema
5. ⚠️ No password reset email template

We'll need to add a reset token model and email template.

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: The explore phase established:
- What exists (auth, email infrastructure)
- What's missing (reset tokens, template)
- Constraints (password rules, JWT expiry patterns)
No code was written. Claude now has context for planning.]

USER:
Good exploration. I see we need schema changes. Before we plan, one question:
should reset tokens be stored in the database or can we use signed JWTs?

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
Good question. Let me think through the tradeoffs:

**Option A: Database tokens**
- Can be invalidated immediately (delete from DB)
- Can enforce "single use" by deleting after use
- Requires DB query on every validation
- Need to handle cleanup of expired tokens

**Option B: Signed JWTs**
- No database required
- Can't be invalidated before expiry
- Slightly faster validation (no DB query)
- If JWT leaked, valid until expiry

**Recommendation:** Database tokens are better for password reset because:
1. Security-critical operation warrants ability to invalidate
2. "Single use" is expected behavior
3. If user requests multiple resets, we can invalidate old ones
4. Matches industry standard practice

Should I proceed with database tokens in the plan?

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: Clarifying question before planning prevents rework.
The decision (DB tokens) shapes the entire implementation.]

USER:
Yes, use database tokens. Now create a detailed implementation plan.
Think hard about the security implications and edge cases.

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 2: PLAN                                                               │
└─────────────────────────────────────────────────────────────────────────────┘

CLAUDE:
[Extended thinking engaged]

# Password Reset Implementation Plan

## Overview
Add secure password reset flow: request email → click link → set new password.

## Security Considerations
- Token must be cryptographically random (not sequential/guessable)
- Short expiry (1 hour) limits exposure window
- Single use - invalidate after successful reset
- Rate limit requests to prevent email bombing
- Don't reveal if email exists (timing attack prevention)

## Implementation Steps

### Step 1: Database Schema (15 min)
**Files:** prisma/schema.prisma

Add PasswordResetToken model:
```prisma
model PasswordResetToken {
  id        String   @id @default(uuid())
  token     String   @unique
  userId    String
  user      User     @relation(fields: [userId], references: [id])
  expiresAt DateTime
  usedAt    DateTime?
  createdAt DateTime @default(now())
}
```

**Verify:** Run `prisma db push`, no errors

### Step 2: Token Generation Utility (20 min)
**Files:** src/auth/reset-tokens.ts (new)

Create functions:
- `generateResetToken()` - Returns crypto-random 32-byte hex string
- `createPasswordResetToken(userId)` - Creates DB record, returns token
- `validateResetToken(token)` - Returns user if valid, null if not
- `invalidateResetToken(token)` - Marks as used

**Verify:** Unit tests pass

### Step 3: Request Reset Endpoint (30 min)
**Files:** src/app/api/auth/reset-password/request/route.ts (new)

POST /api/auth/reset-password/request
- Accept: { email }
- Find user by email
- If found: create token, send email
- If not found: do nothing (don't reveal)
- Always return: { message: "If email exists, reset link sent" }
- Rate limit: 3 requests per email per hour

**Verify:** API test, rate limit test

### Step 4: Email Template (20 min)
**Files:** src/lib/email/templates/password-reset.tsx (new)

Create email with:
- Subject: "Reset your password"
- Body: Explanation, reset link, expiry warning
- Link: {BASE_URL}/auth/reset-password?token={token}

**Verify:** Preview email in browser, link format correct

### Step 5: Reset Password Endpoint (30 min)
**Files:** src/app/api/auth/reset-password/confirm/route.ts (new)

POST /api/auth/reset-password/confirm
- Accept: { token, newPassword }
- Validate token (exists, not expired, not used)
- Validate new password (same rules as registration)
- Hash new password
- Update user's passwordHash
- Invalidate token
- Return: { success: true }

**Verify:** Full flow test

### Step 6: Frontend Pages (45 min)
**Files:**
- src/app/auth/forgot-password/page.tsx (new)
- src/app/auth/reset-password/page.tsx (new)

Forgot password page:
- Email input form
- Submit calls request endpoint
- Show success message (check your email)

Reset password page:
- Read token from URL
- New password + confirm inputs
- Submit calls confirm endpoint
- Redirect to login on success

**Verify:** Manual testing, error states

### Step 7: Integration Tests (30 min)
**Files:** src/__tests__/auth/password-reset.test.ts (new)

Test cases:
- Request reset for existing user → email sent
- Request reset for non-existing user → no email, same response
- Use valid token → password updated
- Use expired token → error
- Use already-used token → error
- Rate limiting → 429 after limit

**Verify:** All tests pass

## Edge Cases Handled
- User requests multiple resets: Each creates new token, all valid until used
- User changes email then requests reset: Token tied to user ID, works
- Attacker guesses tokens: 32 bytes = 2^256 possibilities, infeasible
- Server restart during reset: Stateless, DB has all needed info

## Risks and Mitigations
| Risk | Mitigation |
|------|------------|
| Email deliverability | Use established service (Resend) |
| Token brute force | Cryptographically random, rate limited |
| User enumeration | Same response for exists/doesn't exist |

## Estimated Total Time: 3-4 hours

Ready to proceed with Step 1?

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: Plan includes:
- Security considerations upfront
- Specific files to create/modify
- Verification step for each
- Edge cases and risks
This is the checkpoint where human reviews before coding begins.]

USER:
Plan looks good. Proceed with Step 1 (database schema).

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 3: CODE                                                               │
└─────────────────────────────────────────────────────────────────────────────┘

CLAUDE:
Implementing Step 1: Database Schema

[Edits prisma/schema.prisma - adds PasswordResetToken model]

Running verification:

> npx prisma db push

✓ Schema pushed successfully

> npx prisma generate

✓ Generated Prisma Client

Step 1 complete. Schema updated, Prisma client regenerated.

Ready for Step 2?

───────────────────────────────────────────────────────────────────────────────

USER:
Yes, proceed with Step 2.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
Implementing Step 2: Token Generation Utility

[Creates src/auth/reset-tokens.ts]

```typescript
import crypto from 'crypto';
import { db } from '@/lib/db';

export function generateResetToken(): string {
  return crypto.randomBytes(32).toString('hex');
}

export async function createPasswordResetToken(userId: string) {
  const token = generateResetToken();
  const expiresAt = new Date(Date.now() + 60 * 60 * 1000); // 1 hour

  await db.passwordResetToken.create({
    data: {
      token,
      userId,
      expiresAt,
    },
  });

  return token;
}

export async function validateResetToken(token: string) {
  const resetToken = await db.passwordResetToken.findUnique({
    where: { token },
    include: { user: true },
  });

  if (!resetToken) return null;
  if (resetToken.usedAt) return null;
  if (resetToken.expiresAt < new Date()) return null;

  return resetToken.user;
}

export async function invalidateResetToken(token: string) {
  await db.passwordResetToken.update({
    where: { token },
    data: { usedAt: new Date() },
  });
}
```

[Creates src/auth/__tests__/reset-tokens.test.ts]

Running verification:

> npm test src/auth/__tests__/reset-tokens.test.ts

 PASS  src/auth/__tests__/reset-tokens.test.ts
  ✓ generateResetToken returns 64 character hex string
  ✓ createPasswordResetToken creates token in database
  ✓ validateResetToken returns user for valid token
  ✓ validateResetToken returns null for expired token
  ✓ validateResetToken returns null for used token
  ✓ invalidateResetToken marks token as used

Test Suites: 1 passed, 1 total
Tests:       6 passed, 6 total

Step 2 complete. Ready for Step 3?

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: Each step:
1. Implements the planned code
2. Runs verification
3. Confirms success before moving on
This prevents accumulating bugs.]

USER:
Yes, continue through Step 5. I'll review the API endpoints together.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
[Implements Steps 3, 4, 5]

Step 3: Request endpoint created
Step 4: Email template created
Step 5: Confirm endpoint created

Running full verification:

> npm test src/app/api/auth/reset-password/

 PASS  src/app/api/auth/reset-password/__tests__/request.test.ts
 PASS  src/app/api/auth/reset-password/__tests__/confirm.test.ts

Manual test of email preview: ✓ Link format correct

All API endpoints implemented and tested. Ready for Step 6 (frontend)?

───────────────────────────────────────────────────────────────────────────────

[... Steps 6 and 7 proceed similarly ...]

───────────────────────────────────────────────────────────────────────────────

USER:
All steps complete and tests pass. Move to commit phase.

───────────────────────────────────────────────────────────────────────────────

┌─────────────────────────────────────────────────────────────────────────────┐
│ PHASE 4: COMMIT                                                             │
└─────────────────────────────────────────────────────────────────────────────┘

CLAUDE:
Creating commit and documentation:

[Creates commit]

```
feat(auth): add password reset functionality

- Add PasswordResetToken model for secure token storage
- Implement request and confirm endpoints with rate limiting
- Add password reset email template
- Create forgot-password and reset-password pages
- Include comprehensive test coverage

Security measures:
- Cryptographically random 32-byte tokens
- 1-hour expiration
- Single-use enforcement
- Rate limiting (3 requests/email/hour)
- User enumeration prevention

Closes #127
```

[Updates CHANGELOG.md]

```markdown
## [Unreleased]

### Added
- Password reset functionality
  - Users can request reset email from login page
  - Reset links expire after 1 hour
  - Comprehensive security measures implemented
```

[Creates PR description]

Ready for PR creation?

───────────────────────────────────────────────────────────────────────────────

USER:
Create the PR.

───────────────────────────────────────────────────────────────────────────────

CLAUDE:
> gh pr create --title "feat(auth): add password reset functionality" --body "..."

PR created: https://github.com/acme/dashboard/pull/128

## Summary
- Added password reset flow (request → email → reset page → confirm)
- Database token storage with expiration and single-use
- Rate limiting to prevent abuse
- Full test coverage

## Test Plan
- [ ] Request reset for valid email → email received
- [ ] Request reset for invalid email → same response (no user enumeration)
- [ ] Click reset link → lands on reset page with token
- [ ] Submit new password → password updated, redirected to login
- [ ] Try to reuse link → error message
- [ ] Wait 1+ hour, try link → error message (expired)

## Files Changed
- prisma/schema.prisma (token model)
- src/auth/reset-tokens.ts (new)
- src/app/api/auth/reset-password/request/route.ts (new)
- src/app/api/auth/reset-password/confirm/route.ts (new)
- src/lib/email/templates/password-reset.tsx (new)
- src/app/auth/forgot-password/page.tsx (new)
- src/app/auth/reset-password/page.tsx (new)
- Tests for all new code

───────────────────────────────────────────────────────────────────────────────

[ANNOTATION: The commit phase:
- Creates meaningful commit message
- Updates documentation
- Creates PR with test plan
- Ready for human code review]

```

**Key patterns demonstrated:**
1. **Explore phase asked questions** before writing code
2. **Plan phase was comprehensive** with security considerations
3. **Code phase verified each step** before proceeding
4. **Commit phase created audit trail** with PR and documentation

---

## Common Pitfalls

### Pitfall 1: Skipping Explore

**Symptom:** Generated code doesn't match existing patterns

**Solution:** Always explore first, even for "simple" changes

### Pitfall 2: Vague Plans

**Symptom:** Implementation stalls or goes in wrong direction

**Solution:** Make plans specific enough that another developer could follow them

### Pitfall 3: Never Clearing Context

**Symptom:** Claude gives increasingly confused or irrelevant responses

**Solution:** Use `/clear` between tasks; don't let sessions run indefinitely

### Pitfall 4: Implementing Without Verification

**Symptom:** Accumulating bugs discovered at the end

**Solution:** Verify after each implementation step, not just at the end

---

## Key Takeaways

1. **Follow the 4-phase workflow:** Explore → Plan → Code → Commit
2. **Think LoopScript:** Treat workflows as explicit specifications with checkpoints
3. **Never skip exploration** — Understanding before implementation prevents rework
4. **Use subagents effectively** — Understand their isolation and synthesis patterns
5. **Use "think" keywords** in the planning phase for better designs
6. **Define checkpoint evidence** — Know what proves each phase is complete
7. **Verify incrementally** — Check each step before proceeding
8. **Use CRPs when stuck** — Structure decision requests for efficient resolution
9. **Save plans** for complex features (GitHub issues or local files)
10. **Maintain context hygiene** — Use `/clear` between tasks, `/compact` to preserve progress

---

## Practical Exercise: Complete Workflow Practice

Choose a feature to implement (real or practice) and execute the full workflow:

### Part 1: Explore (~5 minutes)
Write a prompt that:
- Asks Claude to read relevant code
- Explicitly says "don't write code yet"
- Identifies what context you need

### Part 2: Plan (~10 minutes)
Write a prompt that:
- Uses "think" or "think hard"
- Requests a detailed implementation plan
- Includes verification steps

### Part 3: Code (~15 minutes)
Write a prompt that:
- References the plan
- Includes incremental verification
- Handles one step at a time

### Part 4: Commit (~5 minutes)
Write a prompt that:
- Creates a meaningful commit
- Updates relevant documentation
- Prepares for review

Reflect: How did this workflow compare to jumping straight to code?

---

## Next Step

[Step 07: Test-Driven Development with AI](07-test-driven-development-with-ai.md) — Learn Anthropic's "favorite workflow" for verifiable AI-assisted development.

---

## Sources

[^anthropic]: Anthropic. (2025). "Claude Code Best Practices." Official documentation on workflow patterns. https://docs.anthropic.com/en/docs/claude-code/best-practices

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. Sections on LoopScript and Consultation Request Packs (CRPs). https://arxiv.org/abs/2509.06216

- Stanford CS146S, Week 4: Coding Agent Patterns. Stanford University, 2025.
- Anthropic: Subagent Usage Documentation. https://docs.anthropic.com/en/docs/agents-and-tools

---

## Further Reading

1. **Anthropic: Building Effective Agents** — Research on designing reliable AI agent systems. https://www.anthropic.com/research/building-effective-agents

2. **Anthropic: Claude Code Best Practices** — Official guide to the 4-phase workflow. https://docs.anthropic.com/en/docs/claude-code/best-practices

3. **SASE Paper (arXiv:2509.06216v2)** — Academic framework including LoopScript and CRP specifications. https://arxiv.org/abs/2509.06216

4. **Stanford CS146S Course Materials** — Academic course on modern software development with AI. https://cs146s.stanford.edu/

5. **Anthropic: Agent Design Patterns** — Patterns for building reliable multi-step AI agents. https://docs.anthropic.com/en/docs/agents-and-tools/agent-loop
