# Step 05: Context Engineering and Documentation

**Level:** Intermediate
**Time:** ~35 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Understand why context is key ("AI cannot infer from omission")
2. Create effective AGENTS.md and CLAUDE.md files
3. Write AI-optimized PRDs with phased instructions
4. Document tech stacks explicitly for consistent AI output

---

In [Step 04](04-effective-prompting-for-code.md), you learned to write specific, structured prompts. But retyping context in every prompt is inefficient. This chapter shows you how to create persistent context files that make every prompt more effective.

## Why Context Engineering Matters

> **"AI cannot infer from omission."**

When you work with a human developer, they bring assumptions from experience. If you forget to mention that you use TypeScript, a senior developer might notice the `.ts` files and figure it out.

AI models don't work this way. They only know what's in their context window. If you don't tell them something, they'll either:
- Make a default assumption (which may be wrong)
- Generate code based on their most common training patterns
- "Hallucinate" details that seem plausible

**Context engineering** is the practice of systematically providing AI with the information it needs to produce good output—without repeating it in every prompt.

### Specs Are the New Source Code

A transformative concept from Stanford CS146S (via Ravi Mehta [^mehta]) is that **specifications are becoming as important as source code itself**. In AI-assisted development:

- **Specifications drive generation:** Well-written specs directly translate into working code
- **Version control applies to specs:** Track PRD changes just like code changes
- **Specs are executable:** With the right AI tools, a spec can be "compiled" into implementation
- **Quality of spec = quality of output:** Vague specs produce inconsistent code; precise specs produce predictable results

This represents a fundamental shift: the primary artifact a developer creates is increasingly the specification, not the implementation. The AI handles the translation from spec to code, while the human ensures the spec captures the true requirements.

**Implications for context engineering:**
- Treat context files as first-class artifacts
- Review and iterate on specs with the same rigor as code reviews
- Build a "spec library" of well-tested patterns you can reuse
- Document not just what to build, but why and how it connects to other parts

---

## The Context File Ecosystem

Modern AI coding tools use several files to persist context:

| File | Purpose | Tools |
|------|---------|-------|
| `AGENTS.md` | Cross-tool agent instructions | Claude Code, Cursor, Copilot, 20+ tools |
| `CLAUDE.md` | Claude-specific project context | Claude Code |
| `.cursorrules` | Cursor-specific rules | Cursor |
| `PRD.md` | Product requirements | Universal (referenced in prompts) |
| `TECH_STACK.md` | Technology documentation | Universal (included in context) |

### Context File Hierarchy

Understanding how context files inherit and override each other is crucial for effective context engineering:

```
┌─────────────────────────────────────────────────────────────────┐
│              CONTEXT FILE HIERARCHY                             │
│                                                                 │
│   ~/.claude/CLAUDE.md              (Global defaults)            │
│          │                                                      │
│          ▼                                                      │
│   /project/AGENTS.md               (Cross-tool, universal)      │
│          │                                                      │
│          ├───► CLAUDE.md           (Claude-specific)            │
│          ├───► .cursorrules        (Cursor-specific)            │
│          └───► GEMINI.md           (Gemini-specific)            │
│          │                                                      │
│          ▼                                                      │
│   /project/src/AGENTS.md           (Subdirectory override)      │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Priority: Subdirectory > Project > Global              │   │
│   │                                                         │   │
│   │  • Global settings provide defaults for all projects    │   │
│   │  • Project-level files override global                  │   │
│   │  • Subdirectory files override parent directories       │   │
│   │  • Tool-specific files (CLAUDE.md) override AGENTS.md   │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### What Goes Where: Context File Decision Matrix

Use this matrix to decide where to put different types of information:

```
┌─────────────────────────────────────────────────────────────────┐
│              WHAT GOES WHERE?                                   │
│                                                                 │
│   Content Type                 │ Recommended File               │
│   ─────────────────────────────┼──────────────────────────────  │
│   Tech stack, versions         │ AGENTS.md / TECH_STACK.md      │
│   Build/test commands          │ AGENTS.md / CLAUDE.md          │
│   Code style rules             │ AGENTS.md                      │
│   Claude-specific behavior     │ CLAUDE.md                      │
│   Cursor-specific rules        │ .cursorrules                   │
│   Team norms/philosophy        │ MENTORSCRIPT.md                │
│   Feature requirements         │ PRD.md                         │
│   API contracts                │ API.md / OpenAPI spec          │
│   Architecture decisions       │ ADR/ directory                 │
│   Global preferences           │ ~/.claude/CLAUDE.md            │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Rule of Thumb:                                         │   │
│   │                                                         │   │
│   │  • Universal info → AGENTS.md (works with all tools)    │   │
│   │  • Tool quirks → Tool-specific file                     │   │
│   │  • Team culture → MENTORSCRIPT.md                       │   │
│   │  • Requirements → PRD.md                                │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### The Key Insight

These files are **prompts that persist across sessions**. Instead of explaining your project in every conversation, you write it once and the AI reads it automatically.

---

## AGENTS.md: The Cross-Tool Standard

### What Is AGENTS.md?

`AGENTS.md` is an emerging standard for providing agent-specific context. It's supported by 20+ AI coding tools and provides a single place to document information that AI needs but humans might find too detailed for a README.

**Official specification:** https://agents.md/

### Where to Place AGENTS.md

- **Repository root:** Main file for the whole project
- **Subdirectories:** Nested files for specific areas (monorepos)
- **Inherited:** AI tools merge parent directory files with child directory files

### What to Include

A good AGENTS.md covers:

1. **Tech Stack** — Languages, frameworks, versions
2. **Setup Commands** — How to install, build, run
3. **Build & Test** — Commands for CI/CD tasks
4. **Code Style** — Patterns, conventions, preferences
5. **Architecture** — File structure, key patterns
6. **Security Considerations** — Things to never do

### Complete AGENTS.md Template

```markdown
# AGENTS.md

## Project Overview
A brief description of what this project does and its main purpose.

## Tech Stack
- **Language:** TypeScript 5.4
- **Runtime:** Node.js 20 LTS
- **Framework:** Next.js 14 (App Router)
- **Database:** PostgreSQL 16 via Prisma
- **Auth:** NextAuth.js with JWT

## Setup Commands
```bash
pnpm install          # Install dependencies
pnpm db:setup         # Initialize database
pnpm dev              # Start development server
```

## Build & Test Commands
```bash
pnpm build           # Production build
pnpm test            # Run test suite
pnpm test:watch      # Run tests in watch mode
pnpm lint            # Run ESLint
pnpm typecheck       # Run TypeScript compiler
```

## Code Style
- Use ES modules (import/export), not CommonJS (require)
- Use functional components with hooks
- Prefer named exports over default exports
- Use async/await over .then() chains
- Follow existing patterns in src/components/

## File Structure
```
src/
├── app/           # Next.js app router pages
├── components/    # React components
├── lib/           # Utility functions
├── services/      # Business logic
└── types/         # TypeScript type definitions
```

## Key Patterns
- Data fetching: Use Server Components with direct DB queries
- Client state: Zustand stores in src/stores/
- API routes: Located in src/app/api/

## Example Files
- Good component example: src/components/UserCard.tsx
- Good service example: src/services/UserService.ts
- Good test example: src/components/__tests__/UserCard.test.tsx

## Files to Avoid
- src/legacy/ - Old code being phased out
- src/utils/deprecated.ts - Use src/lib/ instead

## Security Considerations
- Never commit .env files
- Always validate user input with Zod
- Use parameterized queries (Prisma handles this)
- Don't expose internal IDs in URLs; use UUIDs
```

### Best Practices

1. **Keep it concise** — This goes into every prompt; brevity matters
2. **Include executable commands** — Exact commands, not descriptions
3. **Point to examples** — Reference good files to follow
4. **Call out what to avoid** — Legacy code, deprecated patterns
5. **Update as practices evolve** — Treat it as a living document

---

## CLAUDE.md: Claude-Specific Context

### What Is CLAUDE.md?

`CLAUDE.md` is specific to Claude Code. It's automatically loaded when Claude Code starts and provides project-specific context that shapes Claude's behavior.

### Locations (in priority order)

1. **Repository root:** `./CLAUDE.md`
2. **Parent directories:** `../CLAUDE.md` (useful for monorepos)
3. **Home folder:** `~/.claude/CLAUDE.md` (global defaults)

### The `#` Key Workflow

During a Claude Code session, press `#` to give Claude an instruction that it will automatically incorporate into `CLAUDE.md`. This is powerful for:

- Documenting discovered commands
- Adding style preferences as you work
- Recording project-specific quirks

**Example workflow:**
1. You discover that tests must run with `NODE_ENV=test pnpm test`
2. Press `#` and say "Remember that tests require NODE_ENV=test"
3. Claude adds this to CLAUDE.md
4. Future sessions automatically know this

### CLAUDE.md Template

```markdown
# CLAUDE.md

## Project Context
This is the customer dashboard for Acme Corp. It displays user analytics
and allows configuration of notification preferences.

## Common Commands
- `pnpm dev` - Start dev server (port 3000)
- `pnpm test` - Run tests (needs NODE_ENV=test)
- `pnpm db:migrate` - Run database migrations
- `pnpm db:seed` - Seed development database

## Code Style
- We use Tailwind CSS; don't use inline styles
- All API responses should be typed with Zod schemas
- Use the logger from src/lib/logger.ts (not console.log)

## Testing Requirements
- All new features need unit tests
- Use vitest and testing-library
- Mock external APIs with MSW

## Repository Etiquette
- Branch naming: feature/ABC-123-short-description
- Commits should reference Jira tickets
- Run typecheck before committing

## Quirks and Warnings
- The `UserProfile` component has a known race condition; don't modify
- Database migrations must be backward compatible
- The /api/legacy/* routes are deprecated; redirect to /api/v2/*

## What to Update on Changes
- Update CHANGELOG.md for user-facing changes
- Update API docs in docs/api.md for endpoint changes
```

### Tuning Your CLAUDE.md

Your CLAUDE.md is a prompt. Like any prompt, it benefits from iteration:

1. **Start simple** — Basic commands and structure
2. **Add as you discover** — Use `#` to add guidelines
3. **Remove what doesn't help** — Prune irrelevant content
4. **Emphasize important rules** — Use "IMPORTANT:" or "NEVER:" for emphasis
5. **Run through the prompt improver** — Anthropic's prompt improver can enhance clarity

### Context File Evolution: A Real Example

Context files should evolve as you learn about your project. Here's how a CLAUDE.md typically grows over the first month:

**Week 1: Initial Setup (after /init)**
```markdown
# CLAUDE.md

## Project Overview
E-commerce dashboard built with Next.js and Prisma.

## Commands
- `npm run dev` - Start development server
- `npm test` - Run tests
- `npm run build` - Production build
```

**Week 2: First Discoveries (via # key)**
```markdown
# CLAUDE.md

## Project Overview
E-commerce dashboard built with Next.js and Prisma.

## Commands
- `npm run dev` - Start development server
- `NODE_ENV=test npm test` - Run tests (NODE_ENV required!)
- `npm run build` - Production build
- `npm run db:push` - Push schema changes to dev database

## Discovered Issues
- Tests fail without NODE_ENV=test because config loader checks it
- Must run `db:push` after pulling if schema.prisma changed
```

**Week 3: Pattern Recognition**
```markdown
# CLAUDE.md

## Project Overview
E-commerce dashboard built with Next.js and Prisma.

## Commands
- `npm run dev` - Start development server
- `NODE_ENV=test npm test` - Run tests (NODE_ENV required!)
- `npm run build` - Production build
- `npm run db:push` - Push schema changes to dev database
- `npm run generate` - Regenerate Prisma client after schema changes

## Code Patterns
- API routes use the handler pattern in src/lib/api-handler.ts
- All database queries go through repository classes in src/repositories/
- Use the logger from src/lib/logger.ts, not console.log

## Discovered Issues
- Tests fail without NODE_ENV=test because config loader checks it
- Must run `db:push` after pulling if schema.prisma changed
- The OrderService has a race condition in concurrent updates - use optimistic locking
```

**Week 4+: Mature Context File**
```markdown
# CLAUDE.md

## Project Overview
E-commerce dashboard built with Next.js 14 (App Router) and Prisma 5.
Handles order management, inventory tracking, and analytics for ~500 daily active merchants.

## Commands
- `npm run dev` - Start development server (port 3000)
- `NODE_ENV=test npm test` - Run tests (NODE_ENV required!)
- `npm run test:watch` - Tests in watch mode for TDD
- `npm run build` - Production build
- `npm run db:push` - Push schema changes to dev database
- `npm run db:migrate` - Create migration for production
- `npm run generate` - Regenerate Prisma client after schema changes

## Code Patterns
- API routes use the handler pattern in src/lib/api-handler.ts
- All database queries go through repository classes in src/repositories/
- Use the logger from src/lib/logger.ts, not console.log
- New components go in src/components/{feature}/ not root components/
- Feature flags checked via src/lib/feature-flags.ts

## Testing Patterns
- Unit tests co-located: `Component.test.tsx` next to `Component.tsx`
- Integration tests in `__tests__/integration/`
- Use `createMockUser()` from `test/factories.ts` for test data

## Known Issues
- Tests fail without NODE_ENV=test because config loader checks it
- Must run `db:push` after pulling if schema.prisma changed
- The OrderService has a race condition in concurrent updates - use optimistic locking
- AVOID modifying the LegacyImporter - it's being replaced in Q2

## Security Reminders
- NEVER log full request bodies (may contain PII)
- All user input validated with Zod schemas
- Rate limiting configured in src/middleware/rate-limit.ts

## What to Update When Changing Things
- API changes: Update OpenAPI spec in docs/api.yaml
- New features: Add to CHANGELOG.md
- Database changes: Update seed data in prisma/seed.ts
- Config changes: Update .env.example
```

**Key insight:** The best context files are grown, not written. Start minimal and add discoveries as you work. Use the `#` key aggressively during early sessions.

---

## MentorScript: Codifying Team Knowledge

### What Is MentorScript?

**MentorScript** is the concept of "mentorship-as-code"—encoding the knowledge, norms, and best practices that a senior team member would share with a junior developer into a structured document that both humans and AI agents can follow.

The name comes from the SASE (Structured Agentic Software Engineering) framework and addresses a critical problem: when senior developers leave, their institutional knowledge often leaves with them. MentorScript captures this knowledge in a form that persists and scales.

### Why MentorScript Matters

The following diagram illustrates the key difference between traditional knowledge transfer and the MentorScript approach:

```
┌─────────────────────────────────────────────────────────────────┐
│              MENTORSCRIPT: MENTORSHIP AS CODE                   │
│                                                                 │
│   Traditional Knowledge Transfer:                               │
│   ┌──────────────────┐      ┌──────────────────┐               │
│   │   Senior Dev     │  ──► │   Junior Dev     │               │
│   │   (Knowledge)    │      │   (Learning)     │               │
│   └──────────────────┘      └──────────────────┘               │
│                                                                 │
│   Problem: Knowledge lost when senior leaves                    │
│   Problem: Each new hire starts from scratch                    │
│   Problem: AI agents can't access tribal knowledge              │
│                                                                 │
│   ─────────────────────────────────────────────────────────     │
│                                                                 │
│   With MentorScript:                                            │
│   ┌──────────────────┐      ┌──────────────────┐               │
│   │   Senior Dev     │  ──► │   MentorScript   │               │
│   │   (Knowledge)    │      │   (Codified)     │               │
│   └──────────────────┘      └────────┬─────────┘               │
│                                      │                          │
│                         ┌────────────┴────────────┐             │
│                         │                         │             │
│                         ▼                         ▼             │
│              ┌──────────────────┐     ┌──────────────────┐     │
│              │   Junior Dev     │     │    AI Agent      │     │
│              │   (Learning)     │     │   (Following)    │     │
│              └──────────────────┘     └──────────────────┘     │
│                                                                 │
│   Benefit: Knowledge persists and scales                        │
│   Benefit: Consistent onboarding for humans and AI              │
│   Benefit: Institutional memory survives team changes           │
└─────────────────────────────────────────────────────────────────┘
```

**Traditional knowledge transfer:**
- Senior developer teaches junior developer
- Knowledge exists only in people's heads
- When senior leaves, knowledge is lost
- Each new hire starts from scratch

**With MentorScript:**
- Team norms are documented explicitly
- Both new developers AND AI agents learn from the same source
- Knowledge persists even when team members change
- Onboarding becomes faster and more consistent

### What to Include in MentorScript

A MentorScript document captures the "why" behind decisions and the patterns that make your team's code distinctive:

**Team Philosophy**
- What matters most to your team (performance, readability, extensibility)
- How you approach tradeoffs
- Your stance on technical debt

**Code Standards**
- Not just "what" formatting rules, but "why" you chose them
- Pattern preferences with explanations
- Common mistakes to avoid and why they're problematic

**Review Standards**
- What reviewers should focus on
- What level of test coverage is expected
- When to approve vs. request changes

**Common Mistakes and Lessons Learned**
- Bugs the team has encountered before
- Architectural decisions that didn't work out
- Patterns that seemed good but caused problems

### MentorScript vs. AGENTS.md

| MentorScript | AGENTS.md |
|--------------|-----------|
| Philosophy and reasoning | Technical facts |
| "Why we do things this way" | "What the tech stack is" |
| Cultural norms | Commands and patterns |
| Evolves from team experience | Evolves from project structure |

Many teams combine both concepts in their context files, but keeping the distinction clear helps ensure you capture both the "what" (AGENTS.md) and the "why" (MentorScript).

### Complete MentorScript Template

```markdown
# MENTORSCRIPT.md

## Team Philosophy

### Core Values
We prioritize **correctness over cleverness**. Code should be obvious to the next
person reading it, even if that means being more verbose.

### Tradeoff Decisions
When facing tradeoffs:
- **Performance vs Readability:** Choose readability unless profiling proves otherwise
- **DRY vs Clarity:** Prefer some duplication over obscure abstractions
- **Speed vs Quality:** Ship when confident, not when convenient
- **New Tech vs Proven:** Favor boring technology; new tools need compelling reasons

### Technical Debt Philosophy
We acknowledge all technical debt with `// TODO(name): reason [JIRA-123]`.
Debt is acceptable for shipping, but must be tracked and have a plan.

---

## Code Standards

### Why We Use TypeScript Strict Mode
We enforce `strict: true` because:
- Catches null/undefined bugs at compile time (saved us 3 production incidents in Q2)
- Forces explicit thinking about optionality
- Makes refactoring safer

### Why We Avoid Default Exports
```typescript
// BAD: Default exports
export default function processUser() { }
// Importing allows any name: import foo from './process-user'

// GOOD: Named exports
export function processUser() { }
// Importing requires the real name: import { processUser } from './process-user'
```

Why? Named exports:
- Improve IDE autocomplete and refactoring
- Make imports grep-able
- Prevent naming confusion across files

### Error Handling Philosophy
```typescript
// BAD: Swallowing errors
try {
  await riskyOperation();
} catch (e) {
  console.log('oops');  // Error context lost
}

// GOOD: Preserve context, handle explicitly
try {
  await riskyOperation();
} catch (error) {
  logger.error('riskyOperation failed', { error, userId, operation: 'signup' });
  throw new UserFacingError('Unable to complete signup. Please try again.');
}
```

Why? Silent failures cause the hardest bugs. Every error should either:
1. Be explicitly handled with user feedback, OR
2. Be logged with full context and re-thrown

---

## Review Standards

### What Reviewers Should Focus On
1. **Correctness:** Does this actually solve the problem?
2. **Edge Cases:** What happens with empty arrays, null users, network failures?
3. **Security:** Any user input that isn't validated? Any secrets in code?
4. **Testability:** Can this be tested? Is it tested?
5. **Maintainability:** Will someone understand this in 6 months?

### What Reviewers Should NOT Focus On
- Stylistic preferences (let the linter handle it)
- Alternative implementations that are equally valid
- Hypothetical future requirements

### When to Approve vs Request Changes
- **Approve:** Code works, tests pass, no security concerns, maintainable
- **Comment (non-blocking):** Suggestions for improvement, not required
- **Request Changes:** Bug found, security issue, missing tests for critical path

---

## Common Mistakes to Avoid

### Mistake 1: The N+1 Query Problem
We've shipped N+1 queries to production 4 times. Now we check:
```typescript
// BAD: N+1 queries
const users = await db.user.findMany();
for (const user of users) {
  user.posts = await db.post.findMany({ where: { authorId: user.id } });
}

// GOOD: Single query with include
const users = await db.user.findMany({
  include: { posts: true }
});
```

### Mistake 2: Not Handling Loading/Error States
Every async operation needs three states:
```typescript
// Component must handle: loading, error, success
function UserProfile({ userId }: Props) {
  const { data, error, isLoading } = useQuery(['user', userId], fetchUser);

  if (isLoading) return <Skeleton />;
  if (error) return <ErrorMessage error={error} />;
  return <Profile user={data} />;
}
```

### Mistake 3: Forgetting Database Migrations Are Irreversible in Production
Once a migration runs in production:
- You CANNOT edit it
- You CANNOT delete it
- You MUST create a new migration to make changes

Triple-check migrations before merging.

---

## Pattern Preferences

### Data Fetching: Server Components First
```typescript
// PREFERRED: Fetch in Server Component
async function UserPage({ params }: Props) {
  const user = await db.user.findUnique({ where: { id: params.id } });
  return <UserProfile user={user} />;  // Pass data to client component
}

// AVOID: Client-side fetching when server would work
'use client';
function UserPage({ params }: Props) {
  const { data: user } = useQuery(...);  // Extra round trip
  return <UserProfile user={user} />;
}
```

Why? Server components:
- Reduce client bundle size
- Eliminate loading spinners for initial data
- Simplify caching
- Improve security (db calls stay on server)

### State Management: Local First
```typescript
// Start with local state
const [isOpen, setIsOpen] = useState(false);

// Lift state only when needed
// <Parent> manages state that <Child1> and <Child2> both need

// Use Zustand only for truly global state
// Authentication, shopping cart, theme preference

// AVOID: Putting everything in global state "for convenience"
```

---

## Lessons From Production Incidents

### Incident: Search Indexing Timeout (Sept 2024)
**What happened:** Search reindex job timed out, left index in corrupted state.
**Root cause:** No transaction wrapping, partial failure not handled.
**Lesson:** All multi-step operations must be atomic or have rollback plans.
**Pattern to follow:** See `src/jobs/safe-reindex.ts` for the correct approach.

### Incident: Memory Leak in WebSocket Handler (Oct 2024)
**What happened:** Server OOM after 3 days uptime.
**Root cause:** Event listeners not cleaned up on connection close.
**Lesson:** Every `addEventListener` or `subscribe` needs corresponding cleanup.
**Pattern to follow:** See `src/realtime/connection-manager.ts` for cleanup pattern.
```

This template captures the institutional knowledge that would otherwise only exist in senior developers' heads.

### Institutional Memory

MentorScript is part of building **institutional memory**—the collective knowledge that makes experienced teams effective. In AI-assisted development, institutional memory becomes even more valuable because:

1. **AI can apply it consistently:** Once documented, AI follows the norms without forgetting
2. **It scales without meetings:** New team members (human or AI) learn from the same source
3. **It improves over time:** Each learned lesson can be added to the knowledge base
4. **It survives transitions:** When people change roles, the knowledge remains

---

## The `/init` Command

Claude Code's `/init` command automatically generates a `CLAUDE.md` file by analyzing your project. This gives you a starting point that you can then customize.

**Usage:**
```bash
claude
> /init
```

Claude will:
1. Analyze your project structure
2. Detect your tech stack
3. Find build and test commands
4. Generate a draft CLAUDE.md

Review and customize the output—the auto-generated file is a starting point, not a finished product.

---

## PRDs for AI-Assisted Development

### How PRDs Have Evolved

Traditional PRDs dealt with human-to-human communication, where context could be inferred. AI-optimized PRDs must be:

1. **Explicit about boundaries** — AI cannot infer from omission
2. **Dependency-ordered** — Build foundations before advanced features
3. **Phased** — 150-200 instructions maximum per phase (based on HumanLayer research)
4. **Behavior-focused** — Measurable behaviors, not adjectives

### Bad vs. Good Requirements

| Bad (Vague) | Good (Specific) |
|-------------|-----------------|
| "The app should be user-friendly" | "User can add an invoice in 3 clicks or fewer" |
| "Fast performance" | "Page load time under 2 seconds on 3G connection" |
| "Secure authentication" | "Password requires 12+ chars, upper, lower, number, symbol" |
| "Handle errors gracefully" | "Show toast with retry button; log error to Sentry" |

### The 5-Phase PRD Structure

Research from HumanLayer suggests that AI agents work best with 150-200 instructions per phase [^humanlayer]. Structure your PRD in phases:

```
┌─────────────────────────────────────────────────────────────────┐
│              5-PHASE PRD WORKFLOW                               │
│                                                                 │
│   Phase 1: Foundation    ────────────────────────────────────►  │
│   (~40 requirements)     [Data models, Auth, Setup]             │
│                               │                                 │
│                               ▼ Checkpoint: Tests pass?         │
│                                                                 │
│   Phase 2: Core Features ────────────────────────────────────►  │
│   (~50 requirements)     [Main functionality, User flows]       │
│                               │                                 │
│                               ▼ Checkpoint: Features work?      │
│                                                                 │
│   Phase 3: Integration   ────────────────────────────────────►  │
│   (~40 requirements)     [External services, APIs]              │
│                               │                                 │
│                               ▼ Checkpoint: Integration OK?     │
│                                                                 │
│   Phase 4: Polish        ────────────────────────────────────►  │
│   (~40 requirements)     [UX, Edge cases, Error states]         │
│                               │                                 │
│                               ▼ Checkpoint: Quality OK?         │
│                                                                 │
│   Phase 5: Production    ────────────────────────────────────►  │
│   (~30 requirements)     [Security, Performance, Monitoring]    │
│                               │                                 │
│                               ▼ DEPLOY                          │
│                                                                 │
│   ┌─────────────────────────────────────────────────────────┐   │
│   │  Key Principles:                                        │   │
│   │                                                         │   │
│   │  • 150-200 requirements max per phase                   │   │
│   │  • Checkpoint between each phase (human review)         │   │
│   │  • Dependencies flow forward (Phase 2 needs Phase 1)    │   │
│   │  • Each phase produces testable, working software       │   │
│   │  • Don't proceed until checkpoint passes                │   │
│   └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

**Phase 1: Foundation (~40 requirements)**
- Core data models
- Database schema
- Basic project structure
- Authentication skeleton

**Phase 2: Core Features (~50 requirements)**
- Primary user flows
- Main functionality
- Essential API endpoints

**Phase 3: Integration (~40 requirements)**
- External service connections
- Third-party APIs
- Webhooks and events

**Phase 4: Polish (~40 requirements)**
- UX improvements
- Edge case handling
- Error states
- Loading states

**Phase 5: Production (~30 requirements)**
- Security hardening
- Performance optimization
- Monitoring and logging
- Documentation

### PRD Template

```markdown
# PRD: [Feature Name]

## Overview
[One paragraph describing the feature and its purpose]

## Success Metrics
- [ ] [Measurable outcome 1]
- [ ] [Measurable outcome 2]

---

## Phase 1: Foundation

### Data Models
1. Create User model with fields: id, email, password_hash, created_at
2. Create Invoice model with fields: id, user_id, amount, status, due_date
3. Add foreign key relationship from Invoice to User

### Database
4. Create migration for users table
5. Create migration for invoices table
6. Add indexes on frequently queried columns

[Continue with numbered requirements...]

---

## Phase 2: Core Features

### User Authentication
20. Implement login endpoint POST /api/auth/login
21. Accept JSON body with email and password
22. Return JWT token on success
23. Return 401 with error message on failure

[Continue...]

---

## Phase 3: Integration
[...]

## Phase 4: Polish
[...]

## Phase 5: Production
[...]
```

### Referencing PRDs in Prompts

```
Following the requirements in PRD.md Phase 2, implement the user authentication
endpoints. Start with requirement #20 (login endpoint) and continue through #25.
```

### Complete 5-Phase PRD Example

Here's a full PRD demonstrating the phased approach with specific, measurable requirements:

```markdown
# PRD: Task Management System

## Overview
A task management application for small teams (5-20 people) to track work items,
assign tasks, and monitor project progress. Built as a web app with real-time updates.

## Success Metrics
- [ ] Team can create, assign, and complete tasks in under 30 seconds each
- [ ] Real-time updates visible to all team members within 2 seconds
- [ ] System handles 100 concurrent users without degradation
- [ ] 95% of user actions complete without errors

---

## Phase 1: Foundation (Requirements 1-40)

### 1.1 Data Models

1. Create User model with fields:
   - id: UUID (primary key)
   - email: string (unique, required)
   - password_hash: string (required)
   - name: string (required)
   - avatar_url: string (optional)
   - created_at: timestamp
   - updated_at: timestamp

2. Create Team model with fields:
   - id: UUID (primary key)
   - name: string (required)
   - slug: string (unique, for URLs)
   - created_at: timestamp

3. Create TeamMembership model (join table):
   - user_id: UUID (foreign key to User)
   - team_id: UUID (foreign key to Team)
   - role: enum ('owner', 'admin', 'member')
   - joined_at: timestamp

4. Create Project model with fields:
   - id: UUID (primary key)
   - team_id: UUID (foreign key to Team)
   - name: string (required)
   - description: text (optional)
   - status: enum ('active', 'archived')
   - created_at: timestamp

5. Create Task model with fields:
   - id: UUID (primary key)
   - project_id: UUID (foreign key to Project)
   - title: string (required, max 200 chars)
   - description: text (optional)
   - status: enum ('todo', 'in_progress', 'done')
   - priority: enum ('low', 'medium', 'high', 'urgent')
   - assignee_id: UUID (foreign key to User, optional)
   - due_date: date (optional)
   - created_by_id: UUID (foreign key to User)
   - created_at: timestamp
   - updated_at: timestamp

6. Add index on Task.project_id for project task listings
7. Add index on Task.assignee_id for user task queries
8. Add index on Task.status for filtered views
9. Add compound index on (project_id, status) for kanban boards

### 1.2 Database Setup

10. Create Prisma schema with all models defined above
11. Generate initial migration: `create_initial_tables`
12. Create seed script with:
    - 3 test users (alice@test.com, bob@test.com, charlie@test.com)
    - 1 test team ("Acme Corp")
    - 2 test projects ("Website Redesign", "Mobile App")
    - 10 sample tasks across projects
13. Document DATABASE_URL format in .env.example

### 1.3 Authentication

14. Implement POST /api/auth/register endpoint:
    - Accept: { email, password, name }
    - Validate email format with Zod
    - Validate password: min 8 chars, at least 1 number
    - Hash password with bcrypt (cost factor 12)
    - Return: { user: { id, email, name }, token }

15. Implement POST /api/auth/login endpoint:
    - Accept: { email, password }
    - Validate credentials against database
    - Return 401 with { error: "Invalid credentials" } on failure
    - Return: { user: { id, email, name }, token } on success

16. Implement JWT token generation:
    - Include user id and email in payload
    - Set expiration to 7 days
    - Sign with secret from environment variable

17. Implement auth middleware:
    - Extract token from Authorization header (Bearer format)
    - Validate token signature and expiration
    - Attach user to request context
    - Return 401 if token invalid or missing

18. Implement GET /api/auth/me endpoint:
    - Requires authentication
    - Return current user details

19. Create auth utilities in src/lib/auth.ts:
    - hashPassword(password: string): Promise<string>
    - verifyPassword(password: string, hash: string): Promise<boolean>
    - generateToken(user: User): string
    - verifyToken(token: string): TokenPayload | null

### 1.4 Basic Project Structure

20. Set up Next.js 14 with App Router
21. Configure TypeScript with strict mode
22. Set up Tailwind CSS
23. Create base layout with navigation placeholder
24. Create src/lib/ directory for utilities
25. Create src/components/ui/ for shared components
26. Set up Vitest for testing
27. Configure path aliases: @/ for src/

### 1.5 Core UI Components

28. Create Button component with variants: primary, secondary, danger, ghost
29. Create Input component with label, error state, and helper text
30. Create Card component for content containers
31. Create Modal component with close button and backdrop
32. Create Avatar component (image with fallback to initials)
33. Create Badge component for status indicators
34. Create Loading spinner component
35. Create Empty state component with icon, title, description, action

### 1.6 Phase 1 Checkpoint

36. All tests pass for auth endpoints
37. Database seed runs successfully
38. Can register new user via API
39. Can login and receive valid JWT
40. Protected endpoints reject unauthenticated requests

---

## Phase 2: Core Features (Requirements 41-95)

### 2.1 Team Management

41. Implement POST /api/teams endpoint:
    - Create team with current user as owner
    - Generate unique slug from team name

42. Implement GET /api/teams endpoint:
    - Return all teams user belongs to
    - Include user's role in each team

43. Implement GET /api/teams/:teamId endpoint:
    - Return team details with member count
    - Return 403 if user not a member

44. Implement POST /api/teams/:teamId/invite endpoint:
    - Accept: { email, role }
    - Only owners/admins can invite
    - Create pending invitation (implement in Phase 3)

45. Create team switcher UI component
46. Create team settings page shell

### 2.2 Project Management

47. Implement POST /api/teams/:teamId/projects endpoint:
    - Accept: { name, description }
    - Validate user is team member

48. Implement GET /api/teams/:teamId/projects endpoint:
    - Return all projects for team
    - Include task counts per status

49. Implement GET /api/projects/:projectId endpoint:
    - Return project details
    - Verify user has access via team membership

50. Implement PATCH /api/projects/:projectId endpoint:
    - Accept: { name?, description?, status? }
    - Only allow archiving if no incomplete tasks

51. Create project list view with cards
52. Create project creation modal
53. Create project header with name and description

### 2.3 Task CRUD

54. Implement POST /api/projects/:projectId/tasks endpoint:
    - Accept: { title, description?, priority, assignee_id?, due_date? }
    - Validate assignee is team member if provided
    - Set created_by to current user

55. Implement GET /api/projects/:projectId/tasks endpoint:
    - Return all tasks for project
    - Support query params: status, assignee_id, priority
    - Include assignee details in response

56. Implement GET /api/tasks/:taskId endpoint:
    - Return full task details with assignee and creator info

57. Implement PATCH /api/tasks/:taskId endpoint:
    - Accept any task field for update
    - Validate assignee if changed
    - Update updated_at timestamp

58. Implement DELETE /api/tasks/:taskId endpoint:
    - Soft delete (set deleted_at) or hard delete based on project setting
    - Return 204 on success

### 2.4 Kanban Board View

59. Create Kanban board component with columns: Todo, In Progress, Done
60. Display tasks as cards within columns
61. Show task: title, priority badge, assignee avatar, due date
62. Implement drag-and-drop between columns (update status)
63. Add loading skeletons while fetching
64. Add empty column message: "No tasks in this column"
65. Persist column order in local storage

### 2.5 Task Detail View

66. Create task detail panel (slide-over or modal)
67. Display all task fields
68. Allow inline editing of title (click to edit)
69. Dropdown for status change
70. Dropdown for priority change
71. User picker for assignee
72. Date picker for due date
73. Textarea for description
74. Show created by and created at
75. Show last updated timestamp

### 2.6 Task List View

76. Create list view as alternative to Kanban
77. Sortable columns: title, status, priority, assignee, due date
78. Click row to open task detail
79. Checkbox for bulk selection
80. Bulk actions: change status, change assignee, delete
81. View toggle between List and Kanban

### 2.7 Task Filtering

82. Filter by status (multi-select)
83. Filter by priority (multi-select)
84. Filter by assignee (multi-select)
85. Filter by due date range
86. "My Tasks" quick filter (assigned to current user)
87. "Overdue" quick filter
88. Clear all filters button
89. Save filter as view (store in local storage)

### 2.8 Phase 2 Checkpoint

90. Can create teams and projects via UI
91. Can create, edit, delete tasks
92. Kanban drag-and-drop updates task status
93. Filtering works correctly
94. All CRUD operations have tests
95. No N+1 queries in list endpoints

---

## Phase 3: Integration (Requirements 96-140)

### 3.1 Real-Time Updates

96. Set up WebSocket server using Socket.io
97. Authenticate WebSocket connections with JWT
98. Create room per project for scoped updates
99. Broadcast task.created event when task created
100. Broadcast task.updated event when task modified
101. Broadcast task.deleted event when task removed
102. Client subscribes to project room on mount
103. Client updates local state on received events
104. Handle reconnection gracefully
105. Show "Reconnecting..." indicator when disconnected

### 3.2 Email Notifications

106. Set up email service (Resend or SendGrid)
107. Create email templates directory with MJML
108. Create "task assigned" email template
109. Create "task due soon" email template
110. Create "mentioned in comment" email template (for Phase 4)
111. Queue emails with background job processor
112. Implement rate limiting: max 10 emails per user per hour
113. Add unsubscribe link to all emails
114. Log email sends for debugging

### 3.3 Team Invitations

115. Create Invitation model:
    - id, team_id, email, role, token, expires_at, accepted_at
116. Send invitation email with unique accept link
117. Implement GET /api/invitations/:token endpoint (validate invitation)
118. Implement POST /api/invitations/:token/accept endpoint
119. Create invitation acceptance page
120. Handle case: invited user doesn't have account (create during accept)
121. Handle case: invited user already has account (link to team)
122. Expire invitations after 7 days

### 3.4 User Settings

123. Create user settings page
124. Implement name change
125. Implement avatar upload (store in S3 or Cloudflare R2)
126. Implement email change (requires verification)
127. Implement password change (requires current password)
128. Implement notification preferences:
    - Email on task assignment (on/off)
    - Email on due date approaching (on/off)
    - Email digest frequency (immediate/daily/weekly)

### 3.5 Search

129. Implement GET /api/search endpoint
130. Search tasks by title and description
131. Search across all user's teams (with permission check)
132. Return results with highlighted matches
133. Limit to 50 results
134. Create search UI with keyboard shortcut (Cmd+K)
135. Show recent searches

### 3.6 Phase 3 Checkpoint

136. Real-time updates work across browser tabs
137. Email notifications send correctly
138. Team invitations work end-to-end
139. Search finds relevant results
140. No security issues in invitation flow

---

## Phase 4: Polish (Requirements 141-180)

### 4.1 Comments & Activity

141. Create Comment model: id, task_id, user_id, content, created_at
142. Implement POST /api/tasks/:taskId/comments
143. Implement GET /api/tasks/:taskId/comments
144. Display comments in task detail view
145. Add comment input with submit button
146. Create Activity model for task history
147. Track: created, status change, assignee change, priority change
148. Display activity feed in task detail
149. Interleave comments and activity chronologically

### 4.2 @Mentions

150. Implement @mention detection in comment text
151. Autocomplete user names when typing @
152. Parse mentions and create notification
153. Link mentions to user profile
154. Highlight mentions in rendered comments

### 4.3 Due Date Features

155. Calculate "due soon" (within 3 days)
156. Calculate "overdue" (past due date)
157. Visual indicators: yellow for due soon, red for overdue
158. Sort by due date option
159. Calendar view for tasks with due dates
160. "Due today" count in navigation

### 4.4 Keyboard Navigation

161. Cmd+K: Open search
162. Cmd+N: Create new task
163. J/K: Navigate up/down in task list
164. Enter: Open selected task
165. Escape: Close modal/panel
166. Tab navigation through form fields
167. Focus trap in modals

### 4.5 Mobile Responsiveness

168. Responsive navigation (hamburger menu on mobile)
169. Kanban board horizontal scroll on mobile
170. Task cards stack properly on small screens
171. Touch-friendly button sizes (min 44x44px)
172. Swipe gestures for common actions

### 4.6 Error Handling & Loading States

173. Global error boundary with friendly message
174. Toast notifications for success/error
175. Skeleton loaders for all async content
176. Retry buttons on failed requests
177. Offline indicator when connection lost
178. Optimistic updates for fast UI feedback

### 4.7 Phase 4 Checkpoint

179. Comments and activity work correctly
180. Due date indicators display properly
181. Keyboard navigation works throughout
182. Mobile experience is usable
183. Error states handled gracefully

---

## Phase 5: Production (Requirements 184-200)

### 5.1 Security Hardening

184. Rate limit all API endpoints
185. Add CORS configuration for production domain
186. Implement CSRF protection
187. Security headers (CSP, HSTS, etc.)
188. Audit logging for sensitive operations
189. Input sanitization review

### 5.2 Performance

190. Database query analysis and index optimization
191. API response caching where appropriate
192. Image optimization for avatars
193. Code splitting for large pages
194. Lighthouse score > 90

### 5.3 Monitoring & Observability

195. Error tracking with Sentry
196. Application metrics (request duration, error rate)
197. Database query monitoring
198. Uptime monitoring with alerts

### 5.4 Documentation

199. API documentation (OpenAPI/Swagger)
200. Deployment guide
201. Environment variable documentation
202. Architecture decision records

### 5.5 Production Checkpoint

203. Security review passed
204. Performance benchmarks met
205. Monitoring dashboards operational
206. Documentation complete
207. Ready for production deployment
```

**How to use this PRD:**

1. **Give Claude the full PRD** at project start for context
2. **Reference phases in prompts:** "Implement Phase 1.3 Authentication (requirements 14-19)"
3. **Use checkpoints as gates:** Don't proceed to Phase 2 until Phase 1 checkpoint passes
4. **Update as you learn:** Add requirements discovered during implementation

---

## Tech Stack Documentation

### Why Document Your Tech Stack?

AI models make assumptions based on training data. Without explicit documentation:

- Might use wrong versions of libraries
- Might suggest deprecated patterns
- Might mix incompatible technologies
- Might hallucinate dependencies

### TECH_STACK.md Template

```markdown
# Technology Stack

## Core Technologies
- **Language:** TypeScript 5.4
- **Runtime:** Node.js 20.11.0 LTS
- **Package Manager:** pnpm 8.15.0

## Frontend
- **Framework:** React 18.2
- **Meta-framework:** Next.js 14.1 (App Router)
- **Styling:** Tailwind CSS 3.4
- **State Management:** Zustand 4.5

## Backend
- **API Style:** REST with tRPC for typed endpoints
- **Validation:** Zod 3.22
- **ORM:** Prisma 5.9
- **Database:** PostgreSQL 16

## Authentication
- **Library:** NextAuth.js 5.0
- **Strategy:** JWT with refresh tokens
- **Session:** Server-side sessions in database

## Testing
- **Unit Tests:** Vitest 1.2
- **Component Tests:** Testing Library 14
- **E2E Tests:** Playwright 1.41
- **API Mocking:** MSW 2.1

## Infrastructure
- **Hosting:** Vercel
- **Database:** Neon (serverless Postgres)
- **CDN:** Vercel Edge
- **Monitoring:** Sentry

## Version Constraints
- Node.js >= 20.0.0
- TypeScript >= 5.3.0
- React >= 18.2.0

## Important Patterns
- Use Server Components by default; add "use client" only when needed
- All API responses validated with Zod schemas
- Database queries only in Server Components or API routes
- Never import server-only code in client components
```

---

## Context Files for Monorepos

In monorepos, use nested context files:

```
my-monorepo/
├── AGENTS.md           # Shared across all packages
├── CLAUDE.md           # Global Claude instructions
├── packages/
│   ├── web/
│   │   ├── AGENTS.md   # Web-specific context
│   │   └── CLAUDE.md   # Web-specific Claude instructions
│   ├── api/
│   │   ├── AGENTS.md   # API-specific context
│   │   └── CLAUDE.md   # API-specific Claude instructions
│   └── shared/
│       └── AGENTS.md   # Shared library context
```

AI tools will merge parent and child context files, with child files taking precedence for conflicts.

---

## Context Debugging: When Things Go Wrong

### Symptoms of Context Problems

Context-related issues manifest in recognizable ways:

**Symptom: AI ignores your instructions**
- Cause: Instructions buried too deep in context, or conflicting instructions
- Check: Is the relevant AGENTS.md or CLAUDE.md being loaded? Are there conflicts?

**Symptom: AI uses wrong patterns or outdated approaches**
- Cause: Context files haven't been updated, or AI training data is overriding your specs
- Check: Are your pattern preferences explicitly documented? Are versions specified?

**Symptom: AI seems confused or gives inconsistent responses**
- Cause: Context window is full of contradictory information from a long session
- Check: When did you last use `/clear` or `/compact`?

**Symptom: AI hallucinates file paths or dependencies**
- Cause: Insufficient context about project structure
- Check: Does AGENTS.md include file structure documentation?

### Diagnostic Strategies

**Strategy 1: Check context loading**
Ask the AI directly what context it has:
- "What do you know about this project's tech stack?"
- "What testing framework should we use here?"
- "What are the code style rules for this project?"

If answers don't match your context files, investigate why they're not being loaded.

**Strategy 2: Context audit**
Review what's actually in the context window:
- Are the right files being included?
- Are they in the right priority order?
- Are there conflicting instructions?

**Strategy 3: Isolation testing**
Start a fresh session (`/clear`) and provide minimal context:
- Does the problem reproduce?
- At what point does context become corrupted?

**Strategy 4: Version mismatch check**
If AI suggests deprecated patterns:
- Are versions explicitly specified in your context files?
- Is the AI's knowledge cutoff more recent than your documented patterns?

### Fixing Context Problems

**Problem: Context not loading**
- Verify file names are correct (CLAUDE.md, not claude.md on case-sensitive systems)
- Check file is in the right directory
- Restart Claude Code to force re-read

**Problem: Context too verbose**
- Consolidate redundant information
- Move detailed specs to linked files
- Keep AGENTS.md/CLAUDE.md focused on essentials

**Problem: Context conflicts**
- Identify which files have conflicting instructions
- Establish clear hierarchy (subdirectory overrides parent)
- Remove or reconcile duplicated guidance

**Problem: Context becomes stale**
- Schedule regular context file reviews
- Add context file updates to your PR checklist
- Use the `#` key to capture discoveries in real-time

### How Long Contexts Fail

Research shows that long context windows don't eliminate the need for good context engineering. Key findings from Drew Breunig [^breunig] and others:

- **Attention degradation:** Important information in the middle of very long contexts gets less attention than information at the beginning or end
- **Conflicting signals:** More context means more opportunity for contradictory information
- **Recency bias:** Information added later in a session may override earlier instructions
- **Signal-to-noise:** Useful context diluted by irrelevant history

The solution isn't longer context windows—it's better context curation.

---

## A2A Protocol: Agent-to-Agent Communication (Preview)

### What Is A2A?

The **Agent-to-Agent (A2A) Protocol** is an emerging standard for how AI agents communicate with each other. While MCP (Model Context Protocol) handles agent-to-tool communication, A2A handles agent-to-agent communication in multi-agent systems.

### Why A2A Matters for Context Engineering

In multi-agent workflows, context becomes more complex:
- **Shared context:** What should all agents know?
- **Agent-specific context:** What's relevant only to certain agents?
- **Handoff context:** What information transfers between agents?
- **Audit trails:** How do we track what each agent did and why?

A2A addresses these challenges by standardizing:
- How agents describe tasks to each other
- How agents share artifacts (code, tests, documentation)
- How agents report results and evidence
- How orchestrators coordinate multiple agents

### A2A and Your Context Files

As multi-agent systems become more common, your context engineering will evolve:

1. **Universal context:** In AGENTS.md, document what all agents need
2. **Agent-specific context:** Create specialized context files for different agent roles
3. **Protocol awareness:** Understand how agents will communicate to structure context appropriately
4. **Evidence requirements:** Document what verification evidence each type of work requires

This is a preview of concepts covered in depth in Chapter 08 (Agent Architecture and Patterns) and Chapter 12 (Future Directions).

---

## Key Takeaways

1. **AI cannot infer from omission** — Explicitly document what matters
2. **Specs are the new source code** — Treat specifications as first-class artifacts
3. **Use AGENTS.md** for cross-tool context (stack, commands, patterns)
4. **Use CLAUDE.md** for Claude-specific context (style, quirks, workflows)
5. **Create MentorScript** for team norms and institutional knowledge
6. **Press `#`** to add discoveries to CLAUDE.md during sessions
7. **Structure PRDs in phases** — 150-200 requirements per phase
8. **Document your tech stack** — Prevent version and pattern mismatches
9. **Nest files for monorepos** — Specific context inherits from general
10. **Debug context problems systematically** — Check loading, conflicts, and staleness
11. **Understand A2A basics** — Multi-agent context needs are evolving

---

## Practical Exercise: Create Your Context Files

For a project you're working on (or a hypothetical one):

### Part 1: Create AGENTS.md
1. Document the tech stack
2. List setup and build commands
3. Define code style preferences
4. Point to example files

### Part 2: Create CLAUDE.md
1. Write project context
2. Add common commands
3. Note any quirks or warnings
4. Define what to update when making changes

### Part 3: Test Your Context
1. Start a new Claude Code session in your project
2. Ask Claude about your project: "What tech stack does this project use?"
3. Verify that Claude picks up the context from your files
4. Iterate on the files based on gaps you discover

---

## Next Step

[Step 06: Explore-Plan-Code-Commit Workflow](06-explore-plan-code-commit-workflow.md) — Master Anthropic's recommended 4-phase workflow for AI-assisted development.

---

## Sources

[^mehta]: Mehta, R. (2025). "Specs Are the New Source Code." Stanford CS146S Guest Lecture, Week 3. Stanford University.

[^humanlayer]: HumanLayer. (2025). "Writing a Good CLAUDE.md" and "Getting AI to Work in Complex Codebases." Research on instruction limits and context optimization. https://humanlayer.dev/blog

[^breunig]: Breunig, D. (2024). "How Long Contexts Fail." Research notes on attention degradation in long context windows. https://www.dbreunig.com/2024/03/14/how-long-contexts-fail.html

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. https://arxiv.org/abs/2509.06216

- AGENTS.md Official Specification. https://agents.md/
- Anthropic: "Claude Code Best Practices." https://docs.anthropic.com/en/docs/claude-code/best-practices
- Stanford CS146S, Week 3: PRDs for Agents. Stanford University, 2025.
- Builder.io: "AGENTS.md Best Tips." https://www.builder.io/blog/agents-md

---

## Further Reading

1. **AGENTS.md Official Specification** — The emerging standard for cross-tool agent context files. https://agents.md/

2. **Anthropic: Claude Code Best Practices** — Official guide including context file recommendations. https://docs.anthropic.com/en/docs/claude-code/best-practices

3. **HumanLayer: Writing a Good CLAUDE.md** — Practical guidance on context file creation and maintenance. https://humanlayer.dev/blog/writing-good-claude-md

4. **Drew Breunig: How Long Contexts Fail** — Deep dive into attention degradation and context rot. https://www.dbreunig.com/2024/03/14/how-long-contexts-fail.html

5. **SASE Paper (arXiv:2509.06216v2)** — Academic framework including MentorScript and context engineering concepts. https://arxiv.org/abs/2509.06216
