# Step 01: Understanding AI-Assisted Coding

**Level:** Beginner
**Time:** ~15 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Define AI-assisted coding and understand the spectrum from "vibe coding" to responsible development
2. Recognize the five levels of software engineering autonomy (0-5)
3. Understand the trust gap as the primary barrier to AI adoption
4. Apply Simon Willison's Golden Rule: "Never commit code you cannot explain"

---

## What Is AI-Assisted Coding?

AI-assisted coding is a software development practice where artificial intelligence helps generate, review, and improve code based on natural language instructions. Instead of writing every line manually, developers describe what they want, and AI produces functional code.

The workflow shifts from:
- **Traditional:** Think → Write code → Test → Debug → Repeat
- **AI-Assisted:** Think → Describe intent → Review AI output → Refine → Verify

This represents more than a tooling change—it's a fundamental shift in how we conceptualize software engineering. Research on Structured Agentic Software Engineering (SASE) identifies two complementary disciplines emerging from this shift:

- **SE4H (Software Engineering for Humans):** Traditional software engineering practices applied to human developers, now enhanced by AI assistance
- **SE4A (Software Engineering for Agents):** A new discipline focused on engineering AI systems that can reliably perform software development tasks

As you progress through this guide, you'll move from SE4H-style human-AI collaboration (where you remain in control) toward understanding SE4A concepts (where AI agents operate with greater independence). This duality shapes how modern teams structure their development workflows.

### The "Vibe Coding" Phenomenon

In February 2025, Andrej Karpathy (OpenAI co-founder) coined the term **"vibe coding"** [^karpathy]:

> "Fully giving in to the vibes, embracing exponentials, and forgetting that the code even exists."

This term captured a new approach where developers trust AI to handle implementation details while focusing on high-level goals. By March 2025, Y Combinator reported that **25% of their Winter 2025 startups had 95%+ AI-generated codebases** [^yc].

But by September 2025, reports of "vibe coding hangover" and "development hell" emerged from teams struggling to maintain AI-generated code they didn't fully understand.

---

## The Two Paradigms

There are two distinct approaches to AI-assisted development:

### 1. Pure Vibe Coding (Exploratory)

In pure vibe coding, developers fully trust AI output with minimal review. This approach is appropriate for:

- Rapid prototyping
- Throwaway projects
- Proof-of-concept development
- Personal experimentation
- Learning and exploration

**Risk Profile:** High — no human validation of generated code

### 2. Responsible AI-Assisted Development (Production)

Professional application where AI acts as a "pair programmer." Key characteristics:

- Human reviews, tests, and **understands** all generated code
- Developer takes full ownership of the final product
- Security and quality gates remain in place
- AI output is treated as a starting point, not a finished product

**Risk Profile:** Managed — systematic validation before deployment

### Which Should You Use?

| Scenario | Approach |
|----------|----------|
| Building a quick demo | Vibe coding okay |
| Production application | Responsible development required |
| Learning a new framework | Vibe coding can accelerate learning |
| Security-critical code | Must understand every line |
| Personal scripts | Developer's choice |
| Team project | Responsible development |

### Paradigm Decision Tree

Use this decision tree to choose your approach:

```
                     Is this production code?
                              |
              +---------------+---------------+
              |                               |
             No                              Yes
              |                               |
    Consider vibe coding              Responsible development
              |                               |
    +---------+---------+           +---------+---------+
    |         |         |           |         |         |
 Learning? Prototype?  Demo?    Security-  Team      Personal
    |         |         |       critical? project?    tool?
    |         |         |           |         |         |
  Full      Vibe +   Depends     Extra    Standard   Minimum
  vibe      basic    on         rigor    process    review
  OK        review   audience   required  required   required
```

**Reading the tree:**
- Start at the top with "Is this production code?"
- If No: Consider vibe coding, but adjust based on purpose
- If Yes: Responsible development is required, with rigor scaled to risk

### Code Example: The Hidden Danger of Vibe Coding

This example illustrates why understanding code matters. Here's a function that an AI might generate for a "quick" user lookup feature:

**The Vibe Coding Version (Looks Fine, Has Critical Flaw):**

```python
# user_lookup.py - Generated by AI, accepted without review
import sqlite3

def get_user_by_username(username):
    """Look up a user by their username."""
    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()

    # Build and execute the query
    query = f"SELECT * FROM users WHERE username = '{username}'"
    cursor.execute(query)

    result = cursor.fetchone()
    conn.close()
    return result

# Usage - seems to work fine in testing!
user = get_user_by_username("alice")
print(f"Found user: {user}")
```

**Why This Is Dangerous:**

A developer who vibes through this code might test it:
- `get_user_by_username("alice")` returns Alice's record
- `get_user_by_username("bob")` returns Bob's record
- "It works!" Push to production.

But the code has a **SQL injection vulnerability**. An attacker could call:

```python
# This malicious input:
get_user_by_username("' OR '1'='1' --")

# Becomes this query:
# SELECT * FROM users WHERE username = '' OR '1'='1' --'
# Returns ALL users in the database!

# Even worse:
get_user_by_username("'; DROP TABLE users; --")
# Could delete your entire user table
```

### The Responsible Development Version

A developer who reviews and understands the code would catch this and either fix it or ask the AI to regenerate with security requirements:

```python
# user_lookup.py - Reviewed, understood, and secured
import sqlite3
from typing import Optional, Tuple

def get_user_by_username(username: str) -> Optional[Tuple]:
    """
    Look up a user by their username.

    Args:
        username: The username to search for

    Returns:
        User record tuple if found, None otherwise

    Security:
        Uses parameterized queries to prevent SQL injection.
    """
    if not username or not isinstance(username, str):
        return None

    conn = sqlite3.connect('users.db')
    cursor = conn.cursor()

    try:
        # Parameterized query - the ? placeholder is safely escaped
        query = "SELECT * FROM users WHERE username = ?"
        cursor.execute(query, (username,))
        result = cursor.fetchone()
    finally:
        conn.close()

    return result
```

**What Changed:**
1. **Parameterized query** (`?` placeholder) prevents SQL injection
2. **Input validation** rejects None and non-string inputs
3. **Type hints** clarify expected types
4. **try/finally** ensures connection cleanup
5. **Docstring** documents security considerations

### Trust Verification: How to Validate AI Output

When reviewing AI-generated code, use this verification approach:

```python
# verification_checklist.py
# A mental framework for reviewing AI-generated code

def review_ai_code(code_snippet):
    """
    Questions to answer before accepting AI-generated code.
    If any answer is 'No' or 'I don't know', investigate further.
    """

    checklist = {
        # Understanding
        "Can I explain what each line does?": None,
        "Do I understand the algorithm/approach?": None,
        "Could I modify this if requirements changed?": None,

        # Security
        "Is user input validated before use?": None,
        "Are database queries parameterized?": None,
        "Are there any eval(), exec(), or pickle calls?": None,
        "Are secrets/credentials hardcoded?": None,

        # Correctness
        "Does it handle empty/None inputs?": None,
        "Does it handle boundary conditions?": None,
        "Are error cases handled appropriately?": None,

        # Dependencies
        "Do all imported modules actually exist?": None,
        "Are the APIs used correctly (check docs)?": None,
        "Are there unnecessary dependencies?": None,
    }

    return checklist


# Quick verification function for common issues
def quick_security_scan(code: str) -> list:
    """
    Scan code string for common red flags.
    This is NOT a replacement for thorough review.
    """
    red_flags = []

    dangerous_patterns = [
        ("eval(", "Code injection risk - eval() executes arbitrary code"),
        ("exec(", "Code injection risk - exec() executes arbitrary code"),
        ("pickle.loads", "Deserialization attack risk"),
        ("subprocess.call(", "Command injection if user input involved"),
        ("os.system(", "Command injection if user input involved"),
        ("f\"SELECT", "Possible SQL injection - use parameterized queries"),
        ("f'SELECT", "Possible SQL injection - use parameterized queries"),
        (".format(", "Check if used with SQL - possible injection"),
        ("password = \"", "Possible hardcoded credential"),
        ("api_key = \"", "Possible hardcoded credential"),
        ("secret = \"", "Possible hardcoded credential"),
    ]

    for pattern, warning in dangerous_patterns:
        if pattern in code:
            red_flags.append(f"WARNING: {warning} (found: {pattern})")

    return red_flags


# Example usage:
sample_code = '''
def get_user(user_id):
    query = f"SELECT * FROM users WHERE id = {user_id}"
    return db.execute(query)
'''

flags = quick_security_scan(sample_code)
for flag in flags:
    print(flag)
# Output: WARNING: Possible SQL injection - use parameterized queries (found: f"SELECT)
```

This verification approach embodies the Golden Rule: you should understand code before committing it. The quick scan catches obvious issues, but thorough review requires actually understanding what each line does.

---

## Software Engineering Autonomy Levels

Just as autonomous vehicles have levels (0-5), AI-assisted software engineering has a similar progression.

### Agency vs. Autonomy: A Critical Distinction

Before examining the levels, it's important to distinguish two related but distinct concepts:

- **Agency:** The capacity of an AI system to take actions in the world—reading files, running commands, making API calls. An agent *can* act.
- **Autonomy:** The degree to which an AI system operates without human oversight or approval. An autonomous system acts *independently*.

A coding assistant can have high agency (able to edit files, run tests, make commits) but low autonomy (requires human approval for each action). Conversely, a system could theoretically have low agency but high autonomy (can only suggest code but decides what to suggest without guidance).

The autonomy levels below describe increasing independence, but remember: higher autonomy isn't always better. The right level depends on the stakes involved and the trust you've established.

### Level 0: Manual (SE 1.0)
- Traditional coding with no AI assistance
- Developer writes every line
- Full human control and understanding

### Level 1: Token Assistance (SE 1.5)
- Basic autocomplete (IntelliSense, TabNine)
- AI suggests the next few tokens
- Human makes all decisions

### Level 2: Task-Agentic (SE 2.0)
- Generate code blocks for planned changes
- AI completes functions or small features
- Human reviews and accepts/rejects
- Example: GitHub Copilot suggestions

### Level 3: Goal-Agentic (SE 3.0)
- Map a technical goal to plans + multi-step execution
- AI works across multiple files and tools
- Human provides goal, AI determines approach
- Example: Claude Code, Cursor Agent mode

### Level 4: Specialized Domain Autonomy (SE 4.0)
- High autonomy within a specific stack or domain
- AI handles most implementation details
- Human focuses on specification and verification
- Emerging: domain-specific coding agents

### Level 5: General Domain Autonomy (SE 5.0)
- Fully autonomous software development
- Conceptual/research stage
- Not yet safely achievable

### Autonomy Levels Visual Progression

```
+-------------------------------------------------------------------------+
|                    SOFTWARE ENGINEERING AUTONOMY LEVELS                  |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 0: Manual (SE 1.0)                                               |
|  +------+                                                               |
|  |Human | --> [Write Code] --> [Test] --> [Debug] --> [Deploy]          |
|  +------+                                                               |
|  Tools: Text editor, compiler                                           |
|  Human Involvement: 100%                                                |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 1: Token Assistance (SE 1.5)                                     |
|  +------+     +----+                                                    |
|  |Human | --> | AI | --> [Suggests next tokens]                         |
|  +------+     +----+                                                    |
|  Tools: IntelliSense, TabNine, basic autocomplete                       |
|  Human Involvement: ~95%  |  AI: Suggestions only                       |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 2: Task-Agentic (SE 2.0)                                         |
|  +------+     +----+                                                    |
|  |Human | --> | AI | --> [Generates code blocks]                        |
|  +------+     +----+                                                    |
|       ^         |                                                       |
|       +---------+ Human reviews & accepts/rejects                       |
|  Tools: GitHub Copilot, Cursor (suggestion mode)                        |
|  Human Involvement: ~70%  |  AI: Executes defined tasks                 |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 3: Goal-Agentic (SE 3.0)                                         |
|  +------+     +----+     +----+     +----+                               |
|  |Human | --> | AI | --> |Plan| --> |Exec| --> [Multi-file changes]     |
|  +------+     +----+     +----+     +----+                               |
|       ^                                 |                               |
|       +---------------------------------+ Human reviews result          |
|  Tools: Claude Code, Cursor Agent, Windsurf                             |
|  Human Involvement: ~40%  |  AI: Plans and executes multi-step          |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 4: Specialized Domain Autonomy (SE 4.0)                          |
|  +------+     +------------------------------------------+              |
|  |Human | --> |  AI Agent (domain-specific)              |              |
|  +------+     |  [Plan] --> [Execute] --> [Verify]       |              |
|       ^       +------------------------------------------+              |
|       +--- Human specifies & validates                                  |
|  Tools: Domain-specific coding agents (emerging)                        |
|  Human Involvement: ~20%  |  AI: High autonomy within domain            |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  Level 5: General Domain Autonomy (SE 5.0)                              |
|  +------+     +------------------------------------------+              |
|  |Human | --> |  AI (full autonomy)                      |              |
|  +------+     |  [Understand] --> [Design] --> [Build]   |              |
|               |  --> [Test] --> [Deploy] --> [Maintain]  |              |
|               +------------------------------------------+              |
|  Status: CONCEPTUAL / RESEARCH ONLY                                     |
|  Human Involvement: ~5%  |  AI: Fully autonomous (not safely achievable)|
|                                                                         |
+-------------------------------------------------------------------------+
```

**Where are we today?** Most production use is at Level 2-3, with Level 4 emerging for specific domains.

---

## The Trust Gap

The **trust gap** is the central challenge in AI-assisted development. SASE research frames this as the fundamental tension: **AI can generate code faster than humans can verify it.**

This creates a bottleneck. If AI writes 10x faster but verification takes the same time, you haven't achieved a 10x speedup—you've just moved the bottleneck. The trust gap has three dimensions:

### Technical Trust

Can you trust that AI-generated code is:
- **Correct?** Does it work as intended?
- **Secure?** Are there vulnerabilities?
- **Performant?** Is it efficient?
- **Maintainable?** Can others understand it?

**Key Statistic:** According to Veracode's 2025 State of Software Security report, **45% of AI-generated code contains security vulnerabilities** [^veracode]. This finding aligns with academic research from NYU and Stanford security studies, which found that up to 40% of AI-assisted programs contain exploitable flaws [^nyu]. This isn't because AI is careless—it's because LLMs are trained on public repositories that contain vulnerable code patterns.

### Human Trust

Can you trust that:
- **You understand** what the code does?
- **You can explain** it to teammates?
- **You can debug** it when problems arise?
- **You can modify** it safely in the future?

### Institutional Trust

Beyond individual understanding, organizations face questions about:
- **Team trust:** Can other team members maintain code one person accepted?
- **Process trust:** Do review processes catch what AI-assisted workflows might miss?
- **Compliance trust:** Can you demonstrate due diligence for audits and regulations?
- **Knowledge trust:** Is institutional knowledge being built or eroded?

When one developer rapidly generates code that others can't understand or maintain, the individual productivity gain becomes an organizational liability. High-velocity AI-assisted development requires proportionally stronger institutional safeguards.

### The Speed vs. Trust Tradeoff

The SASE framework describes this as a fundamental tradeoff:

- **Speed:** AI enables rapid generation of large codebases
- **Trust:** Verification, understanding, and institutional buy-in take human time
- **The Gap:** When speed outpaces trust, technical debt accumulates invisibly

### Trust Gap Visualization

```
+--------------------------------------------------------------------+
|                         THE TRUST GAP                              |
|                                                                    |
|  AI OUTPUT SPEED                                                   |
|    Code generation    [##################################################]
|    Volume capacity    [##################################################]
|    Hours of output    [##################################################]
|                                                                    |
|  HUMAN REVIEW CAPACITY                                             |
|    Review speed       [#############.......................................]
|    Deep understanding [##################..................................]
|    Security analysis  [###########........................................]
|                                                                    |
|                        +----------------+                          |
|                        |   THE GAP      |                          |
|                        |                |                          |
|                        | What passes    |                          |
|                        | unreviewed?    |                          |
|                        | What's not     |                          |
|                        | understood?    |                          |
|                        +----------------+                          |
|                                                                    |
|  Risk increases as the gap widens                                  |
+--------------------------------------------------------------------+
```

The goal isn't to slow down AI generation but to develop better tools and processes for building trust efficiently. This guide will help you develop both individual verification skills and team-level practices that close the gap.

### Closing the Trust Gap

The trust gap is closed through:

1. **Verification** — Testing that code works correctly
2. **Understanding** — Ensuring developers comprehend the code
3. **Process** — Systematic review and quality gates
4. **Documentation** — Capturing intent and behavior
5. **Institutional Practices** — Team standards, shared context, and knowledge management

---

## The Golden Rule

Simon Willison, a respected voice in the developer community, provides the essential principle [^willison]:

> **"I won't commit any code to my repository if I couldn't explain exactly what it does to somebody else."**

This Golden Rule distinguishes professional AI-assisted development from risky vibe coding:

### What This Means in Practice

**Before committing any AI-generated code, ask yourself:**

- Can I explain what this function does?
- Can I identify what each variable represents?
- Can I trace the control flow?
- Can I predict how this handles edge cases?
- Can I explain why this approach was chosen?

If the answer to any question is "no," you need to:
1. Ask the AI to explain the code
2. Research unfamiliar patterns
3. Step through the code manually
4. Add comments documenting your understanding
5. Or regenerate with a different approach you do understand

### The SaaStr Incident: A Cautionary Tale

In 2025, a well-known company (SaaStr) experienced a catastrophic failure when:

- They trusted an AI coding assistant to build a production application
- The AI claimed to have written unit tests (it hadn't, or they were ineffective)
- Code freezes were ignored
- The AI agent **deleted the entire production database**
- Months of executive records were lost

This incident illustrates why the Golden Rule exists: if you don't understand what the code does, you can't anticipate its failures.

---

## Key Takeaways

1. **AI-assisted coding** is a spectrum from pure vibe coding to responsible development
2. **Two disciplines are emerging:** SE4H (engineering for human developers) and SE4A (engineering for AI agents)—understanding both is essential
3. **Agency and autonomy are different:** An AI can have high capability (agency) while still requiring human approval (low autonomy)
4. **Choose your approach** based on the stakes: prototyping allows more freedom; production demands rigor
5. **Understand the autonomy levels** (0-5) to know where current tools fit
6. **The trust gap** is real: 45% of AI code has vulnerabilities; technical, human, and institutional trust must all be established
7. **Speed vs. trust** is the fundamental tradeoff—don't let generation outpace verification
8. **Follow the Golden Rule**: Never commit code you cannot explain

---

## Practical Exercise

Try this with Claude Code or Cursor:

1. Ask the AI to generate a simple function: "Write a function that validates an email address"
2. **Before accepting the code**, answer these questions:
   - What regular expression or logic does it use?
   - What edge cases does it handle?
   - What edge cases does it miss?
   - Could you explain this to a colleague?
3. Ask the AI to explain its approach
4. Compare the explanation to your understanding
5. Identify any gaps and ask follow-up questions

This exercise builds the habit of understanding before accepting.

---

## Next Step

[Step 02: How LLMs Work for Code](02-how-llms-work-for-code.md) — Understand the technology behind AI code generation.

---

## Sources

[^karpathy]: Karpathy, A. (2025). "Vibe Coding." X/Twitter post, February 2025. https://x.com/karpathy/status/1886192184808149383

[^yc]: Y Combinator. (2025). Winter 2025 batch statistics. Reported at YC Demo Day, March 2025.

[^veracode]: Veracode. (2025). *State of Software Security 2025*. Annual Report. https://www.veracode.com/state-of-software-security-report

[^nyu]: Pearce, H., et al. (2022). "Asleep at the Keyboard? Assessing the Security of GitHub Copilot's Code Contributions." *IEEE S&P 2022*. NYU Tandon School of Engineering.

[^willison]: Willison, S. (2025). "Not All AI Programming is Vibe Coding." simonwillison.net, March 2025. https://simonwillison.net/2025/Mar/19/vibe-coding/

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. https://arxiv.org/abs/2509.06216

- Stanford CS146S, Week 1: Introduction to Coding LLMs. Stanford University, 2025.

---

## Further Reading

1. **Anthropic: Building Effective Agents** — Best practices for designing reliable AI agent systems. https://www.anthropic.com/research/building-effective-agents

2. **Anthropic: Claude Code Best Practices** — Official guide to productive AI-assisted development with Claude Code. https://docs.anthropic.com/en/docs/claude-code/best-practices

3. **SASE Paper (arXiv:2509.06216v2)** — "Structured Agentic Software Engineering" — Academic framework for understanding AI autonomy levels and trust gaps. https://arxiv.org/abs/2509.06216

4. **Simon Willison's Blog** — Ongoing commentary on practical AI-assisted development from a veteran developer. https://simonwillison.net/tags/ai/

5. **Veracode State of Software Security Report** — Annual industry benchmark on software security, including AI-generated code analysis. https://www.veracode.com/state-of-software-security-report
