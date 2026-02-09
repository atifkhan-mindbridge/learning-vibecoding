# Step 13: Team Workflows and Governance

**Level:** Advanced
**Time:** ~30 minutes

In [Step 12](12-Production-Deployment-and-Operations.md), you learned to deploy and operate AI-assisted applications in production. This step scales those practices to team and organizational levels, addressing the governance and coordination challenges that emerge when multiple people work with AI.

## Learning Objectives

By the end of this step, you will be able to:

1. Establish team AI development standards
2. Create and manage shared context files
3. Implement governance frameworks
4. Track AI contribution metrics

---

## The Team Challenge

Individual AI-assisted development is one thing. Team-scale adoption requires:

1. **Consistency:** All team members follow the same patterns
2. **Quality:** Standards prevent "race to the bottom"
3. **Accountability:** Clear ownership of AI-generated code
4. **Visibility:** Understanding what AI is contributing

---

## The SE4H/SE4A Duality

Modern AI-assisted teams operate in two modes:

### Software Engineering for Humans (SE4H)

Humans become **Agent Coaches** focusing on:
- Defining intent and strategy
- Providing oversight and guidance
- Making judgment calls
- Mentoring AI (through context files)
- Reviewing and approving work

### Software Engineering for Agents (SE4A)

Engineering the environment where agents operate:
- Tool configuration
- Context file management
- Workflow automation
- Governance enforcement
- Quality gates

```
┌─────────────────────────────────────────────────────────────┐
│                     Team Structure                          │
│                                                             │
│  ┌─────────────────────┐     ┌─────────────────────┐       │
│  │    SE4H (Human)     │     │    SE4A (Agent)     │       │
│  │                     │     │                     │       │
│  │  • Define specs     │ ←→  │  • Execute tasks    │       │
│  │  • Review output    │     │  • Generate code    │       │
│  │  • Make decisions   │     │  • Run tests        │       │
│  │  • Coach agents     │     │  • Create artifacts │       │
│  │                     │     │                     │       │
│  └─────────────────────┘     └─────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## N-to-N Collaboration

Traditional AI assistance pairs one human with one AI assistant. As teams scale, a more sophisticated model emerges: **N-to-N collaboration**, where multiple humans coordinate with multiple AI agents.

### N-to-N Collaboration Model

The following diagram illustrates the N-to-N collaboration architecture:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       N-TO-N COLLABORATION MODEL                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  HUMAN LAYER (N humans)                                                      │
│  ──────────────────────                                                     │
│                                                                              │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │   TECH     │  │  PRODUCT   │  │   SENIOR   │  │   JUNIOR   │            │
│  │   LEAD     │  │  MANAGER   │  │   DEV      │  │   DEV      │            │
│  │            │  │            │  │            │  │            │            │
│  │ Strategy   │  │ Priorities │  │ Review     │  │ Features   │            │
│  │ Architecture│ │ Specs      │  │ Mentoring  │  │ Learning   │            │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘            │
│        │               │               │               │                    │
│        └───────────────┼───────────────┼───────────────┘                    │
│                        │               │                                    │
│                        ▼               ▼                                    │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │                  COORDINATION LAYER                                   │  │
│  │                                                                       │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐ │  │
│  │  │  CONTEXT    │  │    VCRs     │  │    PRDs     │  │   CODE      │ │  │
│  │  │   FILES     │  │ (Decisions) │  │  (Specs)    │  │  (Git)      │ │  │
│  │  │             │  │             │  │             │  │             │ │  │
│  │  │ AGENTS.md   │  │ VCR-001.md  │  │ Phase 1     │  │ Commits     │ │  │
│  │  │ CLAUDE.md   │  │ VCR-002.md  │  │ Phase 2     │  │ Branches    │ │  │
│  │  │ MentorScript│  │ ...         │  │ ...         │  │ PRs         │ │  │
│  │  └─────────────┘  └─────────────┘  └─────────────┘  └─────────────┘ │  │
│  │                                                                       │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                        │               │                                    │
│        ┌───────────────┼───────────────┼───────────────┐                    │
│        │               │               │               │                    │
│        ▼               ▼               ▼               ▼                    │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐            │
│  │  PLANNING  │  │   IMPL     │  │   REVIEW   │  │  TESTING   │            │
│  │   AGENT    │  │   AGENT    │  │   AGENT    │  │   AGENT    │            │
│  │            │  │            │  │            │  │            │            │
│  │ Break down │  │ Write code │  │ Check      │  │ Generate   │            │
│  │ tasks      │  │ per specs  │  │ quality    │  │ tests      │            │
│  └────────────┘  └────────────┘  └────────────┘  └────────────┘            │
│                                                                              │
│  AGENT LAYER (N agents)                                                      │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  KEY: Shared artifacts enable coordination without real-time communication  │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Beyond 1:1 Pairing

The evolution of collaboration models:

| Model | Description | Use Case |
|-------|-------------|----------|
| **1:1** | One human, one AI assistant | Individual development |
| **1:N** | One human orchestrating multiple agents | Complex solo projects |
| **N:1** | Team sharing one AI configuration | Small teams, shared context |
| **N:N** | Multiple humans, multiple specialized agents | Enterprise scale |

### N-to-N Collaboration Principles

In N-to-N collaboration, the key challenges shift from "how do I work with AI" to "how do we coordinate humans and agents at scale":

**Shared Artifacts as Coordination Points:**
- The codebase becomes the primary coordination artifact
- Context files (AGENTS.md, CLAUDE.md, MentorScript) serve as shared understanding
- VCR (Version Controlled Resolution) records capture team decisions
- PRDs and specs define the work boundary for all participants

**Role Differentiation:**
- **Human Coaches:** Define strategy, make judgment calls, approve critical changes
- **Planning Agents:** Break down tasks, create specifications
- **Implementation Agents:** Write code according to specs
- **Review Agents:** Check quality, security, and compliance
- **Testing Agents:** Generate and execute test suites

**Escalation Pathways:**
When agents encounter situations beyond their autonomy level, they generate a CRP (Consultation Request Pack) containing:
- Context: What was being attempted
- Options: Potential approaches identified
- Tradeoffs: Pros and cons of each option
- Recommendation: Agent's suggested path forward

The human responds with a VCR (Version Controlled Resolution) that becomes institutional memory for future similar situations.

### Coordination Mechanisms

**Asynchronous Coordination:**
- Agents work independently on assigned tasks
- Progress tracked through shared files and commit history
- Human checkpoints at defined milestones
- No real-time supervision required for routine work

**Synchronous Checkpoints:**
- Architecture decisions require human approval
- Security-sensitive changes need senior review
- Cross-agent conflicts escalated to human arbitration
- Phase transitions (e.g., design to implementation) validated by humans

**Institutional Memory:**
- VCRs accumulate as a knowledge base of team decisions
- MentorScript captures evolving team patterns and preferences
- Context files updated to reflect lessons learned
- New agents "onboard" by reading accumulated artifacts

---

## Shared Context Files

### Team AGENTS.md

Checked into the repository, shared by all team members:

```markdown
# AGENTS.md

## Team Standards

### Code Style
- TypeScript strict mode required
- Functional components only (React)
- No classes except for error types
- Prefer composition over inheritance

### Commit Standards
- Conventional commits format
- Reference ticket in body
- One logical change per commit

### Testing Requirements
- Unit tests for all business logic
- Integration tests for API endpoints
- E2E tests for critical user flows
- Minimum 80% coverage for new code

### Security Requirements
- All user input validated with Zod
- Parameterized queries only (Prisma handles this)
- No secrets in code (use environment variables)
- Review security for any auth/payment changes

## AI-Specific Guidelines

### What AI Can Do Autonomously
- Implement features following established patterns
- Write tests
- Fix linting errors
- Update documentation

### What Requires Human Review
- Any authentication/authorization changes
- Database schema changes
- External API integrations
- Performance-critical code
- Security-related changes

### What AI Should NOT Do
- Create new architectural patterns without discussion
- Modify CI/CD configuration
- Change dependency versions without justification
- Access production data or systems
```

### Team CLAUDE.md

```markdown
# CLAUDE.md - Team Configuration

## Project Context
[Your project description]

## Common Commands
- `pnpm dev` - Start development
- `pnpm test` - Run tests
- `pnpm lint` - Check linting
- `pnpm build` - Production build

## Workflow Expectations

### Before Starting Work
1. Read the ticket/issue completely
2. Check existing patterns for similar features
3. Ask clarifying questions before implementing

### During Implementation
1. Follow existing patterns (see example files below)
2. Write tests alongside code
3. Keep commits atomic

### Before Submitting
1. All tests pass
2. Linting clean
3. Self-review diff for issues
4. Update documentation if needed

## Example Files to Follow
- Component pattern: src/components/UserCard.tsx
- Service pattern: src/services/UserService.ts
- Test pattern: src/__tests__/UserService.test.ts
- API route pattern: src/app/api/users/route.ts

## Team Conventions
- PR titles: [TYPE] Short description
- Branch names: type/ticket-number-short-description
- All PRs require at least one approval
```

### Version-Controlled MCP Configuration

Share MCP servers across the team:

```json
// .mcp.json (checked into repo)
{
  "servers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"]
    },
    "database": {
      "command": "npx",
      "args": ["@mcp/postgres-server"],
      "env": {
        "DATABASE_URL": "${DATABASE_URL}"
      }
    },
    "sentry": {
      "command": "npx",
      "args": ["@mcp/sentry-server"]
    }
  }
}
```

---

## Team MentorScript

Beyond technical context files, teams benefit from a **MentorScript** - a "mentorship-as-code" artifact that codifies team culture, norms, and accumulated wisdom for both human and AI team members.

### What is MentorScript?

A MentorScript captures the implicit knowledge that senior engineers share through mentorship:

- **Team Philosophy:** Core beliefs about quality, speed, and tradeoffs
- **Decision Principles:** How to make judgment calls in ambiguous situations
- **Anti-Patterns:** Common mistakes to avoid (learned from experience)
- **Pattern Preferences:** When to use which approaches
- **Review Standards:** What reviewers look for and why

### MentorScript vs. Other Context Files

| File | Purpose | Audience |
|------|---------|----------|
| **AGENTS.md** | Technical standards and constraints | All AI tools |
| **CLAUDE.md** | Project context and commands | Claude Code specifically |
| **MentorScript** | Team culture and judgment | Both humans and AI |

MentorScript complements technical files by encoding the "soft" knowledge that determines whether technically correct code is actually appropriate for the team.

### MentorScript Components

**Team Philosophy:**
What does this team believe? Examples:
- "We ship fast but never ship broken"
- "Tests are documentation, not overhead"
- "Code review is mentorship, not gatekeeping"
- "AI amplifies our capabilities, it doesn't replace our judgment"

**Decision Frameworks:**
How to make choices in ambiguous situations:
- "When in doubt, choose the more maintainable option over the faster one"
- "If a library exists for it, use the library unless there's a compelling reason not to"
- "Prefer explicit over implicit, even if it's more verbose"

**AI Integration Standards:**
Team-specific guidance for AI use:
- When AI should be used (boilerplate, tests, documentation)
- When AI should not be used (security-critical, novel algorithms)
- Attribution requirements for AI-generated code
- Review standards for AI-assisted work

**Learning Resources:**
Pointers to deeper knowledge:
- Team wiki and architecture decision records
- Example code that embodies team patterns
- Senior engineers to consult for specific domains

### MentorScript Evolution

MentorScript is a living document that evolves:

**Initial Creation (Week 1-2):**
- Senior engineers document existing implicit knowledge
- Focus on most common decisions and patterns
- Keep it concise - aim for what fits in one read

**Active Refinement (Ongoing):**
- Add new entries when patterns emerge
- Update when decisions prove wrong
- Remove guidance that becomes standard practice

**Triggers for Updates:**
- Post-incident reviews reveal missing guidance
- Code reviews repeatedly address same issue
- New team members ask similar questions
- AI consistently makes the same mistakes

### Complete MentorScript Template

Below is a complete MentorScript template that teams can adapt to their needs:

```markdown
# MENTORSCRIPT.md - Engineering Team Alpha

## Team Philosophy

We are a high-trust, high-quality team. We believe in:
- Shipping fast but never shipping broken
- Testing as documentation
- Code review as mentorship, not gatekeeping
- AI as amplifier, not replacement

## AI Integration Standards

### When AI Should Be Used
- Boilerplate and scaffolding
- Test generation
- Documentation
- Code review assistance
- Refactoring under test

### When AI Should NOT Be Used
- Authentication/authorization implementation (use libraries)
- Cryptographic code (use audited libraries)
- Financial calculations (human review required)
- Compliance-critical code (legal review required)

### AI Attribution Requirements
All AI-assisted code must include:
1. `AI-Assisted: true` in commit message
2. AI tool name (Claude Code, Cursor, etc.)
3. Reviewer acknowledgment of AI involvement

## Code Style Preferences

### Naming
- Use descriptive names over short names
- Prefer `getUserById` over `getUser`
- Avoid abbreviations except well-known ones (ID, URL)

### Error Handling
- Always handle errors explicitly
- Prefer custom error types over generic errors
- Log errors with context, not just message

### Testing
- Every feature needs tests before merge
- Prefer integration tests over mocks
- Use property-based tests for validation logic

## Common Patterns (AI Should Follow)

### Data Fetching
\`\`\`typescript
// Preferred pattern for data fetching
async function fetchUsers(): Promise<User[]> {
  try {
    const response = await api.get('/users');
    return response.data;
  } catch (error) {
    logger.error('Failed to fetch users', { error });
    throw new FetchError('users', error);
  }
}
\`\`\`

### Error Boundaries
\`\`\`typescript
// Preferred pattern for error handling
class ServiceError extends Error {
  constructor(
    public service: string,
    public operation: string,
    public cause: Error
  ) {
    super(`${service}.${operation} failed: ${cause.message}`);
  }
}
\`\`\`

## Review Standards

### What Reviewers Check
1. Does it meet requirements?
2. Are there tests?
3. Is AI-generated code understood?
4. Security considerations addressed?
5. Performance acceptable?

### Review SLA
- P0/P1: Same day review
- P2: Within 24 hours
- P3: Within 48 hours

## Learning Resources
- Team wiki: [link]
- Architecture decisions: /docs/adr/
- Example code: /examples/
- Ask: @tech-lead or @senior-engineers
```

This template provides a starting point. Teams should customize the philosophy, patterns, and standards to reflect their actual practices and values.

### Integration with AGENTS.md

While AGENTS.md and MentorScript serve different purposes, they work together:

**AGENTS.md handles:**
- "Use TypeScript strict mode"
- "Tests required for all features"
- "No direct database queries outside services"

**MentorScript handles:**
- "We prefer integration tests over unit tests with mocks because we've found they catch more real bugs"
- "When choosing between performance and readability, choose readability unless you have measured evidence of a performance problem"
- "New patterns should be discussed in design review before implementation - don't create architectural precedents in isolation"

Both humans and AI agents benefit from this separation: AGENTS.md provides the rules, MentorScript provides the reasoning and judgment.

---

## Governance Frameworks

### Enterprise Approach

Large organizations need formal governance:

```
┌─────────────────────────────────────────────────────────────┐
│              AI Code Governance Framework                    │
│                                                             │
│  ┌─────────────────────────────────────────────────┐       │
│  │                   POLICY LAYER                   │       │
│  │  • Acceptable use policy                        │       │
│  │  • Risk classification criteria                 │       │
│  │  • Review requirements by risk level            │       │
│  └─────────────────────────────────────────────────┘       │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────┐       │
│  │                  PROCESS LAYER                   │       │
│  │  • Mandatory review gates                       │       │
│  │  • Security scan requirements                   │       │
│  │  • Approval workflows                           │       │
│  └─────────────────────────────────────────────────┘       │
│                          │                                  │
│                          ▼                                  │
│  ┌─────────────────────────────────────────────────┐       │
│  │                 TECHNICAL LAYER                  │       │
│  │  • CI/CD enforcement                            │       │
│  │  • Automated scanning                           │       │
│  │  • Metrics collection                           │       │
│  └─────────────────────────────────────────────────┘       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Startup Approach

Lighter-weight for smaller teams:

```markdown
## AI Development Guidelines

### Trust but Verify
- AI-generated code is welcome
- All code must be reviewed by a human
- Reviewer takes responsibility for what they approve

### Risk-Based Review
- Low risk (UI, utilities): Regular review
- Medium risk (business logic): Thorough review
- High risk (auth, payment, data): Senior review + security scan

### Documentation
- AI-assisted code should be noted in PR
- Unusual approaches should be explained
- Test coverage required
```

---

## Tracking AI Contribution Metrics

### What to Track

| Metric | What It Measures | Why It Matters |
|--------|-----------------|----------------|
| **AI Code Ratio** | % of code from AI | Adoption rate |
| **Acceptance Rate** | % of AI code accepted | Quality of prompting |
| **Defect Rate** | Bugs per AI LOC vs human LOC | Quality comparison |
| **Review Time** | Time to review AI code | Process efficiency |
| **Rework Rate** | Changes needed after AI generation | First-pass quality |

### Implementation

```python
# Track AI contribution in git metadata
# Add to commit messages: AI-Assisted: true/false

def get_ai_contribution_metrics(repo_path, start_date, end_date):
    commits = get_commits(repo_path, start_date, end_date)

    ai_commits = [c for c in commits if 'AI-Assisted: true' in c.message]
    human_commits = [c for c in commits if 'AI-Assisted: true' not in c.message]

    ai_lines = sum(c.lines_added for c in ai_commits)
    human_lines = sum(c.lines_added for c in human_commits)

    return {
        'ai_commit_count': len(ai_commits),
        'human_commit_count': len(human_commits),
        'ai_lines_added': ai_lines,
        'human_lines_added': human_lines,
        'ai_code_ratio': ai_lines / (ai_lines + human_lines) if (ai_lines + human_lines) > 0 else 0
    }
```

### Commit Message Convention

```
feat(auth): add password reset flow

- Implemented reset token generation
- Added email sending for reset link
- Created reset confirmation page

AI-Assisted: true
AI-Tool: Claude Code
Reviewed-By: @engineer

Closes #123
```

### Complete Team Metrics Dashboard Script

Below is a complete, runnable Python script for generating weekly AI contribution reports. This can be run as a cron job or integrated into CI/CD pipelines:

```python
#!/usr/bin/env python3
"""
Generate weekly AI contribution metrics for the team.

Usage:
    python ai_metrics_dashboard.py [--days 7] [--output report.md]

Requirements:
    - Git repository with AI-Assisted commit tags
    - Python 3.8+
"""

import subprocess
import json
import argparse
from datetime import datetime, timedelta
from collections import defaultdict
from pathlib import Path


def get_commits_since(days_ago: int) -> list[str]:
    """Get commits from the last N days."""
    since_date = (datetime.now() - timedelta(days=days_ago)).strftime('%Y-%m-%d')
    try:
        result = subprocess.check_output([
            'git', 'log', f'--since={since_date}',
            '--pretty=format:%H|%an|%ae|%s|%b|||',  # Using ||| as record separator
            '--all'
        ], text=True, stderr=subprocess.DEVNULL)
        # Split by record separator and filter empty entries
        return [r.strip() for r in result.split('|||') if r.strip()]
    except subprocess.CalledProcessError:
        return []


def get_commit_stats(sha: str) -> dict:
    """Get lines added/removed for a commit."""
    try:
        result = subprocess.check_output([
            'git', 'show', '--stat', '--format=', sha
        ], text=True, stderr=subprocess.DEVNULL)

        lines_added = 0
        lines_removed = 0
        for line in result.split('\n'):
            if 'insertion' in line or 'deletion' in line:
                parts = line.split(',')
                for part in parts:
                    if 'insertion' in part:
                        lines_added += int(part.strip().split()[0])
                    if 'deletion' in part:
                        lines_removed += int(part.strip().split()[0])
        return {'added': lines_added, 'removed': lines_removed}
    except (subprocess.CalledProcessError, ValueError):
        return {'added': 0, 'removed': 0}


def parse_commit(commit_line: str) -> dict | None:
    """Parse a commit line into structured data."""
    parts = commit_line.split('|', 4)
    if len(parts) < 5:
        return None

    sha, author, email, subject, body = parts[0], parts[1], parts[2], parts[3], parts[4]

    # Detect AI assistance
    full_text = f"{subject} {body}"
    ai_assisted = 'AI-Assisted: true' in full_text or 'ai-assisted: true' in full_text.lower()

    # Detect AI tool
    ai_tool = None
    if ai_assisted:
        tool_patterns = [
            ('Claude Code', ['Claude Code', 'claude-code']),
            ('Cursor', ['Cursor', 'cursor']),
            ('GitHub Copilot', ['Copilot', 'copilot', 'GitHub Copilot']),
            ('ChatGPT', ['ChatGPT', 'chatgpt']),
            ('Gemini', ['Gemini', 'gemini']),
        ]
        for tool_name, patterns in tool_patterns:
            if any(p in full_text for p in patterns):
                ai_tool = tool_name
                break
        if not ai_tool:
            ai_tool = 'Unknown'

    return {
        'sha': sha,
        'author': author,
        'email': email,
        'subject': subject,
        'ai_assisted': ai_assisted,
        'ai_tool': ai_tool
    }


def calculate_metrics(commits: list[str]) -> dict:
    """Calculate team AI metrics."""
    metrics = {
        'total_commits': 0,
        'ai_assisted_commits': 0,
        'by_author': defaultdict(lambda: {'total': 0, 'ai': 0, 'lines_added': 0, 'ai_lines': 0}),
        'by_tool': defaultdict(int),
        'ai_ratio': 0.0,
        'total_lines_added': 0,
        'ai_lines_added': 0
    }

    for commit_line in commits:
        parsed = parse_commit(commit_line)
        if not parsed:
            continue

        stats = get_commit_stats(parsed['sha'])

        metrics['total_commits'] += 1
        metrics['total_lines_added'] += stats['added']
        metrics['by_author'][parsed['author']]['total'] += 1
        metrics['by_author'][parsed['author']]['lines_added'] += stats['added']

        if parsed['ai_assisted']:
            metrics['ai_assisted_commits'] += 1
            metrics['ai_lines_added'] += stats['added']
            metrics['by_author'][parsed['author']]['ai'] += 1
            metrics['by_author'][parsed['author']]['ai_lines'] += stats['added']
            if parsed['ai_tool']:
                metrics['by_tool'][parsed['ai_tool']] += 1

    if metrics['total_commits'] > 0:
        metrics['ai_ratio'] = metrics['ai_assisted_commits'] / metrics['total_commits']

    if metrics['total_lines_added'] > 0:
        metrics['ai_line_ratio'] = metrics['ai_lines_added'] / metrics['total_lines_added']
    else:
        metrics['ai_line_ratio'] = 0.0

    return metrics


def generate_report(metrics: dict, days: int) -> str:
    """Generate a markdown report."""
    report = []
    report.append("# Weekly AI Contribution Report")
    report.append(f"\n**Generated:** {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    report.append(f"**Period:** Last {days} days")
    report.append("")

    # Summary section
    report.append("## Summary")
    report.append("")
    report.append(f"| Metric | Value |")
    report.append(f"|--------|-------|")
    report.append(f"| Total commits | {metrics['total_commits']} |")
    report.append(f"| AI-assisted commits | {metrics['ai_assisted_commits']} |")
    report.append(f"| AI commit ratio | {metrics['ai_ratio']:.1%} |")
    report.append(f"| Total lines added | {metrics['total_lines_added']:,} |")
    report.append(f"| AI-assisted lines | {metrics['ai_lines_added']:,} |")
    report.append(f"| AI line ratio | {metrics['ai_line_ratio']:.1%} |")
    report.append("")

    # By team member section
    report.append("## By Team Member")
    report.append("")
    report.append("| Author | Commits | AI Commits | AI Ratio | Lines | AI Lines |")
    report.append("|--------|---------|------------|----------|-------|----------|")
    for author, data in sorted(metrics['by_author'].items(), key=lambda x: -x[1]['total']):
        ratio = data['ai'] / data['total'] if data['total'] > 0 else 0
        report.append(
            f"| {author} | {data['total']} | {data['ai']} | {ratio:.1%} | "
            f"{data['lines_added']:,} | {data['ai_lines']:,} |"
        )
    report.append("")

    # By AI tool section
    if metrics['by_tool']:
        report.append("## By AI Tool")
        report.append("")
        report.append("| Tool | Commits |")
        report.append("|------|---------|")
        for tool, count in sorted(metrics['by_tool'].items(), key=lambda x: -x[1]):
            report.append(f"| {tool} | {count} |")
        report.append("")

    # Insights section
    report.append("## Insights")
    report.append("")

    if metrics['ai_ratio'] > 0.5:
        report.append("- **High AI adoption** - Over half of commits are AI-assisted")
    elif metrics['ai_ratio'] > 0.2:
        report.append("- **Moderate AI adoption** - AI assistance is becoming common")
    else:
        report.append("- **Low AI adoption** - Consider training on AI-assisted workflows")

    if metrics['by_author']:
        top_ai_user = max(metrics['by_author'].items(),
                         key=lambda x: x[1]['ai'] / x[1]['total'] if x[1]['total'] > 0 else 0)
        if top_ai_user[1]['total'] > 0:
            ratio = top_ai_user[1]['ai'] / top_ai_user[1]['total']
            report.append(f"- **Top AI user:** {top_ai_user[0]} ({ratio:.1%} AI-assisted)")

    report.append("")

    return '\n'.join(report)


def main():
    parser = argparse.ArgumentParser(description='Generate AI contribution metrics')
    parser.add_argument('--days', type=int, default=7, help='Number of days to analyze')
    parser.add_argument('--output', type=str, default='ai-metrics-report.md',
                       help='Output file path')
    parser.add_argument('--json', action='store_true', help='Also output JSON metrics')
    args = parser.parse_args()

    print(f"Analyzing commits from the last {args.days} days...")

    commits = get_commits_since(args.days)
    print(f"Found {len(commits)} commits")

    metrics = calculate_metrics(commits)
    report = generate_report(metrics, args.days)

    # Print to console
    print("\n" + report)

    # Save markdown report
    with open(args.output, 'w') as f:
        f.write(report)
    print(f"\nReport saved to {args.output}")

    # Optionally save JSON metrics
    if args.json:
        json_path = Path(args.output).with_suffix('.json')
        # Convert defaultdict to regular dict for JSON serialization
        json_metrics = {
            **metrics,
            'by_author': dict(metrics['by_author']),
            'by_tool': dict(metrics['by_tool'])
        }
        with open(json_path, 'w') as f:
            json.dump(json_metrics, f, indent=2)
        print(f"JSON metrics saved to {json_path}")


if __name__ == "__main__":
    main()
```

**Usage Examples:**

```bash
# Basic usage - last 7 days
python ai_metrics_dashboard.py

# Custom time range
python ai_metrics_dashboard.py --days 30

# Custom output file
python ai_metrics_dashboard.py --output weekly-report.md

# Also generate JSON for further processing
python ai_metrics_dashboard.py --json
```

This script can be integrated into CI/CD pipelines or run as a weekly cron job to track team AI adoption over time.

---

## Team Workflow Patterns

### Pattern 1: Pair Programming with AI

```
┌─────────────────────────────────────────────────────────────┐
│              Human + AI Pair Programming                     │
│                                                             │
│  Human (Driver)          AI (Navigator)                     │
│  ┌──────────────┐       ┌──────────────┐                   │
│  │ • Define task │  ←→  │ • Suggest impl│                   │
│  │ • Review code │       │ • Find bugs  │                   │
│  │ • Make calls  │       │ • Write tests│                   │
│  │ • Commit work │       │ • Document   │                   │
│  └──────────────┘       └──────────────┘                   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Pattern 2: Ticket-Driven Development

```markdown
## Workflow: Ticket → AI Implementation → Human Review

### 1. Ticket Creation (Human)
- Write clear acceptance criteria
- Link to relevant context (design, PRD)
- Set risk level (low/medium/high)

### 2. Implementation (AI)
- Read ticket and linked context
- Implement according to criteria
- Write tests
- Create PR with MRP

### 3. Review (Human)
- Verify against acceptance criteria
- Check code quality
- Validate tests
- Approve or request changes

### 4. Merge (Human)
- Final approval
- Merge to main
- Close ticket
```

### Pattern 3: Async Agent Work

```
┌─────────────────────────────────────────────────────────────┐
│              Async Agent Workflow                           │
│                                                             │
│  Morning:                                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Human assigns tickets to AI with context files    │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                  │
│                          ▼                                  │
│  Daytime:                                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  AI works on tickets, creates PRs                  │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                  │
│                          ▼                                  │
│  End of day:                                                │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Human reviews PRs, provides feedback, merges      │    │
│  └────────────────────────────────────────────────────┘    │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Cross-Team Coordination

When multiple teams in an organization adopt AI-assisted development, coordination challenges emerge that require explicit attention.

### Federated AGENTS.md Pattern

Rather than one monolithic AGENTS.md, organizations can use a federated approach:

**Organization-Level (org-agents.md):**
- Company-wide coding standards
- Security requirements applicable to all code
- Compliance frameworks and audit requirements
- Shared tooling and infrastructure patterns

**Domain-Level (domain-agents.md):**
- Domain-specific patterns (e.g., payment processing, data pipelines)
- Shared libraries and their usage conventions
- Cross-team API contracts and expectations
- Domain expertise and edge cases

**Team-Level (AGENTS.md):**
- Team-specific conventions and preferences
- Local patterns that extend organization standards
- Team member roles and escalation paths
- Project-specific context

**Inheritance Model:**
```
org-agents.md          (organization-wide)
    └── domain-agents.md   (domain-specific)
            └── AGENTS.md      (team-specific)
```

Each level inherits from its parent and can extend or (rarely) override inherited guidance.

### N-to-N Collaboration Configuration

For teams operating in N-to-N mode (multiple humans, multiple agents), a configuration file can formalize roles, capabilities, and workflows:

```yaml
# team-agents-config.yaml
# Configuration for multi-human, multi-agent collaboration

team:
  name: "Platform Engineering"

  humans:
    - role: tech_lead
      name: "Sarah"
      specialties: ["architecture", "security"]
      approval_required_for: ["auth", "payments", "infrastructure"]

    - role: senior_engineer
      name: "Alex"
      specialties: ["backend", "database"]
      approval_required_for: ["schema_changes", "migrations"]

    - role: engineer
      name: "Jordan"
      specialties: ["frontend", "testing"]

agents:
  - name: "planning-agent"
    purpose: "Break down tasks and create specifications"
    autonomy_level: 3
    tools: ["read", "search", "plan"]
    escalation_triggers:
      - "architectural_decision"
      - "external_dependency"

  - name: "implementation-agent"
    purpose: "Write code according to specifications"
    autonomy_level: 2
    tools: ["read", "write", "test", "commit"]
    escalation_triggers:
      - "security_related"
      - "unfamiliar_pattern"

  - name: "review-agent"
    purpose: "Review code for quality and security"
    autonomy_level: 2
    tools: ["read", "analyze", "comment"]
    auto_approve: false

  - name: "testing-agent"
    purpose: "Generate and run tests"
    autonomy_level: 3
    tools: ["read", "write", "test"]

workflows:
  feature_development:
    steps:
      - agent: planning-agent
        action: create_specification
        requires_approval: tech_lead

      - agent: implementation-agent
        action: implement
        parallel: true
        requires_approval: none

      - agent: testing-agent
        action: test
        requires_approval: none

      - agent: review-agent
        action: review
        requires_approval: senior_engineer

      - human: any
        action: final_merge

  bug_fix:
    steps:
      - agent: implementation-agent
        action: diagnose_and_fix
        requires_approval: none

      - agent: testing-agent
        action: verify_fix
        requires_approval: none

      - human: any
        action: review_and_merge

escalation:
  default_path: ["implementation-agent", "review-agent", "senior_engineer", "tech_lead"]
  security_path: ["review-agent", "tech_lead"]
  architecture_path: ["planning-agent", "tech_lead"]
```

This configuration provides a declarative way to specify team structure and coordination patterns. While current tools don't natively consume this format, it serves as documentation and can inform custom orchestration tooling.

### Knowledge Sharing Across Teams

**Cross-Pollination Patterns:**
- Teams share successful prompt patterns in a central repository
- VCRs from one team become learning material for others
- MentorScripts can be forked and adapted across teams
- Regular cross-team AI practice reviews identify best approaches

**Anti-Patterns to Avoid:**
- Siloed AI usage where teams don't share learnings
- Inconsistent governance leading to compliance gaps
- Duplicated context files that drift apart
- No mechanism for propagating discovered patterns

### Scaling Governance

**Centralized Elements:**
- Security scanning requirements
- Audit trail standards
- Risk classification criteria
- Tool approval and procurement

**Decentralized Elements:**
- Day-to-day workflow decisions
- Team-specific conventions
- Local tooling preferences
- Prompt engineering approaches

The key is distinguishing between decisions that require organizational consistency (security, compliance, audit) and those that benefit from team autonomy (workflow, conventions, tooling).

---

## Legal and Compliance

### Code Ownership

Key questions to address:

- Who owns AI-generated code? (Usually the company)
- Is there IP risk from training data? (Depends on model/provider)
- What license applies? (Same as other company code)

### Compliance Considerations

```markdown
## AI Code Compliance Checklist

### Data Privacy
- [ ] AI doesn't have access to customer PII
- [ ] AI prompts don't include sensitive data
- [ ] Generated code doesn't expose PII

### Regulatory
- [ ] AI usage documented for audit
- [ ] Human review for regulated features
- [ ] Approval workflow for compliance-critical code

### Intellectual Property
- [ ] Using enterprise AI agreement
- [ ] No external code snippets without license review
- [ ] Proprietary algorithms human-written
```

### Audit Trail

```json
// Maintain audit trail for AI-generated code
{
  "commit_sha": "abc123",
  "ai_assisted": true,
  "ai_tool": "Claude Code",
  "ai_model": "claude-3-opus",
  "prompt_summary": "Implement password reset feature",
  "files_affected": ["src/auth/reset.ts", "src/auth/reset.test.ts"],
  "reviewed_by": "engineer@company.com",
  "approved_at": "2026-02-08T14:30:00Z"
}
```

---

## Team Onboarding

### New Team Member AI Onboarding

```markdown
## AI-Assisted Development Onboarding

### Day 1: Foundation
1. Read team AGENTS.md and CLAUDE.md
2. Set up Claude Code with team configuration
3. Complete: First AI-assisted task (low risk)

### Week 1: Practice
4. Complete 3-5 AI-assisted tasks
5. Get feedback on prompting approach
6. Review other team members' AI PRs

### Week 2+: Integration
7. Tackle medium-risk tasks
8. Contribute improvements to context files
9. Share learnings with team

### Resources
- Team AI guidelines: [link]
- Example prompts: [link]
- Common patterns: [link]
- Who to ask: @senior-engineer
```

### Automated Onboarding Script

Below is a script that automates key onboarding tasks for new team members joining an AI-assisted development team:

```bash
#!/bin/bash
# ai-onboarding.sh
# Automated onboarding for new team members

set -e

echo "🚀 AI-Assisted Development Onboarding Script"
echo "============================================="
echo ""

# Colors for output
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Configuration
TEAM_REPO="${TEAM_REPO:-your-org/your-repo}"
ONBOARDING_BRANCH="onboarding-$(date +%Y%m%d)"

print_step() {
    echo -e "${GREEN}✓ $1${NC}"
}

print_info() {
    echo -e "${YELLOW}ℹ $1${NC}"
}

# Step 1: Verify prerequisites
echo "Step 1: Checking prerequisites..."
command -v git >/dev/null 2>&1 || { echo "git is required but not installed."; exit 1; }
command -v claude >/dev/null 2>&1 || { echo "Claude Code is required but not installed."; exit 1; }
print_step "Prerequisites verified (git, claude)"
echo ""

# Step 2: Clone and setup repository
echo "Step 2: Setting up repository..."
if [ ! -d "$(basename $TEAM_REPO)" ]; then
    git clone "git@github.com:$TEAM_REPO.git"
    cd "$(basename $TEAM_REPO)"
else
    cd "$(basename $TEAM_REPO)"
    git pull origin main
fi
print_step "Repository ready"
echo ""

# Step 3: Display key context files
echo "Step 3: Key context files to read..."
echo ""

if [ -f "AGENTS.md" ]; then
    print_info "AGENTS.md - Team AI standards and guidelines"
    echo "    Location: $(pwd)/AGENTS.md"
    echo "    Run: cat AGENTS.md | head -50"
fi

if [ -f "CLAUDE.md" ]; then
    print_info "CLAUDE.md - Claude-specific project context"
    echo "    Location: $(pwd)/CLAUDE.md"
fi

if [ -f "MENTORSCRIPT.md" ]; then
    print_info "MENTORSCRIPT.md - Team philosophy and patterns"
    echo "    Location: $(pwd)/MENTORSCRIPT.md"
fi

echo ""
print_step "Context files identified"
echo ""

# Step 4: Verify Claude Code configuration
echo "Step 4: Verifying Claude Code configuration..."
if [ -f ".mcp.json" ]; then
    print_info "MCP configuration found"
    echo "    Servers configured:"
    cat .mcp.json | grep -A1 '"servers"' | head -5 || true
fi
print_step "Claude Code ready"
echo ""

# Step 5: Create practice task
echo "Step 5: Creating practice task..."
git checkout -b "$ONBOARDING_BRANCH" 2>/dev/null || git checkout "$ONBOARDING_BRANCH"

cat > "onboarding-task.md" << 'EOF'
# Onboarding Practice Task

Welcome to the team! Complete this task to practice our AI-assisted workflow.

## Task: Add a Utility Function

Create a simple utility function with tests. This exercises our standard workflow.

### Requirements
1. Create a utility function in `src/utils/` (or appropriate location)
2. Function: `formatDuration(milliseconds: number): string`
3. Should return human-readable duration (e.g., "2h 30m", "45s", "1d 3h")
4. Write tests covering edge cases

### Workflow to Follow
1. Read AGENTS.md and CLAUDE.md first
2. Use Claude Code to implement
3. Follow our commit message convention
4. Include `AI-Assisted: true` in commit if using AI

### Acceptance Criteria
- [ ] Function handles 0 milliseconds
- [ ] Function handles values < 1 second
- [ ] Function handles hours, days
- [ ] Tests cover all cases
- [ ] Code follows team patterns

### Getting Help
- Slack: #engineering-help
- Team lead: @tech-lead
- AI tips: #ai-assisted-dev
EOF

git add onboarding-task.md
git commit -m "docs: add onboarding practice task

Setting up onboarding task for new team member.

AI-Assisted: false"

print_step "Practice task created: onboarding-task.md"
echo ""

# Step 6: Summary and next steps
echo "============================================="
echo "Onboarding setup complete!"
echo "============================================="
echo ""
echo "Next steps:"
echo "  1. Read the context files listed above"
echo "  2. Complete the practice task in onboarding-task.md"
echo "  3. Use 'claude' to start Claude Code"
echo "  4. Ask questions in #engineering-help"
echo ""
echo "Useful commands:"
echo "  claude                    - Start Claude Code"
echo "  git log --oneline -10    - See recent commits"
echo "  cat AGENTS.md            - View team AI standards"
echo ""
echo "Happy coding! 🎉"
```

**Corresponding Python version for cross-platform support:**

```python
#!/usr/bin/env python3
"""
Automated onboarding for new team members.

Usage:
    python ai_onboarding.py --repo your-org/your-repo --name "New Developer"
"""

import subprocess
import argparse
import sys
from pathlib import Path
from datetime import datetime


def run_command(cmd: list[str], check: bool = True) -> subprocess.CompletedProcess:
    """Run a command and return the result."""
    return subprocess.run(cmd, capture_output=True, text=True, check=check)


def check_prerequisites() -> list[str]:
    """Check that required tools are installed."""
    missing = []
    tools = ['git', 'claude']

    for tool in tools:
        result = run_command(['which', tool], check=False)
        if result.returncode != 0:
            missing.append(tool)

    return missing


def setup_repository(repo: str) -> Path:
    """Clone or update the repository."""
    repo_name = repo.split('/')[-1]
    repo_path = Path(repo_name)

    if not repo_path.exists():
        run_command(['git', 'clone', f'git@github.com:{repo}.git'])
    else:
        run_command(['git', '-C', str(repo_path), 'pull', 'origin', 'main'], check=False)

    return repo_path


def find_context_files(repo_path: Path) -> dict[str, Path]:
    """Find key context files in the repository."""
    files = {}
    for filename in ['AGENTS.md', 'CLAUDE.md', 'MENTORSCRIPT.md', '.mcp.json']:
        file_path = repo_path / filename
        if file_path.exists():
            files[filename] = file_path
    return files


def create_practice_task(repo_path: Path, developer_name: str) -> None:
    """Create an onboarding practice task."""
    branch_name = f"onboarding-{datetime.now().strftime('%Y%m%d')}"

    # Create branch
    run_command(['git', '-C', str(repo_path), 'checkout', '-b', branch_name], check=False)

    # Create task file
    task_content = f'''# Onboarding Practice Task

Welcome {developer_name}! Complete this task to practice our AI-assisted workflow.

## Task: Add a Utility Function

Create a simple utility function with tests.

### Requirements
1. Create a utility function in `src/utils/` (or appropriate location)
2. Function: `formatDuration(milliseconds: number): string`
3. Should return human-readable duration (e.g., "2h 30m", "45s")
4. Write tests covering edge cases

### Workflow to Follow
1. Read AGENTS.md and CLAUDE.md first
2. Use Claude Code to implement
3. Follow our commit message convention
4. Include `AI-Assisted: true` in commit if using AI

### Getting Help
- Slack: #engineering-help
- Team lead: @tech-lead
'''

    task_path = repo_path / 'onboarding-task.md'
    task_path.write_text(task_content)

    # Commit
    run_command(['git', '-C', str(repo_path), 'add', 'onboarding-task.md'])
    run_command([
        'git', '-C', str(repo_path), 'commit', '-m',
        'docs: add onboarding practice task\n\nAI-Assisted: false'
    ])


def generate_onboarding_checklist(developer_name: str, context_files: dict) -> str:
    """Generate a personalized onboarding checklist."""
    checklist = f'''
# AI-Assisted Development Onboarding Checklist
## {developer_name} - {datetime.now().strftime('%Y-%m-%d')}

### Day 1: Foundation
- [ ] Clone repository and verify setup
- [ ] Read all context files:
'''
    for filename, path in context_files.items():
        checklist += f'  - [ ] {filename} ({path})\n'

    checklist += '''
- [ ] Install and configure Claude Code
- [ ] Complete first AI-assisted task (practice task)

### Week 1: Practice
- [ ] Complete 3-5 AI-assisted tasks
- [ ] Get feedback on prompting approach from senior developer
- [ ] Review 2+ PRs from other team members
- [ ] Attend team AI practices meeting

### Week 2+: Integration
- [ ] Tackle a medium-risk task independently
- [ ] Contribute one improvement to context files
- [ ] Share one learning with the team

### Sign-off
- [ ] Senior developer approval
- [ ] Onboarding complete
'''
    return checklist


def main():
    parser = argparse.ArgumentParser(description='AI-Assisted Development Onboarding')
    parser.add_argument('--repo', required=True, help='Repository (org/repo)')
    parser.add_argument('--name', required=True, help='New developer name')
    args = parser.parse_args()

    print("AI-Assisted Development Onboarding")
    print("=" * 40)

    # Check prerequisites
    missing = check_prerequisites()
    if missing:
        print(f"Missing required tools: {', '.join(missing)}")
        sys.exit(1)
    print("✓ Prerequisites verified")

    # Setup repository
    repo_path = setup_repository(args.repo)
    print(f"✓ Repository ready: {repo_path}")

    # Find context files
    context_files = find_context_files(repo_path)
    print(f"✓ Found {len(context_files)} context files")

    # Create practice task
    create_practice_task(repo_path, args.name)
    print("✓ Practice task created")

    # Generate checklist
    checklist = generate_onboarding_checklist(args.name, context_files)
    checklist_path = repo_path / f'onboarding-checklist-{args.name.lower().replace(" ", "-")}.md'
    checklist_path.write_text(checklist)
    print(f"✓ Checklist saved: {checklist_path}")

    print("\nOnboarding setup complete!")
    print(f"Next: Read {list(context_files.keys())}")


if __name__ == "__main__":
    main()
```

These scripts automate the mechanical parts of onboarding while ensuring new team members are guided through the essential context files and workflows.

---

## Change Management for AI Adoption

Adopting AI-assisted development at team scale is not just a technical change - it requires deliberate change management to succeed.

### Team Adoption Journey

The following diagram illustrates the typical adoption journey for teams:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       TEAM ADOPTION JOURNEY                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STAGE 1                STAGE 2              STAGE 3            STAGE 4     │
│  EXPLORATION           STANDARDS           INTEGRATION        OPTIMIZATION  │
│  (Weeks 1-4)           (Weeks 4-8)         (Weeks 8-16)       (Ongoing)     │
│                                                                              │
│     ┌───┐               ┌───┐                ┌───┐              ┌───┐       │
│     │ ? │ ─────────────►│ = │ ─────────────►│ ∫ │ ────────────►│ ↑ │       │
│     └───┘               └───┘                └───┘              └───┘       │
│                                                                              │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐
│  │ CHARACTERISTICS │ │ CHARACTERISTICS │ │ CHARACTERISTICS │ │ CHARACTER.  │
│  │                 │ │                 │ │                 │ │             │
│  │ • Individual    │ │ • Shared        │ │ • MRP in        │ │ • Cross-team│
│  │   experiments   │ │   AGENTS.md     │ │   code review   │ │   coord.    │
│  │ • Informal      │ │ • CLAUDE.md     │ │ • Metrics       │ │ • Org-wide  │
│  │   sharing       │ │   created       │ │   tracking      │ │   practices │
│  │ • No standards  │ │ • Light         │ │ • Formal        │ │ • Regular   │
│  │                 │ │   governance    │ │   training      │ │   retros    │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────┘
│                                                                              │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐ ┌─────────────┐
│  │ SUCCESS METRIC  │ │ SUCCESS METRIC  │ │ SUCCESS METRIC  │ │ SUCCESS     │
│  │                 │ │                 │ │                 │ │ METRIC      │
│  │ Growing team    │ │ Consistent      │ │ Measurable      │ │ Org-wide    │
│  │ interest        │ │ practices       │ │ productivity    │ │ adoption    │
│  │                 │ │ across team     │ │ improvement     │ │ & optimize  │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘ └─────────────┘
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  ADOPTION CURVE                                                          ││
│  │                                                                          ││
│  │  100% │                                            ┌────────────────────││
│  │       │                                     ┌──────┘                     ││
│  │   50% │                              ┌──────┘                            ││
│  │       │                       ┌──────┘                                   ││
│  │    0% │───────────────────────┘                                          ││
│  │       └──────────────────────────────────────────────────────────────────││
│  │         Week 1    Week 4    Week 8    Week 16   Week 24                  ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  KEY: Allow each stage to stabilize before pushing to the next              │
└─────────────────────────────────────────────────────────────────────────────┘
```

Teams typically progress through four stages:

**Stage 1: Individual Exploration (Weeks 1-4)**
- Individual developers experiment with AI tools
- Informal sharing of tips and discoveries
- No team standards yet established
- Success metric: Growing interest from team members

**Stage 2: Team Standards (Weeks 4-8)**
- Team creates shared AGENTS.md and CLAUDE.md
- Light governance framework established
- Attribution conventions defined
- Success metric: Consistent practices across team

**Stage 3: Process Integration (Weeks 8-16)**
- MRP (Merge-Readiness Pack) integrated into code review
- Metrics tracking implemented
- Formal training for all team members
- Success metric: Measurable productivity improvement

**Stage 4: Continuous Improvement (Ongoing)**
- Cross-team coordination established
- Best practices shared organization-wide
- Regular retrospectives on AI usage
- Success metric: Organization-wide adoption and optimization

### Common Resistance Patterns

Teams encounter predictable resistance that leaders should anticipate:

| Resistance Pattern | Underlying Concern | Effective Response |
|-------------------|--------------------|--------------------|
| **"AI will take my job"** | Career anxiety | Emphasize AI as amplification, show new skills becoming valuable |
| **"I can't trust AI code"** | Quality concerns | Demonstrate MRP framework, show verification practices |
| **"It's too slow to review everything"** | Efficiency worries | Train on efficient review patterns, show net time savings |
| **"I prefer my own workflow"** | Comfort with status quo | Allow gradual adoption, start with pain points they identify |
| **"Legal won't approve this"** | Risk aversion | Involve legal early, show governance and audit capabilities |

### Success Metrics for Adoption

Track adoption progress with both quantitative and qualitative metrics:

**Quantitative Metrics:**
- AI code ratio over time (adoption rate)
- Defect rate comparison (AI-assisted vs. non-AI)
- Time to complete similar tasks (productivity)
- Review cycle time (process efficiency)
- Rework rate after AI generation (quality)

**Qualitative Indicators:**
- Team sentiment about AI tools (survey regularly)
- Quality of context files (improving over time?)
- Cross-team knowledge sharing (happening organically?)
- New hire time-to-productivity (decreasing?)

### Enterprise vs. Startup Governance Spectrum

The following diagram illustrates the governance spectrum from startup to enterprise:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  ENTERPRISE VS STARTUP GOVERNANCE SPECTRUM                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  STARTUP                                                        ENTERPRISE  │
│  ◄────────────────────────────────────────────────────────────────────────► │
│                                                                              │
│  VELOCITY                                                          CONTROL  │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                                                                          ││
│  │   STARTUP                   MID-SIZE                    ENTERPRISE       ││
│  │   ────────                  ─────────                   ──────────       ││
│  │                                                                          ││
│  │   REVIEW                    REVIEW                      REVIEW           ││
│  │   ┌──────────┐             ┌──────────┐                ┌──────────┐     ││
│  │   │ Post-hoc │             │  PR-time │                │ Multi-   │     ││
│  │   │  audit   │             │  review  │                │  gate    │     ││
│  │   └──────────┘             └──────────┘                └──────────┘     ││
│  │                                                                          ││
│  │   TRACKING                  TRACKING                    TRACKING         ││
│  │   ┌──────────┐             ┌──────────┐                ┌──────────┐     ││
│  │   │ Commit   │             │ Weekly   │                │ Full     │     ││
│  │   │  tags    │             │ metrics  │                │  audit   │     ││
│  │   └──────────┘             └──────────┘                │  trail   │     ││
│  │                                                         └──────────┘     ││
│  │   SECURITY                  SECURITY                    SECURITY         ││
│  │   ┌──────────┐             ┌──────────┐                ┌──────────┐     ││
│  │   │ Trust    │             │ Automated│                │ Security │     ││
│  │   │  devs    │             │  scans   │                │  team    │     ││
│  │   └──────────┘             └──────────┘                │  review  │     ││
│  │                                                         └──────────┘     ││
│  │   APPROVAL                  APPROVAL                    APPROVAL         ││
│  │   ┌──────────┐             ┌──────────┐                ┌──────────┐     ││
│  │   │ Self-    │             │ Peer     │                │ Formal   │     ││
│  │   │  govern  │             │ review   │                │ approval │     ││
│  │   └──────────┘             └──────────┘                │ workflow │     ││
│  │                                                         └──────────┘     ││
│  │                                                                          ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  CALIBRATION FACTORS:                                                        │
│  ┌─────────────────┬─────────────────┬─────────────────┬─────────────────┐ │
│  │  Regulatory     │  Risk           │   Team          │   Existing      │ │
│  │  Requirements   │  Tolerance      │   Size          │   Infrastructure│ │
│  │  Low ←──────► High│ High ←───► Low │ Small ←──► Large│ None ←────► Full│ │
│  └─────────────────┴─────────────────┴─────────────────┴─────────────────┘ │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│  GOAL: Find the right balance for your organization's context               │
└─────────────────────────────────────────────────────────────────────────────┘
```

Different organizational contexts require different adoption strategies:

**Startup Approach:**
- Move fast, iterate quickly
- Trust developers to self-govern
- Lightweight tracking (commit tags, weekly review)
- Post-hoc audit rather than pre-approval
- Goal: Maximum velocity with acceptable risk

**Enterprise Approach:**
- Formal governance and approval workflows
- Multiple review gates for different risk levels
- Comprehensive audit trails
- Security scanning and compliance checks
- Goal: Controlled adoption with minimal risk

Most organizations fall somewhere on the spectrum and should calibrate their approach based on:
- Regulatory requirements
- Risk tolerance
- Team size and maturity
- Existing governance infrastructure

---

## Key Takeaways

1. **SE4H + SE4A** represents the dual nature of modern development teams
2. **N-to-N collaboration** scales beyond individual human-AI pairing to team-scale coordination
3. **Shared context files** (AGENTS.md, CLAUDE.md, MentorScript) ensure consistency
4. **MentorScript** captures team culture and judgment as "mentorship-as-code"
5. **Cross-team coordination** uses federated context files and shared learning
6. **Version-control MCP config** so all team members have same tools
7. **Governance scales** from light (startups) to formal (enterprise)
8. **Track AI metrics** to understand contribution and quality
9. **Change management** follows predictable stages from exploration to integration
10. **Clear workflows** define how humans and AI collaborate
11. **Legal/compliance** considerations need explicit attention
12. **Onboarding** should include AI-assisted development practices

---

## Practical Exercise: Team Standards Design

Design AI development standards for your team (real or hypothetical):

### Part 1: Context Files
1. Draft a team AGENTS.md
2. Create a CLAUDE.md template
3. Define what goes in each

### Part 2: Governance
1. Define risk levels for different code areas
2. Specify review requirements per level
3. Create approval workflow

### Part 3: Metrics
1. Decide what to track
2. Design commit message convention
3. Plan how to report metrics

### Part 4: Workflow
1. Map your team's typical workflow
2. Identify where AI fits
3. Define handoff points between human and AI

---

## Next Step

[Step 14: Future of AI-Assisted Development](14-future-of-ai-assisted-development.md) — Explore emerging trends and prepare for what's next.

---

## Sources

### Primary References

1. **Academic Papers**
   - Chen, M., et al. "Structured Agentic Software Engineering (SASE)." arXiv:2509.06216v2 (2025)
     - SE4H/SE4A framework, N-to-N collaboration patterns
   - "Software Engineering for LLMs: Research Status and Challenges." arXiv:2506.23762 (2025)

2. **Anthropic Resources**
   - "How Anthropic Teams Use Claude Code" (2025) - Internal practices document
   - "Claude Code Best Practices" - https://www.anthropic.com/engineering/claude-code-best-practices

3. **Stanford CS146S: The Modern Software Developer**
   - Week 4: Team Patterns for AI-Assisted Development
   - Week 7: Governance and Compliance

4. **Specifications and Standards**
   - AGENTS.md Specification: https://agents.md
   - Model Context Protocol: https://modelcontextprotocol.io

5. **Industry Resources**
   - HumanLayer. "Writing a Good CLAUDE.md" - https://www.humanlayer.dev/blog/writing-a-good-claude-md
   - Builder.io. "AGENTS.md Best Tips" - https://www.builder.io/blog/agents-md

---

## Further Reading

1. **AGENTS.md Specification** - The official specification for creating context files that guide AI behavior in repositories.
   https://agents.md

2. **SASE Paper: Structured Agentic Software Engineering** (arXiv:2509.06216v2) - Comprehensive framework for team-scale AI development including SE4H/SE4A concepts.
   https://arxiv.org/abs/2509.06216

3. **"Writing a Good CLAUDE.md"** - HumanLayer's practical guide to creating effective Claude-specific context files.
   https://www.humanlayer.dev/blog/writing-a-good-claude-md

4. **Model Context Protocol Documentation** - Official docs for MCP, essential for team-shared tool configurations.
   https://modelcontextprotocol.io/

5. **"Claude Code Best Practices"** - Anthropic's official guidance that includes team workflow recommendations.
   https://www.anthropic.com/engineering/claude-code-best-practices
