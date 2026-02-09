# Step 10: Code Review and Quality Assurance

**Level:** Advanced
**Time:** ~35 minutes

In [Step 09](09-Security-in-AI-Assisted-Development.md), you learned about security vulnerabilities in AI-generated code. This step builds on that foundation by introducing systematic review frameworks that ensure quality, security, and correctness at scale.

## Learning Objectives

By the end of this step, you will be able to:

1. Establish AI code review standards
2. Understand the MRP (Merge-Readiness Pack) concept
3. Use multi-agent verification patterns
4. Build evaluation frameworks for AI-generated code

---

## The Review Challenge

When agents can produce code faster than humans can review it, a bottleneck emerges:

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  Agent Output:     ████████████████████████ (High velocity) │
│                                                             │
│  Human Review:     ████░░░░░░░░░░░░░░░░░░░░ (Limited)       │
│                                                             │
│  Result: Backlog, or unreviewed code reaches production     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

**The solution:** Transform review from "read every line" to "verify evidence of correctness."

---

## The MRP Concept

### What Is an MRP?

A **Merge-Readiness Pack (MRP)** is an evidence bundle that argues a change is ready to merge [^1]. Instead of reviewing raw diffs, reviewers evaluate structured evidence.

[^1]: Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025). The MRP concept is a core contribution of the SASE framework.

### MRP Structure

An MRP contains five components:

| Component | What It Proves |
|-----------|---------------|
| **Functional Completeness** | Code implements all requirements |
| **Verification Quality** | Tests adequately cover the change |
| **Code Hygiene** | Code follows standards and patterns |
| **Rationale** | Why this approach was chosen |
| **Auditability** | Change can be traced and reproduced |

### MRP Template

```markdown
# Merge-Readiness Pack

## 1. Functional Completeness

### Requirements Addressed
- [x] User can reset password via email
- [x] Token expires after 1 hour
- [x] Token is single-use
- [x] Rate limiting prevents abuse

### Evidence
- Screenshot of password reset flow: [link]
- Test output showing all requirements pass: [link]

## 2. Verification Quality

### Test Coverage
- Unit tests: 15 new, 0 modified, 100% pass
- Integration tests: 3 new, 100% pass
- Edge cases tested:
  - Expired token: [test_expired_token]
  - Invalid token: [test_invalid_token]
  - Rate limit exceeded: [test_rate_limit]

### Manual Verification
- [ ] Tested happy path manually
- [ ] Tested error states manually
- [ ] Verified email delivery

## 3. Code Hygiene

### Static Analysis
- ESLint: 0 errors, 0 warnings
- TypeScript: No type errors
- Semgrep security scan: 0 findings

### Patterns Followed
- Used existing AuthService pattern
- Followed project error handling conventions
- Added appropriate logging

## 4. Rationale

### Approach Chosen
We used JWT-based tokens stored hashed in the database rather than
session-based tokens because:
1. Stateless verification is faster
2. Consistent with existing auth patterns
3. Enables cross-service token validation

### Alternatives Considered
- Session-based: Rejected due to database lookup per verification
- Magic links: Rejected due to security concerns with URL exposure

## 5. Auditability

### Changes Summary
- 3 files modified
- 2 files created
- 127 lines added, 12 removed

### Commit History
1. feat: add password reset endpoint
2. feat: add email service for reset tokens
3. test: add password reset tests
4. docs: update API documentation

### Reproducibility
- All tests pass on fresh checkout
- Environment variables documented
- No external dependencies added
```

### Generating MRPs with AI

```
The password reset feature is complete. Generate a Merge-Readiness Pack:

1. Functional Completeness:
   - List each requirement from the spec
   - Link to test or evidence proving it works

2. Verification Quality:
   - Run all tests and include output
   - List edge cases covered

3. Code Hygiene:
   - Run linters and include results
   - Note which patterns were followed

4. Rationale:
   - Explain the approach chosen
   - Note alternatives that were considered and rejected

5. Auditability:
   - Summarize all changes
   - List commits in order
   - Confirm reproducibility
```

---

## Evidence-Based Review

### Evidence-Based Review Architecture

The following diagram illustrates how evidence-based review transforms the review process:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    EVIDENCE-BASED REVIEW ARCHITECTURE                        │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  TRADITIONAL REVIEW                    EVIDENCE-BASED REVIEW                 │
│  ─────────────────                    ────────────────────                  │
│                                                                              │
│  ┌─────────────────┐                  ┌─────────────────────┐               │
│  │  Code Changes   │                  │   Merge-Readiness   │               │
│  │    (Diff)       │                  │      Pack (MRP)     │               │
│  │                 │                  │                     │               │
│  │  500+ lines     │                  │  • Test Results     │               │
│  │  to read        │                  │  • Coverage %       │               │
│  └────────┬────────┘                  │  • Lint Results     │               │
│           │                           │  • Security Scan    │               │
│           ▼                           │  • Rationale        │               │
│  ┌─────────────────┐                  └──────────┬──────────┘               │
│  │  Human reads    │                             │                          │
│  │  every line     │                             ▼                          │
│  │                 │                  ┌─────────────────────┐               │
│  │  Time: O(n)     │                  │  Human evaluates    │               │
│  │  (n = lines)    │                  │  evidence bundle    │               │
│  └────────┬────────┘                  │                     │               │
│           │                           │  Time: O(1)         │               │
│           ▼                           │  (constant)         │               │
│  ┌─────────────────┐                  └──────────┬──────────┘               │
│  │  Quality varies │                             │                          │
│  │  by reviewer    │                             ▼                          │
│  │  fatigue        │                  ┌─────────────────────┐               │
│  └─────────────────┘                  │  Focus on judgment: │               │
│                                       │  • Approach correct?│               │
│                                       │  • Context aware?   │               │
│                                       │  • Future-proof?    │               │
│                                       └─────────────────────┘               │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  KEY INSIGHT: Machines verify mechanics, humans verify judgment             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Paradigm Shift

Traditional code review requires reviewers to read every line of a diff. This becomes a bottleneck when AI can generate code faster than humans can review it.

**Evidence-based review** transforms the process:

| Traditional Review | Evidence-Based Review |
|-------------------|----------------------|
| Read every line of diff | Evaluate evidence bundle |
| Time: O(lines changed) | Time: O(evidence items) |
| Thoroughness varies by reviewer | Consistent evaluation criteria |
| Human checks what AI could verify | Human focuses on judgment calls |

### What Reviewers Should Focus On

With an MRP in hand, human reviewers can focus on what humans do best:

**Human Judgment Required:**
- Does the rationale make sense?
- Is the approach appropriate for this context?
- Are there organizational or business considerations the AI missed?
- Does this align with long-term architecture goals?

**Automated Verification (Trust the MRP):**
- Tests pass (verified by CI)
- Coverage adequate (measured by tools)
- Linting clean (automated checks)
- Security scan pass (automated tools)

### The Five-Criteria Evidence Bundle

The SASE framework formalizes MRP evidence into five criteria:

1. **Functional Completeness** - Does the code do what it claims?
2. **Verification Quality** - Are tests comprehensive and meaningful?
3. **Code Hygiene** - Does it meet style and quality standards?
4. **Rationale** - Is the approach justified?
5. **Auditability** - Can changes be traced and reproduced?

A reviewer evaluates these criteria rather than reading every line of code.

---

## VCR (Version Controlled Resolution)

### VCR/CRP Workflow

The following diagram illustrates how CRPs and VCRs work together in AI-assisted development:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        VCR/CRP WORKFLOW                                      │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│    AGENT WORKING                          HUMAN COACH                        │
│    ─────────────                          ───────────                        │
│          │                                      │                            │
│          ▼                                      │                            │
│    ┌───────────────┐                           │                            │
│    │ Agent hits    │                           │                            │
│    │ decision point│                           │                            │
│    │ beyond scope  │                           │                            │
│    └───────┬───────┘                           │                            │
│            │                                    │                            │
│            ▼                                    │                            │
│    ┌───────────────┐    CRP (Consultation      │                            │
│    │ Generate CRP  │    Request Pack)          │                            │
│    │ with options  │ ──────────────────────────►│                            │
│    │ and analysis  │                           │                            │
│    └───────────────┘                           │                            │
│                                                 ▼                            │
│                                          ┌───────────────┐                  │
│                                          │ Human reviews │                  │
│                                          │ options and   │                  │
│                                          │ makes decision│                  │
│                                          └───────┬───────┘                  │
│                                                  │                           │
│                                                  ▼                           │
│            ┌───────────────┐    VCR (Version    │                            │
│            │ Agent receives│ ◄──Controlled      │                            │
│            │ decision +    │    Resolution)     │                            │
│            │ rationale     │                    │                            │
│            └───────┬───────┘                    │                            │
│                    │                             │                            │
│                    ▼                             │                            │
│            ┌───────────────┐                    │                            │
│            │ Agent resumes │                    │                            │
│            │ work with     │                    │                            │
│            │ guidance      │                    │                            │
│            └───────────────┘                    │                            │
│                                                 │                            │
├─────────────────────────────────────────────────────────────────────────────┤
│  VCRs are stored in version control, becoming institutional memory          │
│  Future agents (and humans) can reference past VCRs for similar decisions   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### What Is a VCR?

A **Version Controlled Resolution** is a documented record of a human decision made during AI-assisted development. When an agent encounters a decision that requires human judgment, the human's response is captured as a VCR.

### Why VCRs Matter

VCRs serve multiple purposes:

| Purpose | Benefit |
|---------|---------|
| **Institutional Memory** | Future agents can reference past decisions |
| **Consistency** | Similar situations are handled the same way |
| **Auditability** | Decisions can be traced and justified |
| **Learning** | Patterns emerge that can inform CLAUDE.md updates |

### VCR Structure

A VCR captures:

**1. Context** - What situation prompted the decision?

**2. Question** - What specific question was asked?

**3. Decision** - What did the human decide?

**4. Rationale** - Why was this decision made?

**5. Applicability** - When does this decision apply?

**6. Exceptions** - When does it NOT apply?

### VCR Example

```
VCR ID: VCR-2026-0042
Date: 2026-02-08
Related CRP: CRP-2026-0041

CONTEXT:
Agent implementing authentication needed guidance on token storage.

QUESTION:
Should JWT tokens be stored in localStorage or httpOnly cookies?

DECISION:
Use httpOnly cookies.

RATIONALE:
1. XSS attacks cannot access httpOnly cookies
2. Automatic inclusion in requests reduces complexity
3. Server-side invalidation is possible
4. Defense-in-depth: even if XSS protections fail, tokens are protected

APPLICABILITY:
- All authentication tokens
- All session management
- API tokens that need persistence

EXCEPTIONS:
- Third-party widget tokens that need JavaScript access
- Short-lived tokens for non-sensitive operations

FOLLOW-UP:
Updated CLAUDE.md with token storage preference.
```

### Storing and Indexing VCRs

VCRs should be:
- Stored in version control (e.g., `docs/decisions/` or `vcrs/`)
- Named consistently (e.g., `VCR-YYYY-NNNN.md`)
- Indexed in a summary document for easy discovery
- Tagged with relevant topics for searchability

Over time, VCRs become a knowledge base that new team members and future AI agents can reference.

### Complete VCR Template and Example

Use this template for creating VCRs in your project:

```markdown
<!-- vcrs/VCR-2026-0042.md -->

# Version Controlled Resolution

## Metadata
| Field | Value |
|-------|-------|
| **VCR ID** | VCR-2026-0042 |
| **Date** | 2026-02-08 |
| **Related CRP** | CRP-2026-0041 |
| **Author** | Sarah Chen (Tech Lead) |
| **Status** | Active |
| **Tags** | authentication, security, tokens, storage |

---

## Context

During implementation of the authentication module (Sprint 23), the coding agent
encountered a decision point about client-side token storage. This decision has
security implications and affects the architecture of future features.

**Project:** User Authentication System (v2.0)
**Sprint:** 23
**Feature:** JWT-based Authentication

---

## Question Asked

> "Should JWT tokens be stored in localStorage or httpOnly cookies for our
> web application's authentication system?"

This question was raised in CRP-2026-0041 with three options analyzed.

---

## Decision

**Use httpOnly cookies for JWT token storage.**

Implementation requirements:
1. Set `httpOnly: true` on all authentication cookies
2. Set `secure: true` to require HTTPS
3. Set `sameSite: 'strict'` to prevent CSRF
4. Implement CSRF token for state-changing requests as additional protection
5. Set appropriate expiration (access token: 15 min, refresh token: 7 days)

---

## Rationale

### Security Analysis

| Factor | localStorage | httpOnly Cookie | Winner |
|--------|-------------|-----------------|--------|
| XSS Protection | Vulnerable | Protected | Cookie |
| CSRF Protection | N/A | Requires mitigation | localStorage |
| Developer Simplicity | Simpler | More setup | localStorage |
| Automatic Transmission | Manual | Automatic | Cookie |
| Server Invalidation | Difficult | Possible | Cookie |

### Key Considerations

1. **XSS is harder to fully prevent than CSRF**
   - XSS can happen through any user input, third-party scripts, or npm dependencies
   - CSRF has well-established mitigations (tokens, SameSite cookies)
   - One successful XSS attack can steal all localStorage tokens

2. **Defense in Depth**
   - Even with strong XSS protections, assuming they could fail is prudent
   - httpOnly provides an additional layer when XSS protections fail
   - This aligns with our security principle: "Never trust a single defense"

3. **Organizational Precedent**
   - Our main product already uses httpOnly cookies
   - Security team has approved patterns for this approach
   - Consistency reduces cognitive load and audit scope

4. **Future-Proofing**
   - Server-side invalidation enables forced logout
   - Easier to implement token rotation
   - Better support for multi-device session management

### Rejected Alternatives

**Option A: localStorage**
- Rejected because XSS vulnerability outweighs simplicity benefits
- Our app handles user-generated content, increasing XSS risk surface

**Option C: Memory + Refresh Token (most secure)**
- Rejected because complexity doesn't justify marginal security improvement
- Token loss on page refresh creates poor UX for our use case
- May reconsider for high-security features in future

---

## Applicability

This decision applies to:

### MUST Apply
- All authentication tokens (access and refresh)
- Session identifiers
- Any token granting access to user data
- API tokens stored in browser

### SHOULD Apply
- Third-party service tokens (when possible)
- Remember-me tokens
- Any persistent credentials

### Exceptions Allowed
- Short-lived tokens for non-sensitive operations (< 5 min, no PII access)
- Tokens that must be accessible from JavaScript for third-party widgets
- Tokens used only for public/anonymous features

---

## Implementation Notes

### Cookie Configuration

```typescript
// Correct implementation
res.cookie('access_token', token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'strict',
  maxAge: 15 * 60 * 1000, // 15 minutes
  path: '/',
});
```

### CSRF Protection

Since we're using cookies, add CSRF token to state-changing requests:

```typescript
// Include CSRF token in forms
<input type="hidden" name="_csrf" value="{{csrfToken}}" />

// Or in AJAX headers
fetch('/api/action', {
  headers: {
    'X-CSRF-Token': getCsrfToken()
  }
});
```

---

## Follow-up Actions

- [x] Updated CLAUDE.md with token storage preference
- [x] Added cookie utility functions to shared library
- [ ] Update security documentation (assigned: @security-team)
- [ ] Add to onboarding guide (assigned: @docs-team)
- [ ] Audit existing token storage (assigned: @security-team, due: 2026-02-20)

---

## Related Documents

- CRP-2026-0041: Original consultation request
- ADR-015: Authentication Architecture Decision Record
- SEC-007: Cookie Security Policy
- CLAUDE.md: Section "Authentication Preferences"

---

## Revision History

| Date | Change | Author |
|------|--------|--------|
| 2026-02-08 | Initial decision | Sarah Chen |
```

**VCR Index Template:**

Also maintain an index file for discoverability:

```markdown
<!-- vcrs/INDEX.md -->

# VCR Index

## By Topic

### Authentication & Security
- [VCR-2026-0042](VCR-2026-0042.md) - JWT Token Storage (httpOnly cookies)
- [VCR-2026-0038](VCR-2026-0038.md) - Password Hashing Algorithm (Argon2)
- [VCR-2026-0035](VCR-2026-0035.md) - Session Timeout Policy (24 hours)

### Database & Data
- [VCR-2026-0040](VCR-2026-0040.md) - Soft Delete vs Hard Delete
- [VCR-2026-0033](VCR-2026-0033.md) - UUID vs Auto-increment IDs

### API Design
- [VCR-2026-0041](VCR-2026-0041.md) - Pagination Strategy (cursor-based)
- [VCR-2026-0036](VCR-2026-0036.md) - Error Response Format

## Recent Decisions

| VCR | Date | Topic | Status |
|-----|------|-------|--------|
| VCR-2026-0042 | 2026-02-08 | Token Storage | Active |
| VCR-2026-0041 | 2026-02-07 | Pagination | Active |
| VCR-2026-0040 | 2026-02-05 | Soft Delete | Active |

## Superseded Decisions

| VCR | Superseded By | Reason |
|-----|---------------|--------|
| VCR-2025-0012 | VCR-2026-0038 | Bcrypt replaced by Argon2 |
```

---

## CRP (Consultation Request Pack)

### What Is a CRP?

A **Consultation Request Pack** is a structured request from an AI agent to a human when the agent needs guidance. Instead of vague questions, the agent provides context, options, and a recommendation.

### When to Create a CRP

Agents should create CRPs when:
- A decision requires human judgment or authority
- Multiple valid approaches exist with significant tradeoffs
- The decision has long-term or architectural implications
- Security, compliance, or policy considerations are involved
- The agent is uncertain and the stakes are non-trivial

### CRP Structure

**1. Context** - What is the agent working on?

**2. Current State** - What has been done, what's in progress?

**3. Specific Question** - What exactly needs to be decided?

**4. Options** - What alternatives has the agent identified?
   - For each option: pros, cons, tradeoffs

**5. Recommendation** - What does the agent suggest and why?

**6. Urgency** - How soon is a decision needed?

**7. Response Format** - How should the human respond?

### CRP Example

```
CRP ID: CRP-2026-0041
Date: 2026-02-08
Urgency: Medium (blocking implementation)

CONTEXT:
Implementing authentication module per the PRD.
Reached decision point requiring human judgment.

CURRENT STATE:
- Completed: User model, registration endpoint
- In Progress: Login/session management
- Blocked: Token storage decision

SPECIFIC QUESTION:
How should JWT tokens be stored on the client?

OPTIONS:

Option A: localStorage
  Pros: Simple, persists across sessions, easy JS access
  Cons: XSS vulnerable, manual request attachment

Option B: httpOnly Cookies
  Pros: XSS safe, automatic request inclusion, server invalidation
  Cons: CSRF protection needed, slightly more complex

Option C: Memory + Refresh Token
  Pros: Most secure for access token
  Cons: Most complex, token lost on refresh

RECOMMENDATION:
Option B (httpOnly cookies) - best security/simplicity balance.

DECISION NEEDED BY:
2026-02-09 EOD to maintain sprint timeline.

HOW TO RESPOND:
Create a VCR with your decision and rationale.
```

### CRP Workflow

1. Agent encounters decision point
2. Agent creates CRP with full context
3. Human reviews CRP and options
4. Human creates VCR with decision
5. Agent continues with the decided approach
6. VCR is stored for future reference

This workflow ensures humans remain in control of important decisions while agents continue working efficiently on implementation.

### Complete CRP Template and Example

Use this template when creating Consultation Request Packs:

```markdown
<!-- crps/CRP-2026-0041.md -->

# Consultation Request Pack

## Metadata
| Field | Value |
|-------|-------|
| **CRP ID** | CRP-2026-0041 |
| **Date Created** | 2026-02-08 |
| **Agent/Author** | Coding Agent (Claude) |
| **Project** | User Authentication System v2.0 |
| **Sprint** | 23 |
| **Urgency** | Medium (blocking implementation) |

---

## Context

I am implementing the authentication module according to the PRD (Product
Requirements Document) for Sprint 23. The module handles user login, session
management, and token refresh.

**Relevant PRD Sections:**
- Section 3.1: User Authentication Flow
- Section 3.4: Security Requirements
- Section 5.2: Performance Requirements (< 200ms login latency)

**Related Code:**
- `src/auth/` - Authentication module (in progress)
- `src/models/User.ts` - User model (complete)
- `src/middleware/auth.ts` - Auth middleware (blocked on this decision)

---

## Current State

### Completed
- [x] User model with password hashing (Argon2)
- [x] Registration endpoint with email verification
- [x] Password reset flow with secure tokens
- [x] Login endpoint (basic implementation)

### In Progress
- [ ] Session management <- **BLOCKED**
- [ ] Token refresh mechanism <- **BLOCKED**
- [ ] Auth middleware <- **BLOCKED**

### Blocking Issue
Token storage strategy must be decided before implementing session management.
This decision affects:
- How tokens are sent to the client
- How tokens are included in subsequent requests
- Security model for the entire authentication system
- Frontend implementation approach

---

## Specific Question

**How should JWT tokens be stored on the client side?**

This is a security-sensitive decision with long-term architectural implications.
Multiple valid approaches exist, each with different tradeoffs. Human judgment
is required because:

1. Organizational security policies may prefer certain approaches
2. The decision affects code we don't control (frontend)
3. Wrong choice has significant security implications
4. This decision is difficult to change later

---

## Options Identified

### Option A: localStorage

**Description:**
Store JWT in browser's localStorage. Send in Authorization header.

**Implementation:**
```javascript
// Client-side
localStorage.setItem('token', jwt);

// On each request
fetch('/api/data', {
  headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
});
```

**Pros:**
- Simple implementation
- Works easily with any API
- Token persists across browser sessions
- Easy to debug (visible in DevTools)
- Works well with mobile apps using same API

**Cons:**
- Vulnerable to XSS attacks (JavaScript can read localStorage)
- Must manually attach to every request
- Cannot be invalidated server-side without checking database
- Token visible in browser DevTools

**Security Assessment:** Medium risk
- Any XSS vulnerability exposes all tokens
- Mitigated by strong CSP and input sanitization

**Effort Estimate:** 2 story points

---

### Option B: httpOnly Cookies

**Description:**
Store JWT in httpOnly cookie. Browser automatically sends with requests.

**Implementation:**
```typescript
// Server-side
res.cookie('token', jwt, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 900000 // 15 minutes
});

// Client-side - no code needed, cookie sent automatically
fetch('/api/data', { credentials: 'include' });
```

**Pros:**
- Not accessible via JavaScript (XSS protected)
- Automatically sent with requests (no frontend code needed)
- Server can invalidate by clearing cookie
- Well-established security pattern

**Cons:**
- Requires CSRF protection (sameSite helps but not complete)
- Slightly more complex server setup
- Cookies sent to all matching routes (potential for leakage)
- CORS configuration more complex

**Security Assessment:** Lower risk
- XSS cannot directly steal tokens
- CSRF mitigated by sameSite + CSRF tokens
- Defense in depth

**Effort Estimate:** 3 story points

---

### Option C: Memory + Refresh Token in Cookie

**Description:**
Store access token in memory only. Store refresh token in httpOnly cookie.
Re-fetch access token on page load.

**Implementation:**
```typescript
// Server-side
res.cookie('refresh_token', refreshJwt, { httpOnly: true, secure: true });
res.json({ access_token: accessJwt }); // In response body

// Client-side
let accessToken = null; // In-memory only

async function getAccessToken() {
  if (!accessToken || isExpired(accessToken)) {
    const response = await fetch('/api/auth/refresh', { credentials: 'include' });
    accessToken = (await response.json()).access_token;
  }
  return accessToken;
}
```

**Pros:**
- Most secure option
- Access token never persisted
- Refresh token has XSS protection
- Short-lived access tokens limit damage window

**Cons:**
- Most complex implementation
- Token lost on page refresh (requires re-fetch)
- Higher latency on first request after refresh
- More complex frontend state management

**Security Assessment:** Lowest risk
- Best protection against token theft
- Short-lived tokens minimize damage if compromised

**Effort Estimate:** 5 story points

---

## Comparative Analysis

| Factor | Option A | Option B | Option C |
|--------|----------|----------|----------|
| XSS Protection | None | Full | Full |
| CSRF Protection | Natural | Needs mitigation | Needs mitigation |
| Implementation Complexity | Low | Medium | High |
| Frontend Changes | Medium | Low | High |
| Mobile App Compatibility | Excellent | Moderate | Moderate |
| Server-Side Invalidation | Difficult | Easy | Easy |
| Story Points | 2 | 3 | 5 |

---

## My Recommendation

**I recommend Option B (httpOnly Cookies)** for the following reasons:

1. **Best security/complexity balance** - Provides strong XSS protection without
   the complexity of Option C

2. **Matches existing patterns** - I noticed other services in this codebase use
   cookie-based authentication (see `services/admin/auth.ts`)

3. **Within sprint capacity** - 3 story points fits our remaining sprint budget,
   while Option C's 5 points would risk spillover

4. **Adequate for use case** - For a standard web application without extreme
   security requirements, httpOnly cookies provide sufficient protection

**However**, if any of these conditions are true, I would reconsider:
- This handles financial transactions or highly sensitive data (consider Option C)
- Mobile app API compatibility is critical (consider Option A with strong CSP)
- Organizational policy mandates a specific approach

---

## Decision Needed By

**2026-02-09 EOD** to maintain sprint timeline.

Impact of delay:
- Each day of delay blocks 3 other authentication tasks
- Sprint 23 delivery at risk if decision delayed past 2026-02-10

---

## How to Respond

Please create a VCR (Version Controlled Resolution) with:
1. Your decision (A, B, or C, or a modification)
2. Your rationale
3. Any additional implementation requirements
4. Who else should be consulted (if anyone)

Save the VCR to `vcrs/VCR-2026-0042.md` and I will continue implementation
based on your decision.

---

## Supporting Information

**Security Requirements (from PRD):**
- All authentication tokens must be protected against theft
- Session timeout: 24 hours of inactivity
- Support for forced logout (security incidents)

**Related Decisions:**
- VCR-2026-0038: Password hashing uses Argon2
- VCR-2026-0035: Session timeout is 24 hours

**Questions to Consider:**
- Do we need to support the mobile app with this same API?
- Are there compliance requirements I should know about?
- Is there a preferred organizational pattern for this?
```

**Automation for CRP Creation:**

You can provide agents with a CRP creation prompt:

```
When you encounter a decision that requires human judgment, create a CRP:

1. Create a new file: crps/CRP-YYYY-NNNN.md (increment NNNN)
2. Use the CRP template
3. Include at least 2-3 options with honest tradeoffs
4. Provide your recommendation with reasoning
5. Specify when a decision is needed
6. Pause work on blocked items until VCR is received

Do NOT:
- Make the decision yourself for security/architecture issues
- Assume what the human will choose
- Continue with blocked work while waiting
```

---

## Automated MRP Generation

### Why Automate MRP Generation?

Manual MRP creation is tedious. Since most MRP content comes from automated tools (test results, lint output, coverage reports), this process can be automated.

### MRP Generation Approach

An automated MRP generator:
1. Runs all verification tools (tests, linting, security scans)
2. Collects results into structured format
3. Generates the MRP document
4. Leaves placeholders for human-authored sections (rationale)

### What Can Be Automated

| MRP Section | Automation Level |
|-------------|-----------------|
| Test Results | Fully automated |
| Coverage Metrics | Fully automated |
| Lint/Type Check | Fully automated |
| Security Scan | Fully automated |
| Files Changed | Fully automated |
| Commit History | Fully automated |
| Rationale | Human-authored |
| Approach Justification | Human-authored |

### CI/CD Integration

Automated MRP generation fits naturally into CI/CD:

1. PR triggers CI pipeline
2. Pipeline runs all checks
3. Pipeline generates MRP from results
4. MRP is attached to PR as comment or artifact
5. Reviewer evaluates MRP rather than reading all code
6. Human adds rationale if not already present

### Customization

Teams should customize MRP generation for their needs:
- Which tools to run
- What thresholds indicate "pass"
- Required vs. optional sections
- Format (Markdown, JSON, etc.)

### Complete MRP Generation Script

The following script automates MRP generation for Python projects:

```python
#!/usr/bin/env python3
"""
generate_mrp.py - Automated Merge-Readiness Pack Generator

Generates a comprehensive MRP by running all verification tools
and collecting results into a structured document.

Usage:
    python generate_mrp.py feature-branch
    python generate_mrp.py feature-branch --output mrp.md
    python generate_mrp.py --pr 123  # Generate for a specific PR
"""

import argparse
import json
import subprocess
import sys
from dataclasses import dataclass, field
from datetime import datetime
from pathlib import Path
from typing import Optional


@dataclass
class CheckResult:
    """Result of a single verification check."""
    name: str
    passed: bool
    output: str
    details: dict = field(default_factory=dict)


@dataclass
class MRPReport:
    """Complete MRP report structure."""
    branch: str
    base_branch: str
    generated_at: datetime
    functional: list[CheckResult] = field(default_factory=list)
    verification: list[CheckResult] = field(default_factory=list)
    hygiene: list[CheckResult] = field(default_factory=list)
    files_changed: list[str] = field(default_factory=list)
    commits: list[str] = field(default_factory=list)
    coverage_percent: float = 0.0
    rationale: str = "_To be filled by author_"


def run_command(cmd: list[str], timeout: int = 300) -> tuple[int, str, str]:
    """Run a command and return exit code, stdout, stderr."""
    try:
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True,
            timeout=timeout
        )
        return result.returncode, result.stdout, result.stderr
    except subprocess.TimeoutExpired:
        return 1, "", f"Command timed out after {timeout}s"
    except FileNotFoundError:
        return 1, "", f"Command not found: {cmd[0]}"


def get_commits(branch: str, base: str = "main") -> list[str]:
    """Get list of commits on branch since diverging from base."""
    code, stdout, _ = run_command([
        "git", "log", "--oneline", f"{base}..{branch}"
    ])
    if code == 0:
        return [line.strip() for line in stdout.strip().split('\n') if line]
    return []


def get_files_changed(branch: str, base: str = "main") -> list[str]:
    """Get list of files changed on branch."""
    code, stdout, _ = run_command([
        "git", "diff", "--name-only", f"{base}..{branch}"
    ])
    if code == 0:
        return [line.strip() for line in stdout.strip().split('\n') if line]
    return []


def run_tests() -> CheckResult:
    """Run test suite and collect results."""
    code, stdout, stderr = run_command(["pytest", "--tb=short", "-q"])

    # Parse test results
    passed = code == 0
    test_count = 0
    failed_count = 0

    for line in stdout.split('\n'):
        if 'passed' in line:
            parts = line.split()
            for i, part in enumerate(parts):
                if part == 'passed' and i > 0:
                    try:
                        test_count = int(parts[i-1])
                    except ValueError:
                        pass
                if part == 'failed' and i > 0:
                    try:
                        failed_count = int(parts[i-1])
                    except ValueError:
                        pass

    return CheckResult(
        name="Test Suite",
        passed=passed,
        output=stdout if len(stdout) < 2000 else stdout[-2000:],
        details={
            "total_tests": test_count + failed_count,
            "passed": test_count,
            "failed": failed_count
        }
    )


def run_coverage() -> tuple[CheckResult, float]:
    """Run coverage analysis."""
    code, stdout, stderr = run_command([
        "pytest", "--cov=src", "--cov-report=term-missing", "-q"
    ])

    # Parse coverage percentage
    coverage_pct = 0.0
    for line in stdout.split('\n'):
        if 'TOTAL' in line:
            parts = line.split()
            for part in parts:
                if part.endswith('%'):
                    try:
                        coverage_pct = float(part.rstrip('%'))
                    except ValueError:
                        pass

    passed = coverage_pct >= 80  # Configurable threshold

    return CheckResult(
        name="Test Coverage",
        passed=passed,
        output=stdout[-1500:] if len(stdout) > 1500 else stdout,
        details={
            "coverage_percent": coverage_pct,
            "threshold": 80
        }
    ), coverage_pct


def run_linting() -> CheckResult:
    """Run linting checks."""
    code, stdout, stderr = run_command(["ruff", "check", "src/"])

    return CheckResult(
        name="Linting (Ruff)",
        passed=code == 0,
        output=stdout if code != 0 else "No issues found",
        details={"exit_code": code}
    )


def run_type_check() -> CheckResult:
    """Run type checking."""
    code, stdout, stderr = run_command(["mypy", "src/", "--ignore-missing-imports"])

    # Count errors
    error_count = stdout.count(': error:')

    return CheckResult(
        name="Type Checking (Mypy)",
        passed=code == 0,
        output=stdout if code != 0 else "No type errors",
        details={"error_count": error_count}
    )


def run_security_scan() -> CheckResult:
    """Run security scanning."""
    code, stdout, stderr = run_command([
        "bandit", "-r", "src/", "-f", "json", "-ll"
    ])

    findings = []
    if stdout:
        try:
            data = json.loads(stdout)
            findings = data.get('results', [])
        except json.JSONDecodeError:
            pass

    critical = sum(1 for f in findings if f.get('issue_severity') == 'HIGH')
    medium = sum(1 for f in findings if f.get('issue_severity') == 'MEDIUM')

    return CheckResult(
        name="Security Scan (Bandit)",
        passed=critical == 0,
        output=f"Found {critical} high, {medium} medium severity issues",
        details={
            "high_severity": critical,
            "medium_severity": medium,
            "total_findings": len(findings)
        }
    )


def run_dependency_check() -> CheckResult:
    """Check for vulnerable dependencies."""
    code, stdout, stderr = run_command(["pip-audit", "--format=json"])

    vulnerabilities = []
    if stdout:
        try:
            vulnerabilities = json.loads(stdout)
        except json.JSONDecodeError:
            pass

    return CheckResult(
        name="Dependency Audit",
        passed=len(vulnerabilities) == 0,
        output=f"Found {len(vulnerabilities)} vulnerable dependencies",
        details={"vulnerabilities": len(vulnerabilities)}
    )


def generate_mrp(branch: str, base: str = "main") -> MRPReport:
    """Generate complete MRP for a branch."""
    report = MRPReport(
        branch=branch,
        base_branch=base,
        generated_at=datetime.now()
    )

    print(f"Generating MRP for {branch}...")

    # Collect git information
    print("  Collecting commits...")
    report.commits = get_commits(branch, base)
    report.files_changed = get_files_changed(branch, base)

    # Run verification checks
    print("  Running tests...")
    report.verification.append(run_tests())

    print("  Checking coverage...")
    coverage_result, coverage_pct = run_coverage()
    report.verification.append(coverage_result)
    report.coverage_percent = coverage_pct

    # Run hygiene checks
    print("  Running linter...")
    report.hygiene.append(run_linting())

    print("  Running type checker...")
    report.hygiene.append(run_type_check())

    print("  Running security scan...")
    report.hygiene.append(run_security_scan())

    print("  Checking dependencies...")
    report.hygiene.append(run_dependency_check())

    return report


def render_mrp_markdown(report: MRPReport) -> str:
    """Render MRP report as Markdown."""
    lines = [
        "# Merge-Readiness Pack",
        "",
        f"**Branch:** `{report.branch}` -> `{report.base_branch}`",
        f"**Generated:** {report.generated_at.isoformat()}",
        "",
        "---",
        "",
        "## 1. Functional Completeness",
        "",
        f"### Commits ({len(report.commits)})",
        "",
    ]

    for commit in report.commits:
        lines.append(f"- `{commit}`")

    lines.extend([
        "",
        "### Files Changed",
        "",
        "```",
    ])
    lines.extend(report.files_changed)
    lines.append("```")

    lines.extend([
        "",
        "---",
        "",
        "## 2. Verification Quality",
        "",
    ])

    for check in report.verification:
        status = "PASS" if check.passed else "FAIL"
        emoji = "+" if check.passed else "-"
        lines.append(f"### {check.name}: {status}")
        lines.append("")
        if check.details:
            for key, value in check.details.items():
                lines.append(f"- **{key}:** {value}")
        lines.append("")
        lines.append("```")
        lines.append(check.output[:1000])
        lines.append("```")
        lines.append("")

    lines.extend([
        "---",
        "",
        "## 3. Code Hygiene",
        "",
    ])

    for check in report.hygiene:
        status = "PASS" if check.passed else "FAIL"
        lines.append(f"- **{check.name}:** {status}")
        if not check.passed:
            lines.append(f"  - {check.output[:200]}")

    lines.extend([
        "",
        "---",
        "",
        "## 4. Rationale",
        "",
        report.rationale,
        "",
        "_Author should explain:_",
        "- Why this approach was chosen",
        "- Alternatives considered",
        "- Any tradeoffs made",
        "",
        "---",
        "",
        "## 5. Auditability",
        "",
        f"- **Generated:** {report.generated_at.isoformat()}",
        f"- **Branch:** {report.branch}",
        f"- **Base:** {report.base_branch}",
        f"- **Commits:** {len(report.commits)}",
        f"- **Files Changed:** {len(report.files_changed)}",
        f"- **Test Coverage:** {report.coverage_percent:.1f}%",
        "",
        "---",
        "",
        "## Summary",
        "",
    ])

    all_checks = report.verification + report.hygiene
    passed = sum(1 for c in all_checks if c.passed)
    total = len(all_checks)

    overall = "READY" if passed == total else "NEEDS ATTENTION"
    lines.append(f"**Overall Status:** {overall} ({passed}/{total} checks passing)")

    if passed < total:
        lines.append("")
        lines.append("**Failing Checks:**")
        for check in all_checks:
            if not check.passed:
                lines.append(f"- {check.name}")

    return '\n'.join(lines)


def main():
    parser = argparse.ArgumentParser(description="Generate Merge-Readiness Pack")
    parser.add_argument("branch", nargs="?", default="HEAD",
                       help="Branch to generate MRP for")
    parser.add_argument("--base", default="main",
                       help="Base branch to compare against")
    parser.add_argument("--output", "-o",
                       help="Output file (default: stdout)")
    parser.add_argument("--json", action="store_true",
                       help="Output as JSON instead of Markdown")

    args = parser.parse_args()

    report = generate_mrp(args.branch, args.base)

    if args.json:
        import dataclasses
        output = json.dumps(dataclasses.asdict(report), default=str, indent=2)
    else:
        output = render_mrp_markdown(report)

    if args.output:
        Path(args.output).write_text(output)
        print(f"MRP written to {args.output}")
    else:
        print(output)

    # Exit with error if any checks failed
    all_checks = report.verification + report.hygiene
    if not all(c.passed for c in all_checks):
        sys.exit(1)


if __name__ == "__main__":
    main()
```

**Using the MRP Generator:**

```bash
# Generate MRP for current branch
python generate_mrp.py feature/new-auth

# Generate MRP and save to file
python generate_mrp.py feature/new-auth -o mrp.md

# Generate JSON output for CI/CD integration
python generate_mrp.py feature/new-auth --json > mrp.json

# Compare against different base branch
python generate_mrp.py feature/new-auth --base develop
```

**CI/CD Integration:**

```yaml
# .github/workflows/mrp.yml
name: Generate MRP

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  mrp:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Generate MRP
        run: python scripts/generate_mrp.py ${{ github.head_ref }} --base ${{ github.base_ref }} -o mrp.md

      - name: Comment MRP on PR
        uses: actions/github-script@v6
        with:
          script: |
            const fs = require('fs');
            const mrp = fs.readFileSync('mrp.md', 'utf8');
            github.rest.issues.createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: mrp
            });
```

---

## Documentation Generation

### AI-Assisted Documentation

AI excels at generating documentation from code:
- API documentation from function signatures and docstrings
- README files from project structure
- Architecture docs from code organization
- User guides from feature implementations

### From Code to Docs

**Prompt Pattern:**
```
Review this module and generate documentation:

1. PURPOSE: What does this module do?
2. USAGE: How do you use it? Include examples.
3. API REFERENCE: Document each public function/class
4. DEPENDENCIES: What does it require?
5. CONFIGURATION: What can be configured?

Use the existing documentation style in the project.
```

### Keeping Docs in Sync

Documentation drift is a common problem. AI can help:

**Verification Prompt:**
```
Compare this code with its documentation:

Code: [code file]
Docs: [documentation file]

Identify:
1. Features in code not documented
2. Documentation for features that no longer exist
3. Incorrect or outdated examples
4. Parameter mismatches

List discrepancies without making changes.
```

**Update Prompt:**
```
Update the documentation to match the current code.
Preserve the existing style and format.
Only change what's necessary for accuracy.
```

### Documentation in MRPs

Documentation updates should be part of the MRP:
- List documentation files created or modified
- Note if documentation matches code changes
- Flag if documentation update is still needed

---

## Multi-Agent Review Patterns

### Multi-Agent Review Pipeline

The following diagram shows how multiple specialized agents work together in a review pipeline:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    MULTI-AGENT REVIEW PIPELINE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────┐                                                          │
│  │  CODE CHANGE  │                                                          │
│  │   (PR/Diff)   │                                                          │
│  └───────┬───────┘                                                          │
│          │                                                                   │
│          ▼                                                                   │
│  ┌───────────────────────────────────────────────────────────────────┐     │
│  │                     AUTOMATED REVIEW TIER                          │     │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐             │     │
│  │  │   SECURITY   │  │ PERFORMANCE  │  │    STYLE     │             │     │
│  │  │   REVIEWER   │  │   REVIEWER   │  │   REVIEWER   │             │     │
│  │  │              │  │              │  │              │             │     │
│  │  │ • OWASP scan │  │ • N+1 detect │  │ • Lint check │             │     │
│  │  │ • Input val  │  │ • Complexity │  │ • Patterns   │             │     │
│  │  │ • Auth check │  │ • Caching    │  │ • Naming     │             │     │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘             │     │
│  │         │                 │                 │                      │     │
│  │         └────────────┬────┴─────────────────┘                      │     │
│  │                      ▼                                             │     │
│  │              ┌──────────────┐                                      │     │
│  │              │   FINDINGS   │                                      │     │
│  │              │  AGGREGATOR  │                                      │     │
│  │              └──────┬───────┘                                      │     │
│  └──────────────────────│────────────────────────────────────────────┘     │
│                         │                                                   │
│                         ▼                                                   │
│           ┌─────────────────────────────────┐                              │
│           │  Critical issues?                │                              │
│           └──────────────┬──────────────────┘                              │
│                  ┌───────┴───────┐                                         │
│                  ▼               ▼                                          │
│              [YES]            [NO]                                          │
│                │               │                                            │
│                ▼               ▼                                            │
│     ┌─────────────────┐  ┌─────────────────┐                               │
│     │  BLOCK MERGE    │  │  HUMAN REVIEW   │                               │
│     │  Auto-request   │  │  Focus on:      │                               │
│     │  fixes from     │  │  • Judgment     │                               │
│     │  author         │  │  • Architecture │                               │
│     └─────────────────┘  │  • Context      │                               │
│                          └────────┬────────┘                               │
│                                   │                                         │
│                                   ▼                                         │
│                          ┌─────────────────┐                               │
│                          │  MERGE OR       │                               │
│                          │  REQUEST CHANGES│                               │
│                          └─────────────────┘                               │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  BENEFIT: Humans review evidence + judgment calls, not every line of code   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Pattern 1: Generator-Critic Review

Separate code generation from quality assessment:

```
┌─────────────────────────────────────────────────────┐
│                                                     │
│  ┌───────────┐         ┌───────────┐               │
│  │ Generator │  ──→    │   Critic  │               │
│  │  Agent    │  ←──    │   Agent   │               │
│  └───────────┘         └───────────┘               │
│       │                      │                     │
│       │   Iterate until      │                     │
│       │   Critic approves    │                     │
│       ▼                      │                     │
│  ┌───────────┐              │                     │
│  │  Approved │ ◀────────────┘                     │
│  │   Code    │                                     │
│  └───────────┘                                     │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Implementation:**

```
Use a Generator-Critic pattern:

GENERATOR: Implement the user search feature according to the requirements.

CRITIC: Review the implementation for:
- Correctness: Does it meet all requirements?
- Security: Any vulnerabilities (SQL injection, XSS)?
- Performance: Any obvious inefficiencies?
- Maintainability: Does it follow project patterns?

If Critic finds issues, Generator must address them.
Iterate until Critic approves with no remaining concerns.
```

### Pattern 2: Independent Verification

Use separate context to verify work:

```
Implementation is complete. Now verify with independent review:

1. Start a new context (or use subagent)
2. DO NOT share the implementation approach
3. Only share: requirements and the generated code
4. Ask: Does this code correctly implement the requirements?

The reviewer should evaluate without knowing what the implementer intended.
```

### Pattern 3: Specialized Reviewers

Different agents check different aspects:

```
The feature is implemented. Run specialized reviews:

SECURITY REVIEWER:
- Check for OWASP Top 10 vulnerabilities
- Verify input validation
- Check authentication/authorization

PERFORMANCE REVIEWER:
- Identify potential N+1 queries
- Check for unnecessary computations
- Verify caching opportunities

MAINTAINABILITY REVIEWER:
- Check code follows project patterns
- Verify documentation is adequate
- Check test coverage

Each reviewer provides findings. All critical issues must be addressed.
```

---

## AI as Subjective Reviewer

Beyond objective correctness, AI excels at subjective review:

### What AI Can Catch

| Issue Type | Example |
|-----------|---------|
| Typos | `functoin` → `function` |
| Stale comments | Comment says "TODO" but code is implemented |
| Misleading names | `getUserName()` that returns user ID |
| Inconsistent style | Mixed camelCase and snake_case |
| Dead code | Unreachable branches, unused variables |
| Over-complexity | Unnecessarily nested logic |

### Subjective Review Prompt

```
Review this code for subjective quality issues:

1. Naming:
   - Are variable/function names descriptive?
   - Are names consistent with project conventions?
   - Any misleading names?

2. Comments:
   - Are comments up-to-date with code?
   - Any TODO/FIXME that should be addressed?
   - Any comments that state the obvious?

3. Structure:
   - Is the code unnecessarily complex?
   - Could any logic be simplified?
   - Is the flow easy to follow?

4. Consistency:
   - Does this match patterns elsewhere in the codebase?
   - Any stylistic inconsistencies?

Don't suggest changes that are purely preferential. Only flag issues
that would genuinely cause confusion or maintenance problems.
```

### Handling False Positives

AI reviewers will produce false positives. Handle them systematically:

**1. Document Known False Positives**
Keep a list of patterns that AI incorrectly flags:
- Specific constructs your codebase uses intentionally
- Framework idioms that look suspicious but are correct
- Project-specific patterns

**2. Configure Review Scope**
Tell AI what NOT to flag:
```
Do not flag the following as issues:
- Our custom logging format (looks non-standard but is intentional)
- The use of `eval()` in our template engine (sandboxed and safe)
- Apparent SQL injection in our query builder (it's parameterized internally)
```

**3. Calibrate Over Time**
Track false positive rate and refine prompts:
- If AI keeps flagging the same non-issue, add to exclusions
- If AI misses real issues, strengthen that area of the prompt
- Review calibration quarterly

**4. Human Override Protocol**
When a human determines an AI flag is false positive:
- Document why it's not actually an issue
- Add to known false positives list
- Consider if prompt should be updated

---

## Evaluation Frameworks

### Types of Evaluation

| Type | What It Measures | Method |
|------|-----------------|--------|
| **Code-based** | Objective correctness | Tests, linting, type checking |
| **Model-based** | Subjective quality | LLM judges with rubrics |
| **Human-based** | Real-world acceptability | Expert review |

### Building Evaluations

**Step 1: Define Success Criteria**
```
For this feature, success means:
- All acceptance criteria from spec are met
- No critical security vulnerabilities
- Test coverage > 80%
- No regression in existing tests
- Code review approval from at least 1 team member
```

**Step 2: Create Automated Checks**
```yaml
evaluation:
  code_based:
    - name: tests_pass
      command: npm test
      expect: exit_code_0

    - name: type_check
      command: npm run typecheck
      expect: exit_code_0

    - name: security_scan
      command: npm run security
      expect: no_critical_findings

    - name: coverage
      command: npm run coverage
      expect: greater_than_80_percent
```

**Step 3: Add Model-Based Evaluation**
```
Evaluate this implementation using the following rubric:

CORRECTNESS (0-5):
5 = All requirements fully implemented
3 = Most requirements met, minor gaps
1 = Major requirements missing
0 = Does not address requirements

SECURITY (0-5):
5 = No vulnerabilities, follows best practices
3 = Minor issues, no critical vulnerabilities
1 = Some vulnerabilities present
0 = Critical security flaws

MAINTAINABILITY (0-5):
5 = Clear, well-structured, easy to modify
3 = Readable but some complexity
1 = Difficult to understand or modify
0 = Unmaintainable

OVERALL SCORE: Sum of all categories (0-15)
Passing threshold: 12+
```

### pass@k and pass^k Metrics

**pass@k:** Probability of at least one success in k attempts
- Higher k → higher pass@k (more chances)
- Good for: "Can the AI ever solve this?"

**pass^k:** Probability of all k attempts succeeding
- Higher k → lower pass^k (harder to be consistent)
- Good for: "Can the AI solve this reliably?"

```
If single-attempt success rate is 80%:
- pass@3 = 1 - (0.2)³ = 99.2%   (at least 1 success in 3 tries)
- pass^3 = (0.8)³ = 51.2%      (all 3 succeed)
```

### pass@k vs pass^k Visualization

The following diagram illustrates the critical difference between pass@k and pass^k:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    pass@k vs pass^k COMPARISON                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  SINGLE ATTEMPT SUCCESS RATE: 80%                                           │
│                                                                              │
│  pass@k: AT LEAST ONE success in k attempts                                 │
│  ────────────────────────────────────────                                   │
│                                                                              │
│  k=1: ████████████████████████████████████████░░░░░░░░░░  80.0%            │
│  k=2: ████████████████████████████████████████████████░░  96.0%            │
│  k=3: █████████████████████████████████████████████████░  99.2%            │
│  k=5: ██████████████████████████████████████████████████  99.97%           │
│                                                                              │
│  Interpretation: "Given enough tries, we'll probably get it right"          │
│                                                                              │
│  ─────────────────────────────────────────────────────────────────          │
│                                                                              │
│  pass^k: ALL k attempts succeed                                              │
│  ──────────────────────────────                                             │
│                                                                              │
│  k=1: ████████████████████████████████████████░░░░░░░░░░  80.0%            │
│  k=2: ████████████████████████████████░░░░░░░░░░░░░░░░░░  64.0%            │
│  k=3: ██████████████████████████░░░░░░░░░░░░░░░░░░░░░░░░  51.2%            │
│  k=5: █████████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░  32.8%            │
│                                                                              │
│  Interpretation: "Can we rely on this for repeated/autonomous use?"          │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  PRODUCTION GUIDANCE:                                                        │
│                                                                              │
│  ┌─────────────────────┐          ┌─────────────────────┐                   │
│  │  EXPLORATION/PROTO  │          │  PRODUCTION/AUTO    │                   │
│  │                     │          │                     │                   │
│  │  Use pass@k         │          │  Use pass^k         │                   │
│  │  "Can AI help?"     │          │  "Can AI be trusted?"│                   │
│  │                     │          │                     │                   │
│  │  Accept: 80%+ @3    │          │  Require: 90%+ ^3   │                   │
│  └─────────────────────┘          └─────────────────────┘                   │
│                                                                              │
│  WARNING: High pass@k + Low pass^k = unreliable for autonomous operation    │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Use pass^k for production agents** where reliability matters.

### Complete Evaluation Scoring Script

The following script provides comprehensive evaluation of AI-generated code:

```python
#!/usr/bin/env python3
"""
evaluate_ai_code.py - Comprehensive Evaluation Framework for AI-Generated Code

This script evaluates AI-generated code using multiple criteria:
- Code-based (objective): Tests, linting, type checking
- Model-based (subjective): LLM-judged quality with rubrics
- Metrics: pass@k, pass^k, and custom scoring

Usage:
    python evaluate_ai_code.py --task task.json --output results.json
    python evaluate_ai_code.py --benchmark benchmarks/ --runs 5
"""

import argparse
import json
import subprocess
import tempfile
import statistics
from dataclasses import dataclass, field, asdict
from datetime import datetime
from pathlib import Path
from typing import Optional, Callable
from enum import Enum


# ============================================================
# Data Models
# ============================================================

class EvalType(Enum):
    CODE_BASED = "code_based"
    MODEL_BASED = "model_based"
    HUMAN_BASED = "human_based"


@dataclass
class RubricCriterion:
    """Single criterion in an evaluation rubric."""
    name: str
    description: str
    max_score: int
    scoring_guide: dict[int, str]  # score -> description


@dataclass
class EvaluationRubric:
    """Complete evaluation rubric."""
    name: str
    criteria: list[RubricCriterion]
    passing_threshold: float = 0.8  # Percentage of max score

    @property
    def max_score(self) -> int:
        return sum(c.max_score for c in self.criteria)

    def passing_score(self) -> float:
        return self.max_score * self.passing_threshold


@dataclass
class TaskResult:
    """Result of a single task attempt."""
    task_id: str
    attempt: int
    passed: bool
    code_score: float
    model_score: Optional[float]
    execution_time_ms: int
    errors: list[str] = field(default_factory=list)
    details: dict = field(default_factory=dict)


@dataclass
class EvaluationReport:
    """Complete evaluation report across all attempts."""
    task_id: str
    task_description: str
    total_attempts: int
    results: list[TaskResult]
    pass_at_1: float
    pass_at_k: float
    pass_power_k: float
    mean_score: float
    std_score: float
    generated_at: datetime = field(default_factory=datetime.now)


# ============================================================
# Default Rubric
# ============================================================

DEFAULT_RUBRIC = EvaluationRubric(
    name="Standard AI Code Evaluation",
    criteria=[
        RubricCriterion(
            name="Correctness",
            description="Does the code correctly implement the requirements?",
            max_score=5,
            scoring_guide={
                5: "All requirements fully and correctly implemented",
                4: "All requirements met with minor issues",
                3: "Most requirements met, some gaps",
                2: "Some requirements met, significant gaps",
                1: "Few requirements met",
                0: "Does not address requirements"
            }
        ),
        RubricCriterion(
            name="Security",
            description="Is the code free from security vulnerabilities?",
            max_score=5,
            scoring_guide={
                5: "No vulnerabilities, follows security best practices",
                4: "No critical vulnerabilities, minor improvements possible",
                3: "Low-severity issues only",
                2: "Medium-severity vulnerabilities present",
                1: "High-severity vulnerabilities present",
                0: "Critical security flaws"
            }
        ),
        RubricCriterion(
            name="Maintainability",
            description="Is the code clean, readable, and maintainable?",
            max_score=5,
            scoring_guide={
                5: "Excellent structure, clear naming, well-documented",
                4: "Good structure with minor improvements possible",
                3: "Readable but some complexity issues",
                2: "Difficult to follow in places",
                1: "Hard to understand or modify",
                0: "Unmaintainable"
            }
        ),
    ],
    passing_threshold=0.8
)


# ============================================================
# Code-Based Evaluation
# ============================================================

class CodeBasedEvaluator:
    """Evaluates code using objective, automated checks."""

    def __init__(self, project_dir: Path):
        self.project_dir = project_dir

    def run_tests(self) -> tuple[bool, dict]:
        """Run test suite and return pass/fail with details."""
        result = subprocess.run(
            ["pytest", "--tb=short", "-q", "--json-report", "--json-report-file=-"],
            cwd=self.project_dir,
            capture_output=True,
            text=True
        )

        details = {"exit_code": result.returncode}

        try:
            report = json.loads(result.stdout.split('\n')[-2])
            details["total"] = report.get("summary", {}).get("total", 0)
            details["passed"] = report.get("summary", {}).get("passed", 0)
            details["failed"] = report.get("summary", {}).get("failed", 0)
        except (json.JSONDecodeError, IndexError):
            details["output"] = result.stdout[-500:]

        return result.returncode == 0, details

    def run_linting(self) -> tuple[bool, dict]:
        """Run linting checks."""
        result = subprocess.run(
            ["ruff", "check", "."],
            cwd=self.project_dir,
            capture_output=True,
            text=True
        )

        return result.returncode == 0, {
            "exit_code": result.returncode,
            "issues": result.stdout.count('\n') if result.stdout else 0
        }

    def run_type_check(self) -> tuple[bool, dict]:
        """Run type checking."""
        result = subprocess.run(
            ["mypy", ".", "--ignore-missing-imports"],
            cwd=self.project_dir,
            capture_output=True,
            text=True
        )

        return result.returncode == 0, {
            "exit_code": result.returncode,
            "errors": result.stdout.count(": error:") if result.stdout else 0
        }

    def run_security_scan(self) -> tuple[bool, dict]:
        """Run security scanning."""
        result = subprocess.run(
            ["bandit", "-r", ".", "-f", "json", "-ll"],
            cwd=self.project_dir,
            capture_output=True,
            text=True
        )

        findings = []
        try:
            data = json.loads(result.stdout)
            findings = data.get("results", [])
        except json.JSONDecodeError:
            pass

        high_severity = sum(1 for f in findings if f.get("issue_severity") == "HIGH")

        return high_severity == 0, {
            "total_findings": len(findings),
            "high_severity": high_severity
        }

    def evaluate(self) -> tuple[float, dict]:
        """
        Run all code-based checks and return a normalized score.

        Returns:
            Tuple of (score 0-1, details dict)
        """
        checks = [
            ("tests", self.run_tests, 0.4),      # 40% weight
            ("linting", self.run_linting, 0.2),  # 20% weight
            ("types", self.run_type_check, 0.2), # 20% weight
            ("security", self.run_security_scan, 0.2),  # 20% weight
        ]

        total_score = 0.0
        details = {}

        for name, check_fn, weight in checks:
            passed, check_details = check_fn()
            details[name] = {"passed": passed, **check_details}
            if passed:
                total_score += weight

        return total_score, details


# ============================================================
# Model-Based Evaluation
# ============================================================

class ModelBasedEvaluator:
    """Evaluates code using LLM-based judgment with rubrics."""

    def __init__(self, api_client, rubric: EvaluationRubric):
        self.api = api_client
        self.rubric = rubric

    def build_evaluation_prompt(self, code: str, requirements: str) -> str:
        """Build the evaluation prompt with rubric."""
        rubric_text = []
        for criterion in self.rubric.criteria:
            rubric_text.append(f"\n### {criterion.name} (0-{criterion.max_score})")
            rubric_text.append(criterion.description)
            for score, desc in sorted(criterion.scoring_guide.items(), reverse=True):
                rubric_text.append(f"  {score}: {desc}")

        return f"""You are evaluating AI-generated code. Score the following code
against the requirements using the provided rubric.

## Requirements
{requirements}

## Code to Evaluate
```
{code}
```

## Evaluation Rubric
{"".join(rubric_text)}

## Your Task
1. Analyze the code against each criterion
2. Provide a score for each criterion
3. Justify each score with specific observations

Respond in JSON format:
{{
  "scores": {{
    "Correctness": {{"score": <0-5>, "justification": "..."}},
    "Security": {{"score": <0-5>, "justification": "..."}},
    "Maintainability": {{"score": <0-5>, "justification": "..."}}
  }},
  "overall_assessment": "...",
  "improvement_suggestions": ["...", "..."]
}}
"""

    def evaluate(self, code: str, requirements: str) -> tuple[float, dict]:
        """
        Evaluate code using the LLM with rubric.

        Returns:
            Tuple of (normalized score 0-1, details dict)
        """
        prompt = self.build_evaluation_prompt(code, requirements)

        response = self.api.complete(prompt, response_format="json")

        try:
            result = json.loads(response)
            scores = result.get("scores", {})

            total_score = sum(
                scores.get(c.name, {}).get("score", 0)
                for c in self.rubric.criteria
            )
            normalized = total_score / self.rubric.max_score

            return normalized, {
                "raw_scores": scores,
                "total_score": total_score,
                "max_score": self.rubric.max_score,
                "overall_assessment": result.get("overall_assessment"),
                "suggestions": result.get("improvement_suggestions", [])
            }
        except json.JSONDecodeError:
            return 0.0, {"error": "Failed to parse LLM response"}


# ============================================================
# Pass@k and Pass^k Metrics
# ============================================================

def calculate_pass_at_k(results: list[TaskResult], k: int) -> float:
    """
    Calculate pass@k: probability of at least one success in k attempts.

    Formula: 1 - (n-c choose k) / (n choose k)
    Where n = total attempts, c = successful attempts

    For small samples, we use the empirical estimate.
    """
    n = len(results)
    c = sum(1 for r in results if r.passed)

    if n < k:
        # Not enough samples, return empirical rate
        return c / n if n > 0 else 0.0

    # Calculate using combinatorics
    from math import comb

    if c == n:
        return 1.0
    if c == 0:
        return 0.0

    # pass@k = 1 - (ways to choose k failures) / (ways to choose k from all)
    return 1.0 - comb(n - c, k) / comb(n, k)


def calculate_pass_power_k(results: list[TaskResult], k: int) -> float:
    """
    Calculate pass^k: probability that all k attempts succeed.

    This measures reliability - can the AI solve this consistently?

    Formula: p^k where p is the success rate
    """
    if not results:
        return 0.0

    success_rate = sum(1 for r in results if r.passed) / len(results)
    return success_rate ** k


# ============================================================
# Main Evaluator
# ============================================================

class AICodeEvaluator:
    """Main evaluator coordinating all evaluation types."""

    def __init__(
        self,
        api_client=None,
        rubric: EvaluationRubric = DEFAULT_RUBRIC,
        k_values: list[int] = [1, 3, 5]
    ):
        self.api = api_client
        self.rubric = rubric
        self.k_values = k_values

    def evaluate_single(
        self,
        task_id: str,
        code: str,
        requirements: str,
        project_dir: Path,
        attempt: int = 1
    ) -> TaskResult:
        """Evaluate a single code attempt."""
        import time
        start = time.time()
        errors = []

        # Code-based evaluation
        code_evaluator = CodeBasedEvaluator(project_dir)
        code_score, code_details = code_evaluator.evaluate()

        # Model-based evaluation (if API available)
        model_score = None
        model_details = {}
        if self.api:
            model_evaluator = ModelBasedEvaluator(self.api, self.rubric)
            model_score, model_details = model_evaluator.evaluate(code, requirements)

        # Determine pass/fail
        # Pass requires: tests pass AND (no model eval OR model score >= threshold)
        tests_passed = code_details.get("tests", {}).get("passed", False)
        model_passed = model_score is None or model_score >= self.rubric.passing_threshold

        passed = tests_passed and model_passed

        execution_time = int((time.time() - start) * 1000)

        return TaskResult(
            task_id=task_id,
            attempt=attempt,
            passed=passed,
            code_score=code_score,
            model_score=model_score,
            execution_time_ms=execution_time,
            errors=errors,
            details={
                "code_based": code_details,
                "model_based": model_details
            }
        )

    def evaluate_multiple(
        self,
        task_id: str,
        task_description: str,
        code_generator: Callable[[], tuple[str, Path]],
        num_attempts: int = 5
    ) -> EvaluationReport:
        """
        Evaluate multiple attempts of a task.

        Args:
            task_id: Unique identifier for the task
            task_description: Requirements/description
            code_generator: Function that generates (code, project_dir) tuple
            num_attempts: Number of times to attempt the task

        Returns:
            EvaluationReport with all metrics
        """
        results = []

        for attempt in range(1, num_attempts + 1):
            print(f"  Attempt {attempt}/{num_attempts}...")

            code, project_dir = code_generator()
            result = self.evaluate_single(
                task_id=task_id,
                code=code,
                requirements=task_description,
                project_dir=project_dir,
                attempt=attempt
            )
            results.append(result)

        # Calculate metrics
        scores = [r.code_score for r in results]

        return EvaluationReport(
            task_id=task_id,
            task_description=task_description,
            total_attempts=num_attempts,
            results=results,
            pass_at_1=calculate_pass_at_k(results, 1),
            pass_at_k=calculate_pass_at_k(results, max(self.k_values)),
            pass_power_k=calculate_pass_power_k(results, max(self.k_values)),
            mean_score=statistics.mean(scores) if scores else 0,
            std_score=statistics.stdev(scores) if len(scores) > 1 else 0
        )


# ============================================================
# Reporting
# ============================================================

def generate_markdown_report(report: EvaluationReport) -> str:
    """Generate a Markdown report from evaluation results."""
    lines = [
        f"# Evaluation Report: {report.task_id}",
        "",
        f"**Generated:** {report.generated_at.isoformat()}",
        f"**Total Attempts:** {report.total_attempts}",
        "",
        "## Summary Metrics",
        "",
        f"| Metric | Value |",
        f"|--------|-------|",
        f"| pass@1 | {report.pass_at_1:.1%} |",
        f"| pass@{max([1,3,5])} | {report.pass_at_k:.1%} |",
        f"| pass^{max([1,3,5])} | {report.pass_power_k:.1%} |",
        f"| Mean Score | {report.mean_score:.2f} |",
        f"| Std Dev | {report.std_score:.2f} |",
        "",
        "## Interpretation",
        "",
    ]

    # Add interpretation
    if report.pass_power_k >= 0.8:
        lines.append("**Reliability:** EXCELLENT - AI can solve this consistently")
    elif report.pass_power_k >= 0.5:
        lines.append("**Reliability:** GOOD - AI usually succeeds but has failures")
    elif report.pass_at_k >= 0.8:
        lines.append("**Reliability:** MODERATE - AI can solve this but inconsistently")
    else:
        lines.append("**Reliability:** POOR - AI struggles with this task")

    lines.extend([
        "",
        "## Individual Attempts",
        "",
        "| Attempt | Passed | Code Score | Model Score | Time (ms) |",
        "|---------|--------|------------|-------------|-----------|",
    ])

    for r in report.results:
        passed = "Yes" if r.passed else "No"
        model = f"{r.model_score:.2f}" if r.model_score else "N/A"
        lines.append(
            f"| {r.attempt} | {passed} | {r.code_score:.2f} | {model} | {r.execution_time_ms} |"
        )

    return "\n".join(lines)


# ============================================================
# CLI Interface
# ============================================================

def main():
    parser = argparse.ArgumentParser(description="Evaluate AI-generated code")
    parser.add_argument("--task", help="Task definition JSON file")
    parser.add_argument("--benchmark", help="Directory of benchmark tasks")
    parser.add_argument("--runs", type=int, default=5, help="Number of attempts per task")
    parser.add_argument("--output", "-o", help="Output file for results")
    parser.add_argument("--format", choices=["json", "markdown"], default="markdown")

    args = parser.parse_args()

    evaluator = AICodeEvaluator(k_values=[1, 3, 5])

    # Example: evaluate a single task
    if args.task:
        with open(args.task) as f:
            task = json.load(f)

        # This would integrate with your AI code generation
        def generate_code():
            # Your code generation logic here
            pass

        report = evaluator.evaluate_multiple(
            task_id=task["id"],
            task_description=task["description"],
            code_generator=generate_code,
            num_attempts=args.runs
        )

        if args.format == "markdown":
            output = generate_markdown_report(report)
        else:
            output = json.dumps(asdict(report), default=str, indent=2)

        if args.output:
            Path(args.output).write_text(output)
        else:
            print(output)


if __name__ == "__main__":
    main()
```

**Using the Evaluation Framework:**

```bash
# Evaluate a specific task with 5 attempts
python evaluate_ai_code.py --task tasks/user-auth.json --runs 5

# Run benchmark suite
python evaluate_ai_code.py --benchmark benchmarks/ --output report.md

# Get JSON output for CI integration
python evaluate_ai_code.py --task task.json --format json > results.json
```

**Interpreting Results:**

| Metric | Meaning | Good Threshold |
|--------|---------|----------------|
| pass@1 | First-try success rate | > 80% |
| pass@k | Can solve given k tries | > 95% |
| pass^k | Consistent success across k tries | > 60% |
| Mean Score | Average quality score | > 0.8 |

**Key Insight:** For production systems, prioritize pass^k over pass@k. A 95% pass@3 with 50% pass^3 means the AI can eventually solve problems but isn't reliable enough for autonomous operation.

---

## Transcript Analysis

### Why Read Transcripts

Scores alone don't tell the whole story. Reading agent transcripts reveals:

- **How the agent reasoned** about the problem
- **Where it got stuck** and why
- **What tools it used** (or should have used)
- **Whether success was skill or luck**

### What to Look For

```
When analyzing agent transcripts, look for:

1. REASONING QUALITY
   - Did the agent understand the problem correctly?
   - Did it consider alternative approaches?
   - Did it catch and correct its own mistakes?

2. TOOL USAGE
   - Did it use the right tools for the job?
   - Did it gather sufficient context before acting?
   - Did it verify its work?

3. FAILURE PATTERNS
   - Where did it go wrong?
   - Was the failure due to misunderstanding or execution?
   - Could better prompting have prevented the failure?

4. SUCCESS ANALYSIS
   - Did it succeed for the right reasons?
   - Was the solution robust or fragile?
   - Would it generalize to similar problems?
```

### Building Transcript Reviews into Workflow

```
After completing the feature:

1. Export the conversation transcript
2. Review for quality signals:
   - Did the agent explore before implementing?
   - Did it verify its work at each step?
   - Did it handle errors appropriately?

3. Note improvements for future prompts
4. Add any discovered patterns to CLAUDE.md
```

---

## Code Review Checklist

### Before Requesting Review

```markdown
## Pre-Review Checklist

### Functionality
- [ ] All acceptance criteria met
- [ ] Manual testing completed
- [ ] Edge cases handled

### Quality
- [ ] All tests pass
- [ ] No linting errors
- [ ] Type checking passes
- [ ] Security scan clean

### Documentation
- [ ] Code comments where needed
- [ ] API documentation updated
- [ ] README updated if applicable

### Review Readiness
- [ ] Commits are atomic and well-described
- [ ] No debug code left in
- [ ] No TODO/FIXME left unaddressed
- [ ] MRP prepared (for significant changes)
```

### During Review

```markdown
## Review Checklist

### Correctness
- [ ] Does the code do what it claims?
- [ ] Are requirements met?
- [ ] Are edge cases handled?

### Security
- [ ] Input validation present?
- [ ] No SQL injection risks?
- [ ] Authentication/authorization correct?
- [ ] No sensitive data exposed?

### Maintainability
- [ ] Code is readable?
- [ ] Follows project patterns?
- [ ] Appropriate test coverage?
- [ ] No obvious performance issues?

### Integration
- [ ] Works with existing code?
- [ ] No breaking changes?
- [ ] Migrations handled correctly?
```

---

## Key Takeaways

1. **MRPs transform review** from "read everything" to "verify evidence"
2. **Evidence-based review** lets humans focus on judgment, not verification
3. **VCRs capture decisions** for institutional memory and consistency
4. **CRPs structure questions** so agents get the guidance they need efficiently
5. **Generator-Critic pattern** separates creation from validation
6. **Independent verification** prevents context bias
7. **Specialized reviewers** catch domain-specific issues
8. **AI excels at subjective review** (typos, stale comments, naming)
9. **Handle false positives systematically** to maintain review quality
10. **Use pass^k** for measuring reliability in production contexts
11. **Read transcripts** to understand agent behavior beyond scores
12. **Automate MRP generation** to reduce toil and ensure consistency

---

## Practical Exercise: Multi-Agent Review

Implement a multi-agent review workflow:

### Part 1: Generate Code
Ask AI to implement a feature of moderate complexity (e.g., user profile update endpoint).

### Part 2: Generate MRP
Ask AI to create a Merge-Readiness Pack for the implementation.

### Part 3: Independent Review
In a new context (or with `/clear`), provide only:
- The requirements
- The generated code
- Ask for critical review

### Part 4: Specialized Reviews
Run three focused reviews:
1. Security review
2. Performance review
3. Maintainability review

### Part 5: Synthesis
Combine all reviews:
- What issues were found?
- Which were caught by multiple reviewers?
- What would have been missed without multi-agent review?

---

## Next Step

[Step 11: Building Full Applications with AI](11-building-full-applications-with-ai.md) — Learn patterns for building complete applications with AI assistance.

---

## Sources

### Primary References

1. **Anthropic Engineering Blog**
   - "Demystifying Evals for AI Agents" (2025) - https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
   - Evidence-based review approaches for AI-generated code

2. **Academic Papers**
   - Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)
     - Source for MRP, VCR (Version Controlled Resolution), and CRP (Consultation Request Pack) concepts
   - "A Survey on Code Generation with LLM-based Agents." arXiv:2508.00083 (2025)

3. **Stanford CS146S: The Modern Software Developer**
   - Week 7: Modern Software Support
   - Evaluation framework design principles

4. **Industry Resources**
   - Graphite. "AI Code Review Implementation Best Practices" - https://graphite.dev/blog/ai-code-review
   - GitHub Copilot Code Review documentation

---

## Further Reading

1. **"Demystifying Evals for AI Agents"** - Anthropic's guide to designing evaluation frameworks that measure what matters for AI coding assistance.
   https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents

2. **SASE Paper: Structured Agentic Software Engineering** (arXiv:2509.06216v2) - Introduces the MRP, VCR, and CRP concepts that form the foundation of evidence-based code review.
   https://arxiv.org/abs/2509.06216

3. **Google's Code Review Best Practices** - Foundational guidance on code review that applies to both human and AI-generated code.
   https://google.github.io/eng-practices/review/

4. **"Building Effective Agents"** - Anthropic's guide that includes patterns for multi-agent verification.
   https://www.anthropic.com/engineering/building-effective-agents

5. **Hypothesis Documentation** - Property-based testing library useful for writing tests that verify AI-generated code handles edge cases.
   https://hypothesis.readthedocs.io/
