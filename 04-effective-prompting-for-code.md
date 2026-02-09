# Step 04: Effective Prompting for Code

**Level:** Beginner-Intermediate
**Time:** ~30 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Apply specificity principles to get better code output
2. Use the "onboard a new teammate" mental model
3. Structure prompts using the Context / Task / Constraints / Output Format pattern
4. Use thinking keywords ("think", "think harder", "ultrathink") for complex problems

---

In [Step 03](03-first-ai-coding-session.md), you completed your first AI-assisted coding session and learned the basics of prompting. Now we'll dive deeper into prompt engineering techniques that consistently produce better results.

## The Specificity Principle

The single most important rule in prompting for code:

> **Vague prompts produce vague code.**

The more specific your instructions, the more likely you'll get useful output on the first try.

### Specificity Spectrum

| Vague (Lower Success Rate) | Specific (Higher Success Rate) |
|---------------------------|-------------------------------|
| "Add tests for foo.py" | "Write a test case for foo.py covering the edge case where the user is logged out. Avoid mocks." |
| "Make this faster" | "Optimize this function to reduce time complexity from O(n²) to O(n log n) using a sorting approach" |
| "Fix the bug" | "Fix the IndexError on line 45 that occurs when the input list is empty" |
| "Add error handling" | "Add try/except blocks to handle FileNotFoundError and return a helpful error message to the user" |
| "Create a login system" | "Create a login endpoint that accepts username/password JSON, validates against the users table, and returns a JWT token" |

### Prompt Quality Spectrum Diagram

```
+-------------------------------------------------------------------------+
|                      PROMPT QUALITY SPECTRUM                            |
+-------------------------------------------------------------------------+
|                                                                         |
|  VAGUE                                                      SPECIFIC    |
|  <----------------------------------------------------------->          |
|                                                                         |
|  "Add tests"                                                            |
|       |                                                                 |
|       +---> "Write unit tests for user service"                         |
|                  |                                                      |
|                  +---> "Write pytest tests for UserService              |
|                         covering login success, login failure,          |
|                         and session timeout edge cases.                 |
|                         Use fixtures for test user data.                |
|                         Follow patterns in tests/conftest.py."          |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  SUCCESS RATE:                                                          |
|                                                                         |
|  "Add tests"                                                            |
|  [██░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░]  ~20%             |
|                                                                         |
|  "Write unit tests for user service"                                    |
|  [██████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░]  ~40%             |
|                                                                         |
|  "Write pytest tests for UserService covering..."                       |
|  [██████████████████████████████████████████░░░░░░░░]  ~80%+            |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  WHAT SPECIFICITY ADDS:                                                 |
|                                                                         |
|  +------------------+------------------+------------------+              |
|  |    FRAMEWORK     |    TEST CASES    |    PATTERNS      |              |
|  |    (pytest)      |    (3 specific)  |    (conftest.py) |              |
|  +------------------+------------------+------------------+              |
|  |    COVERAGE      |    DATA SOURCE   |    CONSTRAINTS   |              |
|  |    (edge cases)  |    (fixtures)    |    (implicit)    |              |
|  +------------------+------------------+------------------+              |
|                                                                         |
|  Each additional detail REDUCES assumptions the AI must make            |
|                                                                         |
+-------------------------------------------------------------------------+
```

### Why Specificity Works

When you're vague, the AI must make assumptions about:
- Which approach to use (of many valid options)
- What edge cases to handle
- What style to follow
- What level of complexity is appropriate

Each assumption is a chance for misalignment between what you want and what you get.

---

## The "Onboard a New Teammate" Mental Model

Google's AI team recommends this mental model [^google]:

> **Treat the AI like a skilled engineer who just joined your team and knows nothing about your codebase.**

### What Would a New Teammate Need to Know?

1. **Project context:** What are we building? Why?
2. **Technical context:** What stack are we using? What patterns?
3. **Local conventions:** How do we name things? Structure files?
4. **Current state:** What exists? What are we changing?
5. **Success criteria:** How will we know it works?

### Example: Applying the Mental Model

**Without the mental model:**
> "Add a button to delete users"

**With the mental model:**
> "Our React admin dashboard needs a delete button for users. We use:
> - React 18 with TypeScript
> - Tailwind CSS for styling
> - Our existing Button component from src/components/Button.tsx
> - The deleteUser API endpoint (DELETE /api/users/:id)
>
> Add a red 'Delete' button to the user row in src/components/UserTable.tsx that:
> - Shows a confirmation dialog before deleting
> - Calls the deleteUser API
> - Shows a success toast on completion
> - Removes the row from the table without a full refresh"

The second prompt gives the AI everything a new teammate would need.

---

## The Prompt Structure Template

For consistent results, structure prompts using this template:

### 1. Context
What the AI needs to know about the situation

### 2. Task
What specific action to take

### 3. Constraints
What to avoid, limits, and requirements

### 4. Output Format (optional)
How you want the response structured

---

## The BriefingScript Framework

The SASE (Structured Agentic Software Engineering) research framework [^sase] introduces **BriefingScript**—a structured format for AI task specification that goes beyond basic prompt structure. While the prompt template above works for simple requests, BriefingScript provides a more complete framework for complex tasks.

### BriefingScript Components

A BriefingScript has four essential components:

### BriefingScript Structure Diagram

```
+-------------------------------------------------------------------------+
|                         BRIEFINGSCRIPT                                  |
+-------------------------------------------------------------------------+
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |                     1. SPECIFICATION                              |  |
|  |                                                                   |  |
|  |  WHAT needs to be built                                           |  |
|  |                                                                   |  |
|  |  Include:                                                         |  |
|  |    * Feature description in concrete terms                        |  |
|  |    * User-facing behavior                                         |  |
|  |    * API contracts or interfaces                                  |  |
|  |    * Data requirements                                            |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                              |                                          |
|                              v                                          |
|  +-------------------------------------------------------------------+  |
|  |                      2. CONTEXT                                   |  |
|  |                                                                   |  |
|  |  WHAT the agent needs to know to succeed                          |  |
|  |                                                                   |  |
|  |  Include:                                                         |  |
|  |    * Technology stack and versions                                |  |
|  |    * Existing patterns and conventions                            |  |
|  |    * Relevant code locations                                      |  |
|  |    * Dependencies and constraints                                 |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                              |                                          |
|                              v                                          |
|  +-------------------------------------------------------------------+  |
|  |                   3. SUCCESS CRITERIA                             |  |
|  |                                                                   |  |
|  |  HOW to know the task is complete                                 |  |
|  |                                                                   |  |
|  |  Include:                                                         |  |
|  |    * Testable conditions (pass/fail)                              |  |
|  |    * Acceptance criteria                                          |  |
|  |    * Edge cases that must work                                    |  |
|  |    * Performance requirements (if any)                            |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                              |                                          |
|                              v                                          |
|  +-------------------------------------------------------------------+  |
|  |                   4. VALIDATION PLAN                              |  |
|  |                                                                   |  |
|  |  HOW to verify the implementation is correct                      |  |
|  |                                                                   |  |
|  |  Include:                                                         |  |
|  |    * Tests to run (unit, integration, e2e)                        |  |
|  |    * Manual verification steps                                    |  |
|  |    * Security checks if applicable                                |  |
|  |    * Review criteria                                              |  |
|  |                                                                   |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  KEY INSIGHT: Traditional prompts often lack sections 3 & 4, leading    |
|  to ambiguous "done" definitions and unverified output.                 |
|                                                                         |
+-------------------------------------------------------------------------+
```

**1. Specification**
What needs to be built—the functional requirements:
- Feature description in concrete terms
- User-facing behavior
- API contracts or interfaces
- Data requirements

**2. Context**
What the agent needs to know to succeed:
- Technology stack and versions
- Existing patterns and conventions
- Relevant code locations
- Dependencies and constraints

**3. Success Criteria**
How to know the task is complete:
- Testable conditions
- Acceptance criteria
- Edge cases that must work
- Performance requirements (if any)

**4. Validation Plan**
How to verify the implementation is correct:
- Tests to run (unit, integration, etc.)
- Manual verification steps
- Security checks if applicable
- Review criteria

### Why BriefingScript Matters

Traditional prompts often lack explicit success criteria and validation plans. This leads to:
- Ambiguous "done" definitions
- Missed edge cases
- Code that appears to work but has subtle issues
- Difficulty verifying AI output

BriefingScript forces you to think through verification *before* generation, which:
- Improves first-try success rates
- Creates natural checkpoints for review
- Documents what "correct" means
- Aligns AI output with your actual needs

### BriefingScript Example: Complete API Endpoint

Here's a complete BriefingScript for a real-world feature request:

```
## Specification
Create a password reset endpoint that:
- Accepts a POST request with email address
- Generates a secure reset token
- Sends an email with a reset link
- Returns a generic success message (don't reveal if email exists)

## Context
- FastAPI application with Pydantic models
- PostgreSQL database via SQLAlchemy
- Email sending via SendGrid (client in src/services/email.py)
- User model in src/models/user.py
- Follow existing patterns in src/routes/auth.py

## Success Criteria
- Returns 200 with generic message for any email input
- Token is cryptographically random (32+ bytes)
- Token expires after 1 hour
- Token is hashed before storage (never store plaintext)
- Email is sent only if user exists (but response doesn't reveal this)
- Rate limited to 3 requests per email per hour

## Validation Plan
- Unit tests for token generation and validation
- Integration test with test database
- Manual test: request reset, check email, verify link works
- Security review: verify no information leakage about user existence
- Check rate limiting works correctly
```

### BriefingScript Example: Data Migration Task

```
## Specification
Create a data migration script to move user preferences from the legacy
`user_settings` JSON column to the new normalized `preferences` table.

Migration requirements:
- Process users in batches of 1000
- Handle malformed JSON gracefully (log and skip)
- Support both forward migration and rollback
- Be idempotent (safe to run multiple times)

## Context
Technology:
- Python 3.11, SQLAlchemy 2.0, PostgreSQL 15
- Alembic for migrations (put script in migrations/data/)
- Existing pattern in migrations/data/001_backfill_emails.py

Database Schema:
- users.user_settings: JSONB column with format {"theme": "dark", "notifications": {...}}
- preferences table: user_id, key, value (already created in schema migration)

## Success Criteria
- All valid user_settings are migrated to preferences table
- Malformed JSON entries are logged to migrations/logs/settings_migration_errors.log
- Running migration twice produces same result (idempotent)
- rollback() function restores original state
- Migration completes in < 5 minutes for 100k users

## Validation Plan
Testing:
- Unit test with mock data including edge cases
- Test on staging database backup (anonymized)
- Verify row counts match before/after

Verification Queries:
```sql
-- Count check
SELECT COUNT(*) FROM users WHERE user_settings IS NOT NULL;
SELECT COUNT(DISTINCT user_id) FROM preferences;

-- Spot check random users
SELECT u.id, u.user_settings, p.*
FROM users u
LEFT JOIN preferences p ON u.id = p.user_id
WHERE u.id IN (SELECT id FROM users ORDER BY RANDOM() LIMIT 5);
```
```

### BriefingScript Example: Refactoring Task

```
## Specification
Refactor the OrderService class to use the Repository pattern.

Current state:
- OrderService directly uses SQLAlchemy session
- Business logic and data access are mixed
- Hard to test without database

Target state:
- OrderService receives an OrderRepository via constructor injection
- OrderRepository handles all database operations
- OrderService contains only business logic

## Context
Files involved:
- src/services/order_service.py (refactor)
- src/repositories/order_repository.py (create new)
- tests/services/test_order_service.py (update)

Existing patterns:
- See src/repositories/user_repository.py for repository pattern
- See src/services/user_service.py for service with injected repository

Constraints:
- Maintain all existing public method signatures
- Don't change the Order model
- All existing tests must continue to pass

## Success Criteria
- OrderService has no direct SQLAlchemy imports
- OrderRepository handles: create, get_by_id, get_by_user, update_status, list_with_filters
- All 23 existing OrderService tests pass
- New OrderService tests use mock repository (no database)
- No changes to API layer (routes still work)

## Validation Plan
1. Run existing test suite: `pytest tests/services/test_order_service.py`
2. Run integration tests: `pytest tests/integration/test_orders.py`
3. Manual test: Create order via API, verify in database
4. Review: Confirm no SQLAlchemy in OrderService
```

### When to Use BriefingScript

| Situation | Basic Prompt | BriefingScript |
|-----------|--------------|----------------|
| Simple function | Sufficient | Overkill |
| Bug fix with clear error | Sufficient | Optional |
| New feature with security implications | May work | Recommended |
| Complex multi-file change | May miss requirements | Recommended |
| Task for agentic workflow | Often insufficient | Essential |
| Handoff to another developer/agent | Poor documentation | Excellent documentation |

For complex tasks, the time invested in writing a BriefingScript is repaid through fewer iterations and more reliable output.

---

### Full Example

```
## Context
I'm building a REST API in Python with FastAPI. The codebase uses SQLAlchemy
for database access and Pydantic for validation. We follow the repository
pattern with services in src/services/ and models in src/models/.

## Task
Create a new endpoint to update a user's email address. The endpoint should:
- Accept a PUT request to /users/{user_id}/email
- Validate the new email format
- Check that the email isn't already in use by another user
- Update the user's record
- Return the updated user object

## Constraints
- Use the existing UserRepository in src/repositories/user_repository.py
- Follow our existing error handling pattern (raise HTTPException with appropriate status codes)
- Add type hints to all functions
- Do not modify existing files unless necessary

## Output Format
Provide:
1. The new endpoint code
2. Any new Pydantic models needed
3. A brief explanation of the implementation
```

---

## Thinking Keywords: Extended Reasoning

Claude Code supports special keywords that trigger extended reasoning, giving the model more "thinking time" for complex problems.

### The Thinking Hierarchy

| Keyword | Thinking Budget | Best For |
|---------|-----------------|----------|
| "think" | Standard extended thinking | Most complex tasks |
| "think hard" | More extended thinking | Difficult architectural decisions |
| "think harder" | Even more thinking | Very complex problems |
| "ultrathink" | Maximum thinking budget | The hardest problems |

### Thinking Keywords Decision Tree

```
+-------------------------------------------------------------------------+
|                  WHEN TO USE THINKING KEYWORDS                          |
+-------------------------------------------------------------------------+
|                                                                         |
|                         Is the task complex?                            |
|                               |                                         |
|               +---------------+---------------+                         |
|               |                               |                         |
|              No                              Yes                        |
|               |                               |                         |
|               v                               v                         |
|     +-------------------+          Does it require                      |
|     | Standard prompt   |          tradeoff analysis?                   |
|     | (no keyword)      |                 |                             |
|     +-------------------+       +---------+---------+                   |
|                                 |                   |                   |
|                                No                  Yes                  |
|                                 |                   |                   |
|                                 v                   v                   |
|                          +------------+      Is it architectural?       |
|                          |  "think"   |      (system-wide impact)       |
|                          +------------+            |                    |
|                                            +-------+-------+            |
|                                            |               |            |
|                                           No              Yes           |
|                                            |               |            |
|                                            v               v            |
|                                     +-------------+   Is it critical?   |
|                                     | "think hard"|   (security/core)   |
|                                     +-------------+        |            |
|                                                      +-----+-----+      |
|                                                      |           |      |
|                                                     No          Yes     |
|                                                      |           |      |
|                                                      v           v      |
|                                               +--------------+ +-------------+
|                                               |"think harder"| | "ultrathink"|
|                                               +--------------+ +-------------+
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  EXAMPLES BY LEVEL:                                                     |
|                                                                         |
|  Standard (no keyword):                                                 |
|    - "Write a function to parse JSON"                                   |
|    - "Add input validation to this endpoint"                            |
|                                                                         |
|  "think":                                                               |
|    - "Plan how to implement this new feature"                           |
|    - "Debug why this test is failing intermittently"                    |
|                                                                         |
|  "think hard":                                                          |
|    - "Design a caching strategy for this API"                           |
|    - "Analyze the tradeoffs between these two approaches"               |
|                                                                         |
|  "think harder":                                                        |
|    - "Architect a multi-service event system"                           |
|    - "Design the data model for this complex domain"                    |
|                                                                         |
|  "ultrathink":                                                          |
|    - "Design a security-critical authentication flow"                   |
|    - "Architect a system that must handle 1M concurrent users"          |
|                                                                         |
+-------------------------------------------------------------------------+
```

### When to Use Thinking Keywords

**Use "think" for:**
- Planning a feature implementation
- Debugging a complex issue
- Refactoring decisions

**Use "think hard" or "think harder" for:**
- Architectural decisions with multiple tradeoffs
- Complex algorithm design
- Security-sensitive implementations

**Use "ultrathink" sparingly for:**
- The most challenging technical problems
- When other levels haven't produced good solutions

### Example: Using Thinking Keywords

```
Think about how to implement a caching layer for our API:
- We have ~100 endpoints with varying data freshness requirements
- Some data is user-specific, some is global
- We want to minimize database load while maintaining correctness
- We're using Redis but open to alternatives

Create a detailed plan before writing any code.
```

For a harder problem:

```
Think harder about the authentication flow for our multi-tenant SaaS:
- Users can belong to multiple organizations
- Some endpoints are org-specific, others are user-specific
- We need to support SSO, OAuth, and email/password
- Session management needs to track which org the user is currently acting as

Analyze the tradeoffs of different approaches before recommending a solution.
```

---

## Prompt Chaining: Multi-Step Workflows

Complex tasks often benefit from **prompt chaining**—breaking work into sequential prompts where each step builds on the previous one.

### Why Chain Prompts?

Single large prompts often fail because:
- The AI loses focus on complex multi-part requests
- Errors in early parts cascade through the whole response
- You can't validate intermediate steps
- Context becomes cluttered with partial work

Chaining prompts lets you:
- Validate each step before proceeding
- Course-correct early when something goes wrong
- Build context incrementally
- Apply different thinking levels to different phases

### The Chaining Pattern

**Step 1: Analysis**
> "Before implementing anything, analyze the current codebase:
> - What authentication pattern is used?
> - Where are the auth-related files?
> - What dependencies are already available?
>
> Don't write any code yet—just describe what you find."

**Step 2: Design**
> "Based on your analysis, propose a design for adding password reset:
> - What files need to be created or modified?
> - What's the data flow?
> - What are the key security considerations?
>
> Don't implement yet—let me review the design first."

**Step 3: Implementation**
> "The design looks good. Now implement the password reset feature following your proposed design. Start with the token generation service."

**Step 4: Testing**
> "The implementation is complete. Now write tests that cover:
> - Successful password reset flow
> - Invalid/expired tokens
> - Rate limiting
> - Security edge cases"

### When to Chain vs. Single Prompt

| Situation | Approach |
|-----------|----------|
| Simple, well-defined task | Single prompt |
| Unfamiliar codebase | Chain: analyze first |
| Multiple files/components | Chain: design before implement |
| Security-sensitive | Chain: review at each stage |
| Learning a new concept | Chain: explain before applying |
| Time-sensitive | Single prompt (if confident) |

### Maintaining State Across Chains

When chaining prompts in a conversation:
- Reference previous outputs explicitly: "Using the design from above..."
- Summarize key decisions if the conversation gets long
- Use `/compact` in Claude Code if context grows too large
- Consider starting a new conversation for distinct phases if context rot becomes an issue

### Code Example: Complete Prompt Chain for a Feature

Here's a real prompt chain building a file upload feature. Notice how each step validates before moving forward.

**Step 1: Understand the Requirements**

```
I need to add file upload functionality to our FastAPI app. Before writing
any code, help me understand the requirements by asking clarifying questions.

What I know:
- Users upload profile pictures
- Max file size: 5MB
- Allowed types: JPEG, PNG
- Need to store files somewhere

What I need help deciding:
- Storage approach (local vs cloud)
- File naming strategy
- Image processing needs
```

*AI asks questions, you answer them, establishing shared context.*

**Step 2: Analyze Existing Code**

```
Good, now let's look at the existing codebase before designing. Please examine:
1. Current user model in src/models/user.py
2. Any existing file handling in the codebase
3. The configuration pattern in src/config.py

Don't write implementation code yet - just summarize what exists and what
patterns we should follow.
```

*AI analyzes, you verify the analysis is accurate.*

**Step 3: Create the Design**

```
Based on our discussion and your analysis, please design the file upload
feature. Include:

1. New/modified files list
2. Data flow diagram (text description)
3. API endpoint specification
4. Storage approach decision with rationale
5. Security considerations

Present this as a design document I can review before implementation.
```

*AI produces design, you review and request changes if needed.*

**Step 4: Implement in Stages**

```
The design looks good with one change: use S3 instead of local storage since
we deploy to Kubernetes.

Now implement Step 1 from your design: the S3 client wrapper and configuration.
Include unit tests. Don't implement the other parts yet.
```

*AI implements, you review the S3 client code.*

```
The S3 client looks good. Now implement Step 2: the image processing service
that resizes and validates uploads. Include tests.
```

*Continue for each component...*

**Step 5: Integration and Testing**

```
All components are implemented. Now:
1. Create the API endpoint that ties them together
2. Write integration tests for the full upload flow
3. Add API documentation in docstrings
```

**Step 6: Review and Harden**

```
The feature is working. Before we're done, please:
1. Review for security issues (file type validation, path traversal, etc.)
2. Add appropriate error handling and logging
3. Verify no sensitive data (AWS keys) is logged
4. Check rate limiting is appropriate
```

### Why This Chain Works

| Step | Purpose | What Could Go Wrong Without It |
|------|---------|-------------------------------|
| Requirements | Shared understanding | AI makes wrong assumptions |
| Analysis | Understand existing patterns | New code clashes with codebase |
| Design | Plan before coding | Architectural issues found late |
| Staged Implementation | Review each piece | Errors compound in large changes |
| Integration | Verify components work together | Interface mismatches |
| Hardening | Security and production readiness | Vulnerabilities ship to production |

---

## Error Recovery Prompting

When AI-generated code fails, how you communicate the failure determines how well the AI can help.

### The Error Recovery Pattern

**Step 1: Report the Failure Clearly**

```
The implementation you provided has an error.

When I run: python test_auth.py

I get:
```
Traceback (most recent call last):
  File "test_auth.py", line 15, in test_password_reset
    token = auth_service.generate_reset_token(user)
  File "src/services/auth.py", line 42, in generate_reset_token
    return secrets.token_urlsafe(32).encode('utf-8')
AttributeError: 'str' object has no attribute 'encode'
```

The relevant code is:
[paste the specific function or file section]
```

**Step 2: State What You Need**

```
Please fix this error while:
- Not changing the function signature
- Maintaining compatibility with existing callers
- Keeping the token cryptographically secure
```

### What to Include in Error Reports

| Element | Why It Matters |
|---------|----------------|
| Exact error message | Tells AI the specific failure |
| Full traceback | Shows error location and call chain |
| How you triggered the error | Helps AI reproduce and test |
| Relevant code | Context for understanding the bug |
| What you expected | Clarifies the goal |
| Constraints on the fix | Prevents breaking changes |

### Common Error Recovery Scenarios

**Syntax Error**
> "The code has a syntax error on line 23. The error is: [exact error]. Please fix the syntax while preserving the intended logic."

**Import Error**
> "The code tries to import [module] but it doesn't exist. Are you hallucinating this API, or did you mean a different module? Please provide working code."

**Logic Error (wrong output)**
> "The function runs without errors but produces wrong results:
> - Input: [specific input]
> - Expected: [what you expected]
> - Actual: [what you got]
> Please fix the logic."

**Integration Error**
> "The code doesn't integrate correctly with the existing codebase:
> - It expects [X] but our code provides [Y]
> - Please adjust to match our existing pattern: [show pattern]"

### Avoiding Error Loops

Sometimes AI fixes create new errors, creating frustrating loops. To break out:

1. **Step back:** Ask the AI to explain its approach before fixing
2. **Simplify:** Ask for a simpler solution that avoids the problematic area
3. **Alternative approach:** "This approach isn't working. What's a completely different way to solve this?"
4. **Isolate:** Test the problematic part in isolation before integrating
5. **Human intervention:** Sometimes you need to fix part of it yourself and ask AI to complete the rest

---

## Security-Focused Prompting

Security vulnerabilities are common in AI-generated code. Proactively address this in your prompts.

### Security Prompt Additions

Add security requirements explicitly:

```
Implement this feature following these security requirements:
- Use parameterized queries for all database access (no string concatenation)
- Validate and sanitize all user input
- Do not use pickle for serialization (use JSON instead)
- Do not log sensitive data like passwords or tokens
- Follow OWASP Top 10 guidelines
```

### Security-Focused Prompt Template

```
## Task
[Your task description]

## Security Requirements
- Input validation: [specific requirements]
- Authentication: [requirements]
- Authorization: [requirements]
- Data handling: [requirements]
- Error messages: [what NOT to expose]

## What NOT to Do
- Do not use eval() or exec()
- Do not concatenate user input into SQL queries
- Do not store passwords in plain text
- Do not expose stack traces to users
```

### Example: Secure Password Reset

**Basic prompt (security issues likely):**
> "Create a password reset feature"

**Security-focused prompt:**
> "Create a password reset feature with these security requirements:
> - Generate cryptographically secure tokens (use secrets module)
> - Tokens expire after 1 hour
> - Tokens can only be used once
> - Don't reveal whether an email exists in the system (use generic success message)
> - Hash the token before storing in database
> - Log password reset attempts (without the token value)
> - Rate limit reset requests per email to 3 per hour"

---

## Prompting Anti-Patterns

Avoid these common mistakes:

### Anti-Pattern Gallery

```
+-------------------------------------------------------------------------+
|                     PROMPTING ANTI-PATTERNS                             |
+-------------------------------------------------------------------------+
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [X] ANTI-PATTERN 1: WALL OF TEXT                                 |  |
|  |                                                                   |  |
|  |  "Build a complete e-commerce platform with user authentication,  |  |
|  |   product catalog, shopping cart, checkout, payment processing,   |  |
|  |   order tracking, admin dashboard, analytics, email notifications,|  |
|  |   inventory management, reviews, wishlists, and mobile support..."|  |
|  |                                                                   |  |
|  |  PROBLEM: Too much at once, AI loses focus                        |  |
|  |                                                                   |  |
|  |  [OK] SOLUTION: Break it down                                     |  |
|  |  "Let's build the user auth module first.                         |  |
|  |   Create a login endpoint that..."                                |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [X] ANTI-PATTERN 2: IMPLICIT EXPECTATIONS                        |  |
|  |                                                                   |  |
|  |  "Create a user service"                                          |  |
|  |   (Expects repository pattern, but didn't say so)                 |  |
|  |   (Expects specific error handling, but didn't mention it)        |  |
|  |   (Expects tests, but didn't ask)                                 |  |
|  |                                                                   |  |
|  |  PROBLEM: AI makes different assumptions than you                 |  |
|  |                                                                   |  |
|  |  [OK] SOLUTION: State expectations explicitly                     |  |
|  |  "Create a UserService class following the repository pattern     |  |
|  |   in src/services/. Include error handling for not-found cases    |  |
|  |   and add unit tests using our conftest.py fixtures."             |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [X] ANTI-PATTERN 3: CONTRADICTORY INSTRUCTIONS                   |  |
|  |                                                                   |  |
|  |  "Make this fast but use no additional memory"                    |  |
|  |  "Keep it simple but handle all edge cases"                       |  |
|  |  "Be thorough but keep it short"                                  |  |
|  |                                                                   |  |
|  |  PROBLEM: AI cannot satisfy conflicting requirements              |  |
|  |                                                                   |  |
|  |  [OK] SOLUTION: Acknowledge tradeoffs, prioritize                 |  |
|  |  "Optimize for speed. If there's a speed/memory tradeoff,         |  |
|  |   prefer speed up to 50MB additional memory usage."               |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [X] ANTI-PATTERN 4: SCOPE CREEP                                  |  |
|  |                                                                   |  |
|  |  "Write a function to validate emails, also it should check if    |  |
|  |   the domain exists and maybe we could also integrate with the    |  |
|  |   newsletter service and also add logging and..."                 |  |
|  |                                                                   |  |
|  |  PROBLEM: Requirements drift, focus lost                          |  |
|  |                                                                   |  |
|  |  [OK] SOLUTION: One clear task per prompt                         |  |
|  |  Prompt 1: "Validate email format"                                |  |
|  |  Prompt 2: "Add domain MX record verification"                    |  |
|  |  Prompt 3: "Integrate with newsletter service"                    |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
|  +-------------------------------------------------------------------+  |
|  |  [X] ANTI-PATTERN 5: NO EXAMPLES                                  |  |
|  |                                                                   |  |
|  |  "Format the output appropriately"                                |  |
|  |  "Make it user-friendly"                                          |  |
|  |  "Handle errors gracefully"                                       |  |
|  |                                                                   |  |
|  |  PROBLEM: "Appropriate" means different things to different people|  |
|  |                                                                   |  |
|  |  [OK] SOLUTION: Include concrete input/output examples            |  |
|  |  "Format currency: format_currency(1234.5) -> '$1,234.50'"        |  |
|  |  "Error format: {'error': 'User not found', 'code': 'NOT_FOUND'}" |  |
|  +-------------------------------------------------------------------+  |
|                                                                         |
+-------------------------------------------------------------------------+
```

### Anti-Pattern 1: The Wall of Text

**Problem:** Massive prompts with too much information

**Solution:** Break into focused, incremental prompts

### Anti-Pattern 2: Implicit Expectations

**Problem:** Assuming the AI knows your preferences

```
"Create a user service"
(But you expect a specific architecture that you didn't mention)
```

**Solution:** State expectations explicitly

### Anti-Pattern 3: Contradictory Instructions

**Problem:** Constraints that conflict

```
"Make this fast but use no additional memory"
"Keep it simple but handle all edge cases"
```

**Solution:** Acknowledge tradeoffs; prioritize requirements

### Anti-Pattern 4: Scope Creep in Prompts

**Problem:** Adding more requirements mid-prompt

```
"Write a function to validate emails, also it should check if the domain
exists and maybe we could also integrate with the newsletter service and
also add logging and..."
```

**Solution:** One clear task per prompt; iterate for additions

### Anti-Pattern 5: No Examples

**Problem:** Abstract requirements without concrete examples

**Solution:** Include input/output examples

```
The function should format currency values:
- format_currency(1234.5) → "$1,234.50"
- format_currency(1000000) → "$1,000,000.00"
- format_currency(0.5) → "$0.50"
```

---

## Prompt Templates for Common Tasks

### Bug Fix Template

```
## Bug Description
[What's happening vs. what should happen]

## Steps to Reproduce
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Current Behavior
[What currently happens]

## Expected Behavior
[What should happen]

## Relevant Code
[File path and specific functions]

## Error Messages (if any)
[Exact error text]
```

### Feature Implementation Template

```
## Feature
[One-line description]

## Context
[What exists, why we need this]

## Requirements
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

## Acceptance Criteria
- [ ] [Criterion 1]
- [ ] [Criterion 2]
- [ ] [Criterion 3]

## Constraints
- [Constraint 1]
- [Constraint 2]
```

### Code Review Request Template

```
## Code to Review
[File path or code block]

## What to Look For
- [ ] Logic errors
- [ ] Security vulnerabilities
- [ ] Performance issues
- [ ] Code style
- [ ] Edge cases

## Context
[Why this code was written, what it's trying to achieve]
```

---

## Extended Prompt Library

Here are 10+ additional prompt templates for common development tasks. Copy and adapt these to your needs.

### 1. API Endpoint Template

```
## Task
Create a REST API endpoint for [resource/action].

## Endpoint Specification
- Method: [GET/POST/PUT/DELETE]
- Path: [/api/v1/...]
- Request body: [JSON schema or "none"]
- Response: [JSON schema]
- Status codes: [200, 400, 401, 404, etc. with meanings]

## Context
- Framework: [FastAPI/Express/etc.]
- Authentication: [JWT/API key/none]
- Existing patterns: See [file path]

## Requirements
- [Requirement 1]
- [Requirement 2]
- Input validation: [specific rules]
- Error handling: [specific behavior]

## Example Request/Response
Request:
```json
{}
```

Response:
```json
{}
```
```

### 2. Database Query Template

```
## Task
Write a [SQL/ORM] query to [description].

## Tables Involved
- [table1]: [relevant columns]
- [table2]: [relevant columns]

## Requirements
- Filter by: [conditions]
- Sort by: [field, direction]
- Pagination: [yes/no, method]
- Performance: [index hints, expected row counts]

## Expected Output
[Describe columns and sample row]

## Constraints
- Must use [ORM/raw SQL]
- Avoid [N+1 queries/full table scans/etc.]
- Handle [null values/empty results] by [behavior]
```

### 3. Refactoring Template

```
## Task
Refactor [file/function/class] to [improvement goal].

## Current Code
[File path or code block]

## Problems with Current Code
- [Problem 1]
- [Problem 2]

## Desired Improvements
- [Improvement 1]
- [Improvement 2]

## Constraints
- Maintain public API (no signature changes)
- Keep all existing tests passing
- Follow patterns in [reference file]

## Out of Scope
- [What NOT to change]
```

### 4. Bug Investigation Template

```
## Issue
[Brief description of the bug]

## Reproduction Steps
1. [Step 1]
2. [Step 2]
3. [Step 3]

## Expected Behavior
[What should happen]

## Actual Behavior
[What actually happens]

## Error Messages
```
[Exact error text/stack trace]
```

## Environment
- OS: [Windows/Mac/Linux]
- [Language] version: [version]
- Relevant packages: [versions]

## What I've Tried
- [Attempt 1 and result]
- [Attempt 2 and result]

## Relevant Code
[File path and specific functions]
```

### 5. Performance Optimization Template

```
## Task
Optimize [function/query/endpoint] for better performance.

## Current Performance
- Execution time: [X ms/s]
- Memory usage: [X MB]
- Throughput: [X requests/second]

## Target Performance
- Execution time: [target]
- Memory usage: [target]
- Throughput: [target]

## Current Implementation
[Code or file path]

## Constraints
- Cannot change: [public API/data format/etc.]
- Must maintain: [correctness/thread safety/etc.]
- Available resources: [caching/more memory/etc.]

## Profiling Data (if available)
[Hotspots, bottlenecks identified]
```

### 6. Test Coverage Template

```
## Task
Add tests to improve coverage for [module/feature].

## Current Coverage
- [File]: [X%] coverage
- Missing: [specific untested paths]

## Files to Test
[File paths]

## Testing Framework
[pytest/jest/etc.] with [fixtures/mocks available]

## Test Categories Needed
- [ ] Happy path tests
- [ ] Error handling tests
- [ ] Edge cases: [specific edges]
- [ ] Integration tests

## Existing Test Patterns
See [test file path] for conventions

## Mocking Requirements
- [External service] should be mocked
- [Database] should use [fixtures/test db]
```

### 7. Documentation Generation Template

```
## Task
Generate documentation for [module/API/function].

## Documentation Type
[API docs/README/docstrings/user guide]

## Target Audience
[Developers using the API/End users/New team members]

## Code to Document
[File paths or code blocks]

## Documentation Should Include
- [ ] Purpose/overview
- [ ] Installation/setup (if applicable)
- [ ] Usage examples
- [ ] Parameter descriptions
- [ ] Return value descriptions
- [ ] Error/exception documentation
- [ ] [Other requirements]

## Format
[Markdown/RST/JSDoc/etc.]

## Examples to Include
[Specific use cases to demonstrate]
```

### 8. Security Audit Template

```
## Task
Review this code for security vulnerabilities.

## Code to Audit
[File paths or code blocks]

## Context
- Application type: [web app/API/CLI/etc.]
- Handles: [user input/files/payments/etc.]
- Exposed to: [public internet/internal network/etc.]

## Specific Concerns
- [ ] Input validation
- [ ] Authentication/authorization
- [ ] Data exposure
- [ ] Injection vulnerabilities
- [ ] [Other specific concerns]

## Standards to Check Against
[OWASP Top 10/CWE list/company standards]

## Output Format
For each issue found, provide:
1. Location (file:line)
2. Vulnerability type
3. Severity (Critical/High/Medium/Low)
4. Recommended fix
5. Example exploit (if safe to describe)
```

### 9. Code Explanation Template

```
## Task
Explain this code so I can understand and maintain it.

## Code to Explain
[Code block or file path]

## My Background
[Junior developer/Senior but new to this stack/etc.]

## Specific Questions
1. [What does X do?]
2. [Why is Y implemented this way?]
3. [What would happen if Z?]

## Explanation Should Cover
- [ ] Overall purpose
- [ ] Key algorithms/patterns used
- [ ] Non-obvious design decisions
- [ ] Potential gotchas/edge cases
- [ ] How to modify safely

## Preferred Explanation Style
[Step-by-step walkthrough/High-level overview/Annotated code]
```

### 10. Migration Template

```
## Task
Create a migration from [old state] to [new state].

## Migration Type
[Database schema/API version/Library upgrade/etc.]

## Current State
[Description or schema]

## Target State
[Description or schema]

## Data Transformation Rules
- [Field A] maps to [Field B] via [transformation]
- [Field C] is [deprecated/split/merged]

## Requirements
- [ ] Backwards compatible during rollout
- [ ] Rollback capability
- [ ] Zero downtime
- [ ] Data validation post-migration

## Constraints
- Must complete in [time limit]
- Cannot lock [tables/resources]
- Must handle [edge cases]

## Testing Plan
- [ ] Test with production data sample
- [ ] Verify rollback works
- [ ] Performance test with full dataset
```

### 11. Error Handling Template

```
## Task
Add comprehensive error handling to [module/function].

## Code to Enhance
[File path or code block]

## Error Scenarios to Handle
- [Scenario 1]: [desired behavior]
- [Scenario 2]: [desired behavior]
- [Network failures]: [retry/fail fast/etc.]
- [Invalid input]: [validation errors]

## Error Response Format
```json
{
    "error": {
        "code": "ERROR_CODE",
        "message": "Human readable message",
        "details": {}
    }
}
```

## Logging Requirements
- Log level: [ERROR/WARN/INFO for different cases]
- Include: [request ID/user ID/etc.]
- Exclude: [PII/secrets/etc.]

## Recovery Behavior
- [Scenario]: [retry X times/fallback to Y/etc.]
```

### 12. Integration Template

```
## Task
Integrate [external service/library] into [our application].

## Service to Integrate
- Name: [Service name]
- Documentation: [URL]
- SDK/API: [library name or REST]

## Integration Points
- [Where in our code this will be used]
- [What data flows in/out]

## Requirements
- [ ] Authentication: [API key/OAuth/etc.]
- [ ] Error handling: [retry logic/fallbacks]
- [ ] Rate limiting: [respect limits]
- [ ] Testing: [mock service/sandbox]

## Existing Patterns
See [file path] for how we integrate other services

## Configuration Needed
- [ ] Environment variables
- [ ] Feature flags
- [ ] Secrets management

## Acceptance Criteria
- [Specific functionality that must work]
```

---

## Key Takeaways

1. **Be specific** — Vague prompts produce vague code
2. **Use the "new teammate" model** — Provide all context a skilled engineer would need
3. **Structure prompts** — Context / Task / Constraints / Output Format
4. **Use BriefingScript for complex tasks** — Specification, Context, Success Criteria, Validation Plan
5. **Use thinking keywords** — "think", "think hard", "think harder", "ultrathink" for complex problems
6. **Chain prompts for multi-step work** — Analyze → Design → Implement → Test
7. **Recover from errors systematically** — Include exact errors, relevant code, and constraints on fixes
8. **Add security requirements explicitly** — Don't assume secure defaults
9. **Include examples** — Input/output pairs clarify expectations
10. **Avoid anti-patterns** — No walls of text, no implicit expectations, no scope creep

---

## Practical Exercise: Prompt Refinement

Take this poor prompt and improve it:

**Original:**
> "Add validation to the form"

### Step 1: Apply the new teammate model
What would a new developer need to know?

### Step 2: Structure with the template
Add Context, Task, Constraints, Output Format

### Step 3: Add specificity
What fields? What validation rules? What error messages?

### Step 4: Add security considerations
What could go wrong? What should we prevent?

### Your Improved Prompt:

```
## Context
[Fill in: What tech stack? What form? Where in the codebase?]

## Task
[Fill in: Specific fields and validation rules]

## Constraints
[Fill in: Error handling, UX requirements, security]

## Output Format
[Fill in: What should the AI provide?]
```

Compare your improved prompt to the original. How much more likely is it to produce useful code on the first try?

---

## Next Step

[Step 05: Context Engineering and Documentation](05-context-engineering-and-documentation.md) — Learn to create persistent context files that make every prompt more effective.

---

## Sources

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. https://arxiv.org/abs/2509.06216

[^google]: Google Cloud. (2024). "Five Best Practices for AI Coding Assistants." Google Cloud Blog. https://cloud.google.com/blog/products/ai-machine-learning/best-practices-for-ai-coding-assistants

- Anthropic: "Claude Code Best Practices." https://docs.anthropic.com/en/docs/claude-code/best-practices
- Anthropic: "Prompting Best Practices for Claude." https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering
- Stanford CS146S, Week 1: Effective Prompting. Stanford University, 2025.

---

## Further Reading

1. **Anthropic: Prompt Engineering Guide** — Comprehensive guide to crafting effective prompts for Claude. https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering

2. **SASE Paper (arXiv:2509.06216v2): BriefingScript Framework** — Academic specification of structured task formats for AI agents. https://arxiv.org/abs/2509.06216

3. **Google: Prompting Strategies** — Google's research on effective prompting techniques. https://ai.google.dev/gemini-api/docs/prompting-strategies

4. **OpenAI: Prompt Engineering Guide** — Best practices for prompting GPT models, applicable to all LLMs. https://platform.openai.com/docs/guides/prompt-engineering

5. **Learn Prompting** — Community-driven resource for prompt engineering techniques. https://learnprompting.org/
