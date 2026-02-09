# Step 02: How LLMs Work for Code

**Level:** Beginner
**Time:** ~20 minutes

## Learning Objectives

By the end of this step, you will be able to:

1. Understand token prediction and why LLMs can generate code
2. Recognize context windows and their limitations (including "context rot")
3. Understand non-determinism and temperature effects
4. Grasp why "treat AI output as untrusted input" is essential

---

In [Step 01](01-understanding-ai-assisted-coding.md), we introduced the spectrum from "vibe coding" to responsible AI-assisted development and established the Golden Rule: "Never commit code you cannot explain." Now we'll explore *how* the AI actually generates code, giving you the mental models needed to work with it effectively.

## What Is a Large Language Model (LLM)?

A **Large Language Model** is an AI system trained on massive amounts of text to predict what comes next in a sequence. When you use Claude, GPT-4, or Gemini for coding, you're using an LLM.

### How Training Works

LLMs learn through a multi-stage process:

**Stage 1: Pretraining (Base Model)**
1. Processing billions of text documents (including code from GitHub, documentation, tutorials)
2. Learning statistical patterns in how text and code are structured
3. Building an internal model of language, logic, and programming concepts

At this stage, the model is a "base model"—it can continue text but isn't particularly helpful or safe.

**Stage 2: Instruction Tuning and RLHF**

Base models are transformed into useful assistants through additional training:

- **Instruction Tuning:** The model is trained on examples of instructions paired with good responses, teaching it to follow directions rather than just complete text
- **RLHF (Reinforcement Learning from Human Feedback):** Human evaluators rate model outputs, and these ratings train the model to produce responses humans prefer

This is why modern coding assistants are helpful by default—they've been specifically trained to:
- Follow instructions about code style and requirements
- Explain their reasoning when asked
- Admit uncertainty rather than confidently hallucinate
- Refuse harmful requests

**Key insight:** LLMs don't "understand" code the way humans do—they recognize and reproduce patterns they've seen in training data. However, instruction tuning makes them substantially better at following your specific requirements.

---

## Token Prediction: The Core Mechanism

### What Are Tokens?

LLMs don't process text character by character. They break text into **tokens**—chunks that might be whole words, parts of words, or punctuation.

```
"function calculateTotal(items)"

Might tokenize as:
["function", " calculate", "Total", "(", "items", ")"]
```

Code often tokenizes differently than natural language because of special characters and syntax.

### The Prediction Process

When you prompt an AI to write code:

1. Your prompt is converted to tokens
2. The model predicts the most likely next token
3. That token is added, and the model predicts the next
4. This repeats until the response is complete

```
Prompt: "Write a Python function to calculate the sum of a list"

Model generates token by token:
"def" → " sum" → "_list" → "(" → "numbers" → ")" → ":" → "\n" → ...
```

### Token Prediction Diagram

```
Input: "def calculate_"
         |
         v
+--------------------------------------+
|              LLM                     |
|                                      |
|  +------------------------------+    |
|  |     Pattern Recognition      |    |
|  |                              |    |
|  |  Analyzes training data      |    |
|  |  patterns for code that      |    |
|  |  starts with "def calculate_"|    |
|  +---------------+--------------+    |
|                  |                   |
|                  v                   |
|  +------------------------------+    |
|  |   Probability Distribution   |    |
|  |                              |    |
|  |   "sum"      ████████ 23%    |    |
|  |   "total"    ██████   18%    |    |
|  |   "average"  ████     12%    |    |
|  |   "price"    ███       9%    |    |
|  |   "area"     ██        6%    |    |
|  |   "tax"      ██        5%    |    |
|  |   "score"    ██        4%    |    |
|  |   [other]    ████     23%    |    |
|  +---------------+--------------+    |
|                  |                   |
|                  v                   |
|         Select based on              |
|         temperature setting          |
+--------------------------------------+
         |
         v
Output: "sum" (selected)

Next iteration: "def calculate_sum" --> predict next token...
```

**Key insight:** The model doesn't "understand" that you need a sum function. It predicts "sum" because that token frequently follows "def calculate_" in its training data.

### Why This Matters for Code

Because LLMs predict based on patterns:

- **Common patterns work well:** Standard functions, typical API usage, and well-documented libraries generate reliable code
- **Unusual patterns may fail:** Niche libraries, custom conventions, or novel approaches may produce hallucinated code
- **Training data matters:** If vulnerable code patterns are common in training data, the model may reproduce them

### Code Example: Visualizing Tokenization

Understanding how code gets tokenized helps you predict LLM behavior. Here's how to visualize tokenization using the `tiktoken` library (used by many LLMs):

```python
# tokenization_demo.py
# Install: pip install tiktoken

import tiktoken

def visualize_tokens(text: str, encoding_name: str = "cl100k_base"):
    """
    Visualize how text is broken into tokens.

    Args:
        text: The text to tokenize
        encoding_name: The tokenizer to use (cl100k_base is used by GPT-4/Claude)
    """
    enc = tiktoken.get_encoding(encoding_name)
    tokens = enc.encode(text)

    print(f"Original text ({len(text)} chars):")
    print(f"  '{text}'")
    print(f"\nToken count: {len(tokens)}")
    print(f"\nTokens (ID -> text):")

    for token_id in tokens:
        token_text = enc.decode([token_id])
        # Show whitespace explicitly
        display_text = repr(token_text)
        print(f"  {token_id:6d} -> {display_text}")

    return tokens


# Example 1: Natural language vs code
print("=" * 50)
print("NATURAL LANGUAGE:")
print("=" * 50)
visualize_tokens("The quick brown fox jumps")

print("\n" + "=" * 50)
print("CODE - FUNCTION DEFINITION:")
print("=" * 50)
visualize_tokens("def calculate_total(items):")

print("\n" + "=" * 50)
print("CODE - WITH SPECIAL CHARACTERS:")
print("=" * 50)
visualize_tokens("user_data['email'] = request.json.get('email', '')")
```

**Example Output:**

```
==================================================
NATURAL LANGUAGE:
==================================================
Original text (25 chars):
  'The quick brown fox jumps'

Token count: 5

Tokens (ID -> text):
     791 -> 'The'
    4062 -> ' quick'
   14198 -> ' brown'
   39935 -> ' fox'
   35308 -> ' jumps'

==================================================
CODE - FUNCTION DEFINITION:
==================================================
Original text (26 chars):
  'def calculate_total(items):'

Token count: 7

Tokens (ID -> text):
     755 -> 'def'
   11294 -> ' calculate'
   10304 -> '_total'
       8 -> '('
   12408 -> 'items'
      15 -> '):'
       8 -> ''

==================================================
CODE - WITH SPECIAL CHARACTERS:
==================================================
Original text (52 chars):
  "user_data['email'] = request.json.get('email', '')"

Token count: 17

Tokens (ID -> text):
    1849 -> 'user'
    2657 -> '_data'
     681 -> '[\''
    2609 -> 'email'
   13162 -> '\'] = '
    4993 -> 'request'
    4331 -> '.json'
     815 -> '.get'
     493 -> '(\''
    2609 -> 'email'
   1200 -> '\', '
     820 -> '\'\''
      15 -> ')'
```

**Key Insights from Tokenization:**

1. **Common words/patterns tokenize efficiently:** `def`, `function`, `return` are single tokens
2. **Underscores often combine with words:** `_total`, `_data` are single tokens
3. **Unusual variable names use more tokens:** `myWeirdVar123` might be 4+ tokens
4. **Special characters often standalone:** Brackets, parentheses get their own tokens
5. **Token count affects context usage:** More tokens = less room in context window

---

## Context Windows

### What Is a Context Window?

The **context window** is the maximum amount of text (measured in tokens) an LLM can "see" at once. This includes both your prompt and the model's response.

### Current Context Window Sizes (2026)

| Model | Context Window |
|-------|---------------|
| Claude 3.5/4 | 200,000 tokens |
| GPT-4 Turbo | 128,000 tokens |
| Gemini 1.5 Pro | 1,000,000+ tokens |
| Claude with extended context | Up to 1,000,000 tokens |

**Rough conversion:** 1 token ≈ 4 characters in English, so 200K tokens ≈ 800K characters ≈ a medium-sized codebase.

### Why Context Limits Matter

When working on a large codebase:
- The AI cannot see all files simultaneously
- It must choose which context to include
- Information outside the window is "invisible" to the model
- Long conversations may lose early context

### Context Rot

**Context rot** is a phenomenon where AI performance degrades as conversations grow longer:

1. **Early messages become distant:** The model pays less attention to information far from the end of context
2. **Contradictions accumulate:** Instructions from different parts of the conversation may conflict
3. **Focus drifts:** The model may forget earlier goals or constraints

### Context Window Lifecycle Diagram

```
+-------------------------------------------------------------------------+
|                    CONTEXT WINDOW OVER TIME                             |
+-------------------------------------------------------------------------+
|                                                                         |
|  CONVERSATION START                                                     |
|  +-------+--------+                                                     |
|  |System | User   |                                                     |
|  |Prompt | Prompt |  Unused Space                                       |
|  +-------+--------+.................................................    |
|  [████████........................................................] 10% |
|  Attention: Focused, all context equally weighted                       |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  MID-CONVERSATION                                                       |
|  +-------+------------------+----------+                                |
|  |System | Conversation     | Current  |                                |
|  |Prompt | History          | Task     |                                |
|  +-------+------------------+----------+.....................           |
|  [████████████████████████████████████.......................] 60%      |
|  Attention: Still good, but early messages getting distant              |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  CONTEXT NEARLY FULL (Context Rot Begins)                               |
|  +-------+----------------------------------+---------+                 |
|  |System | Long Conversation History...     | Current |                 |
|  |Prompt | (may contain contradictions)     | Task    |                 |
|  +-------+----------------------------------+---------+                 |
|  [████████████████████████████████████████████████████████████] 100%    |
|  Attention: Early messages may be "forgotten" or deprioritized          |
|             Contradictions may cause inconsistent behavior              |
|                                                                         |
|  WARNING SIGNS:                                                         |
|  - AI forgets earlier instructions                                      |
|  - Responses become less coherent                                       |
|  - AI contradicts its earlier statements                                |
|  - Quality of output degrades                                           |
|                                                                         |
|  ACTIONS:                                                               |
|  - Use /compact to summarize and reset                                  |
|  - Use /clear to start fresh for new tasks                              |
|  - Repeat critical instructions explicitly                              |
|                                                                         |
+-------------------------------------------------------------------------+
```

**Mitigation strategies:**
- Start fresh conversations for new tasks (`/clear` in Claude Code)
- Use `/compact` to summarize and preserve key information
- Keep context focused on the current task
- Repeat important instructions when they become distant

---

## Non-Determinism and Temperature

### Non-Deterministic Output

Unlike traditional programs that produce identical output for identical input, LLMs are **non-deterministic**—the same prompt can produce different responses.

```
Prompt: "Write a hello world function in Python"

Response 1:
def hello():
    print("Hello, World!")

Response 2:
def hello_world():
    return "Hello, World!"
```

Both are valid, but they're different.

### Temperature

**Temperature** is a parameter controlling how "creative" or "random" the model's responses are:

| Temperature | Behavior | Good For |
|-------------|----------|----------|
| 0.0 | Most deterministic, "safest" predictions | Consistent code generation, factual tasks |
| 0.5 | Balanced | Most coding tasks |
| 1.0 | More variation, creative responses | Brainstorming, creative writing |
| > 1.0 | Highly random | Rarely useful |

**For coding:** Lower temperatures (0.0-0.5) typically produce more reliable code.

### Why Non-Determinism Matters

- **Testing AI-generated code:** The same prompt might work today but produce buggy code tomorrow
- **Reproducibility:** You may not be able to recreate a previous result
- **Iteration:** Sometimes regenerating gets a better result; sometimes it gets worse

### Code Example: Temperature Effects on Code Generation

Here's a conceptual example showing how temperature affects code output. When using an API directly, you can control temperature:

```python
# temperature_comparison.py
# Conceptual example - actual API syntax varies by provider

"""
The same prompt at different temperatures produces different results:

PROMPT: "Write a Python function to calculate factorial"

-------------------------------------------
TEMPERATURE 0.0 (Most deterministic)
-------------------------------------------
Multiple calls produce nearly identical output:

Call 1:
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

Call 2:
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

Call 3:
def factorial(n):
    if n <= 1:
        return 1
    return n * factorial(n - 1)

Result: Consistent, predictable code.

-------------------------------------------
TEMPERATURE 0.5 (Balanced)
-------------------------------------------
Outputs vary slightly in style and approach:

Call 1:
def factorial(n):
    if n == 0 or n == 1:
        return 1
    return n * factorial(n - 1)

Call 2:
def factorial(n: int) -> int:
    '''Calculate the factorial of n.'''
    if n <= 1:
        return 1
    return n * factorial(n - 1)

Call 3:
def factorial(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

Result: Valid variations, different styles/approaches.

-------------------------------------------
TEMPERATURE 1.0 (More creative/random)
-------------------------------------------
Outputs vary significantly, may include unusual approaches:

Call 1:
def factorial(n):
    import math
    return math.factorial(n)  # Uses stdlib instead

Call 2:
def factorial(num):
    # Using reduce for a functional approach
    from functools import reduce
    return reduce(lambda x, y: x * y, range(1, num + 1), 1)

Call 3:
def factorial(n):
    '''Calculate n! using memoization for efficiency'''
    cache = {0: 1, 1: 1}
    def helper(x):
        if x not in cache:
            cache[x] = x * helper(x - 1)
        return cache[x]
    return helper(n)

Result: Creative variations, some unexpected approaches.
"""

# Practical guidance for temperature selection:

TEMPERATURE_GUIDE = {
    0.0: {
        "use_for": [
            "Code that must be consistent across runs",
            "Factual/documentation tasks",
            "When you want the most common/standard pattern",
        ],
        "avoid_for": [
            "Brainstorming alternative approaches",
            "Creative problem solving",
        ]
    },
    0.3: {
        "use_for": [
            "Most production code generation",
            "Bug fixes",
            "Refactoring",
        ],
        "avoid_for": [
            "Highly creative tasks",
        ]
    },
    0.7: {
        "use_for": [
            "Exploring different approaches",
            "Getting varied suggestions",
            "Learning/educational contexts",
        ],
        "avoid_for": [
            "Critical production code",
            "Security-sensitive code",
        ]
    },
    1.0: {
        "use_for": [
            "Brainstorming",
            "Creative writing/content",
            "Exploring edge cases",
        ],
        "avoid_for": [
            "Most coding tasks",
            "Anything requiring reliability",
        ]
    }
}

# Note: Most AI coding tools (Claude Code, Cursor) use optimized
# temperature settings automatically. This matters more when
# using APIs directly or configuring advanced settings.
```

### Reproducibility Challenges in Practice

The SASE (Structured Agentic Software Engineering) framework [^sase] identifies reproducibility as a significant barrier to AI adoption in professional contexts:

- **Audit trails:** How do you document what the AI generated when asking again might produce different code?
- **Debugging:** If code worked yesterday but fails today after regeneration, which version was correct?
- **Collaboration:** Two developers asking the same question may get different answers, leading to inconsistent codebases
- **Testing:** AI-assisted test generation may not produce the same tests twice, complicating coverage analysis

**Mitigation strategies:**
- Save important prompts and outputs for reference
- Use version control to track what was actually deployed
- Prefer lower temperature settings for production code
- Document decisions, not just the generated code
- Consider deterministic alternatives for critical paths

---

## Extended Thinking

Modern LLMs can engage in **extended thinking**—allocating more computational resources to reason through complex problems before responding. This is particularly valuable for coding tasks.

### How Extended Thinking Works

When you use thinking keywords (like "think" or "think harder" in Claude Code), the model:

1. **Allocates more reasoning steps** before generating output
2. **Explores multiple approaches** rather than committing to the first plausible answer
3. **Self-checks** for errors and inconsistencies
4. **Considers edge cases** that quick responses might miss

### The Thinking Budget Hierarchy

Different keywords allocate different "thinking budgets":

| Level | Keyword | Best For |
|-------|---------|----------|
| Standard | No keyword | Simple, well-defined tasks |
| Extended | "think" | Complex tasks requiring planning |
| More Extended | "think hard" | Difficult problems with tradeoffs |
| Even More | "think harder" | Very complex architectural decisions |
| Maximum | "ultrathink" | The hardest problems requiring deep analysis |

### Extended Thinking Budget Diagram

```
+-------------------------------------------------------------------------+
|                    THINKING BUDGET ALLOCATION                           |
+-------------------------------------------------------------------------+
|                                                                         |
|  KEYWORD         THINKING BUDGET                     BEST FOR           |
|                                                                         |
|  (none)          [██░░░░░░░░░░░░░░░░░░]  Standard    Simple functions,  |
|                                                      quick fixes        |
|                                                                         |
|  "think"         [████████░░░░░░░░░░░░]  Extended    Feature planning,  |
|                                                      debugging          |
|                                                                         |
|  "think hard"    [████████████░░░░░░░░]  More        Architecture,      |
|                                          Extended    tradeoffs          |
|                                                                         |
|  "think harder"  [████████████████░░░░]  Even More   Complex systems,   |
|                                                      multi-component    |
|                                                                         |
|  "ultrathink"    [████████████████████]  Maximum     Critical security, |
|                                                      hardest problems   |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  COST/BENEFIT:                                                          |
|                                                                         |
|  More thinking = Better reasoning BUT:                                  |
|    - Takes longer (sometimes significantly)                             |
|    - Uses more tokens (context budget)                                  |
|    - Diminishing returns for simple tasks                               |
|                                                                         |
|  RECOMMENDATION: Start with no keyword, escalate if needed              |
|                                                                         |
+-------------------------------------------------------------------------+
|                                                                         |
|  DECISION GUIDE:                                                        |
|                                                                         |
|  Is the task complex?                                                   |
|       |                                                                 |
|       +-- No --> Standard (no keyword)                                  |
|       |                                                                 |
|       +-- Yes --> Are there tradeoffs to analyze?                       |
|                         |                                               |
|                         +-- No --> "think"                              |
|                         |                                               |
|                         +-- Yes --> Is it architectural/system-wide?    |
|                                           |                             |
|                                           +-- No --> "think hard"       |
|                                           |                             |
|                                           +-- Yes --> Is it critical?   |
|                                                         |               |
|                                                         +-- No --> "think harder"
|                                                         |               |
|                                                         +-- Yes --> "ultrathink"
|                                                                         |
+-------------------------------------------------------------------------+
```

### When to Use Extended Thinking

**Use standard (no keyword) for:**
- Simple function implementation
- Well-documented API usage
- Straightforward refactoring

**Use "think" for:**
- Planning feature implementation
- Debugging complex issues
- Designing data structures

**Use "think hard" or higher for:**
- Architectural decisions affecting multiple components
- Security-sensitive implementations
- Performance optimization with complex tradeoffs
- Problems where initial attempts have failed

### Cost-Benefit Considerations

Extended thinking isn't free:
- **Time:** More thinking takes longer (sometimes significantly)
- **Tokens:** Extended thinking uses more of your context budget
- **Diminishing returns:** Not all problems benefit from deeper thinking

Use the minimum thinking level that solves your problem. Start without keywords; escalate if needed.

---

## Tool Use: How AI Agents Act

Beyond generating text, modern coding assistants can **use tools**—taking actions in your development environment. Understanding this architecture helps you work with agents effectively.

### The Action-Observation Loop

When an AI agent operates (like Claude Code running a command), it follows a loop:

1. **Think:** The model decides what action to take based on your request and context
2. **Act:** The model calls a tool (read file, run command, write file)
3. **Observe:** The tool returns results (file contents, command output, success/error)
4. **Repeat:** Based on observations, the model decides the next action

This continues until the task is complete or the model determines it cannot proceed.

### Types of Tool Actions

AI coding assistants typically have access to:

| Tool Category | Examples | Risk Level |
|---------------|----------|------------|
| **Read** | View files, list directories, search code | Low |
| **Execute** | Run tests, lint code, execute scripts | Medium |
| **Write** | Create files, edit code, modify configs | Medium-High |
| **System** | Git operations, package installation | High |
| **External** | API calls, web requests | Varies |

### Why Tool Use Architecture Matters

Understanding this architecture helps you:

- **Predict behavior:** Know what the agent might do with your request
- **Set appropriate boundaries:** Some tools should require confirmation
- **Diagnose failures:** Understand why an agent got stuck or produced wrong results
- **Write better prompts:** Guide the agent toward appropriate tool usage

### Preview: Agent Loops and Autonomy

This action-observation pattern is the foundation of **agentic AI systems**. In later chapters (particularly Chapter 09), we'll explore how agents:

- Chain multiple tool uses together
- Handle errors and retry strategies
- Maintain context across complex operations
- Operate with varying levels of human oversight

For now, understand that your AI coding assistant isn't just a text generator—it's a tool-using agent that can act in your environment.

---

## Why AI Code Can Be Vulnerable

### Training Data Contains Vulnerabilities

LLMs learn from public code repositories, which include:

- Code with security flaws
- Outdated patterns
- Quick hacks that work but aren't secure
- Copy-pasted vulnerable snippets

### Vulnerability Inheritance Diagram

```
+-------------------------------------------------------------------------+
|                    VULNERABILITY INHERITANCE                            |
+-------------------------------------------------------------------------+
|                                                                         |
|  TRAINING DATA (GitHub, Documentation, Tutorials, etc.)                 |
|  +------------------------------------------------------------------+  |
|  |                                                                  |  |
|  |  ████████████████████████████████████  Good patterns (secure,    |  |
|  |                                        well-structured code)     |  |
|  |                                                                  |  |
|  |  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  Mediocre patterns (works but not ideal,      |  |
|  |                    outdated practices)                           |  |
|  |                                                                  |  |
|  |  ░░░░░░░░░░░░░░░░  Vulnerable patterns (SQL injection,          |  |
|  |                    hardcoded secrets, etc.)  <-- 40%+ of code!   |  |
|  |                                                                  |  |
|  +------------------------------------------------------------------+  |
|                               |                                        |
|                               v                                        |
|                    +--------------------+                              |
|                    |    LLM TRAINING    |                              |
|                    |                    |                              |
|                    |  Model learns ALL  |                              |
|                    |  patterns equally  |                              |
|                    +--------------------+                              |
|                               |                                        |
|                               v                                        |
|                    +--------------------+                              |
|                    |  GENERATED CODE    |                              |
|                    |                    |                              |
|                    |  Output reflects   |                              |
|                    |  training mix      |                              |
|                    +--------------------+                              |
|                               |                                        |
|                               v                                        |
|  +------------------------------------------------------------------+  |
|  |  RESULT: AI-generated code may reproduce vulnerable patterns     |  |
|  |                                                                  |  |
|  |  CRITICAL: Newer/larger models don't fix this problem!           |  |
|  |            They've seen MORE vulnerable code, not less.          |  |
|  +------------------------------------------------------------------+  |
|                                                                         |
+-------------------------------------------------------------------------+
```

**Statistics:**
- NYU/Stanford studies found up to **40% of AI-assisted programs** have exploitable flaws [^nyu]
- Veracode 2025 reports **45% of AI-generated code** contains security vulnerabilities [^veracode]
- **Newer or larger models don't necessarily reduce** vulnerability rates—and may increase them due to exposure to more vulnerable code patterns [^stanford]

### Common Vulnerability Patterns

AI-generated code frequently exhibits:

| Vulnerability | Example |
|---------------|---------|
| SQL Injection | `query = f"SELECT * FROM users WHERE id = {user_id}"` |
| XSS (Cross-Site Scripting) | Rendering user input without escaping |
| Hardcoded Secrets | `api_key = "sk-abc123..."` |
| Insecure Deserialization | Using `pickle` instead of `json` |
| Missing Authentication | API endpoints without auth checks |
| Path Traversal | `open(f"/data/{filename}")` without validation |

### The Security Mindset

**Treat AI-generated code like code from an untrusted junior developer:**

1. It might work correctly
2. It might have subtle bugs
3. It might have security vulnerabilities
4. You must review it before trusting it

---

## Hallucinations in Code

### What Are Hallucinations?

LLMs can generate **plausible-sounding but incorrect** information. In code, this manifests as:

- **Fake APIs:** Functions that look real but don't exist
- **Wrong syntax:** Code that looks correct but won't compile
- **Incorrect logic:** Code that runs but produces wrong results
- **Nonexistent packages:** Import statements for libraries that don't exist

### Example: Hallucinated API

```python
# AI might generate:
from datetime import parse_date  # This doesn't exist!

date = parse_date("2025-01-15")
```

The correct code would be:
```python
from datetime import datetime

date = datetime.strptime("2025-01-15", "%Y-%m-%d")
```

### Why Hallucinations Happen

- The model predicts what "looks right" based on patterns
- It doesn't execute code or check documentation
- Confidence in generation doesn't correlate with accuracy
- Rare or complex APIs are more likely to be hallucinated

### Detecting Hallucinations

1. **Run the code:** Syntax errors and import failures catch many issues
2. **Check documentation:** Verify that APIs exist and work as described
3. **Test edge cases:** Hallucinated logic often fails on boundary conditions
4. **Be skeptical of unfamiliar libraries:** Research packages before installing

### Code Example: Verifying API Existence

Before trusting AI-generated code that uses unfamiliar APIs, verify they exist:

```python
# api_verification.py
# Tools for verifying that APIs mentioned in AI code actually exist

import importlib
import sys
from typing import Optional, List, Tuple

def verify_module_exists(module_name: str) -> Tuple[bool, str]:
    """
    Check if a Python module can be imported.

    Returns:
        (exists: bool, message: str)
    """
    try:
        importlib.import_module(module_name)
        return True, f"Module '{module_name}' exists and can be imported"
    except ImportError as e:
        return False, f"Module '{module_name}' not found: {e}"


def verify_attribute_exists(module_name: str, attribute: str) -> Tuple[bool, str]:
    """
    Check if a specific function/class exists in a module.

    Example:
        verify_attribute_exists('datetime', 'parse_date')
        # Returns: (False, "Module 'datetime' has no attribute 'parse_date'")
    """
    try:
        module = importlib.import_module(module_name)
        if hasattr(module, attribute):
            attr = getattr(module, attribute)
            attr_type = type(attr).__name__
            return True, f"'{module_name}.{attribute}' exists (type: {attr_type})"
        else:
            # Show what's actually available
            available = [a for a in dir(module) if not a.startswith('_')]
            similar = [a for a in available if attribute.lower() in a.lower()]
            msg = f"Module '{module_name}' has no attribute '{attribute}'"
            if similar:
                msg += f". Did you mean: {', '.join(similar[:5])}?"
            return False, msg
    except ImportError as e:
        return False, f"Cannot import module '{module_name}': {e}"


def verify_ai_code_imports(code: str) -> List[Tuple[str, bool, str]]:
    """
    Extract imports from code and verify they exist.

    This catches hallucinated modules and APIs.
    """
    import re

    results = []

    # Pattern for "from module import name" or "from module import name as alias"
    from_imports = re.findall(r'from\s+([\w.]+)\s+import\s+([\w,\s]+)', code)
    for module, items in from_imports:
        for item in items.split(','):
            item = item.strip().split(' as ')[0].strip()  # Remove "as alias"
            if item:
                exists, msg = verify_attribute_exists(module, item)
                results.append((f"from {module} import {item}", exists, msg))

    # Pattern for "import module" or "import module as alias"
    direct_imports = re.findall(r'^import\s+([\w.]+)', code, re.MULTILINE)
    for module in direct_imports:
        module = module.split(' as ')[0].strip()
        exists, msg = verify_module_exists(module)
        results.append((f"import {module}", exists, msg))

    return results


# Example usage - testing AI-generated code for hallucinations

print("=" * 60)
print("VERIFYING AI-GENERATED CODE IMPORTS")
print("=" * 60)

# This code contains a hallucinated import (parse_date doesn't exist)
ai_generated_code = '''
from datetime import datetime, parse_date
from collections import defaultdict
from pathlib import Path
import requests
import nonexistent_module

date = parse_date("2025-01-15")
'''

results = verify_ai_code_imports(ai_generated_code)

print("\nVerification Results:")
print("-" * 60)

for import_stmt, exists, message in results:
    status = "OK" if exists else "FAIL"
    print(f"[{status:4}] {import_stmt}")
    if not exists:
        print(f"       {message}")

# Output:
# [FAIL] from datetime import parse_date
#        Module 'datetime' has no attribute 'parse_date'. Did you mean: datetime?
# [OK  ] from datetime import datetime
# [OK  ] from collections import defaultdict
# [OK  ] from pathlib import Path
# [OK  ] import requests
# [FAIL] import nonexistent_module
#        Module 'nonexistent_module' not found: No module named 'nonexistent_module'
```

**Quick verification commands in the terminal:**

```bash
# Check if a module exists and see what it exports
python -c "import datetime; print([x for x in dir(datetime) if not x.startswith('_')])"

# Check if a specific attribute exists
python -c "from datetime import parse_date"  # Will error if doesn't exist

# Search for similar functions
python -c "import datetime; print([x for x in dir(datetime) if 'parse' in x.lower()])"
```

**Red flags that suggest hallucinated APIs:**

1. Functions with suspiciously convenient names (`parse_date`, `auto_format`, `easy_auth`)
2. Single imports from complex modules (`from tensorflow import train_model`)
3. Modules you've never heard of for common tasks
4. Functions that seem "too good to be true" for the standard library

When in doubt, check the official documentation before running AI-generated code.

---

## Model Comparison

Different models have different strengths for coding tasks:

### Claude (Anthropic)

- **Strengths:** Strong reasoning, follows complex instructions, good at explaining code
- **Context:** 200K tokens standard
- **Best for:** Complex refactoring, code explanation, architectural discussions

### GPT-4 (OpenAI)

- **Strengths:** Broad knowledge, good at common patterns, extensive tool integrations
- **Context:** 128K tokens
- **Best for:** General coding tasks, well-documented libraries

### Gemini (Google)

- **Strengths:** Very long context (1M+ tokens), good at multi-file analysis
- **Context:** 1,000,000+ tokens
- **Best for:** Large codebase analysis, comprehensive code review

### Choosing a Model

| Task | Recommended |
|------|-------------|
| Single function generation | Any model works well |
| Multi-file refactoring | Claude or Gemini (long context) |
| Learning new concepts | Any model with explanation emphasis |
| Security-sensitive code | Use any, but always manually review |
| Complex architecture | Claude (strong reasoning) |

---

## Key Takeaways

1. **LLMs predict tokens** based on patterns from training data—they don't "understand" code
2. **Instruction tuning and RLHF** transform base models into helpful assistants that follow your requirements
3. **Context windows limit** how much the model can see; longer conversations may suffer from context rot
4. **Non-determinism means** the same prompt can produce different results—reproducibility requires deliberate practices
5. **Extended thinking** lets you allocate more reasoning to complex problems, but use it judiciously
6. **Tool use architecture** enables agents to act in your environment through an action-observation loop
7. **Training data contains vulnerabilities**, so AI code inherits these patterns
8. **Hallucinations are real**—verify APIs, libraries, and logic before trusting AI output
9. **Treat AI output as untrusted input** that requires review and verification

---

## Practical Exercise

Explore hallucinations and non-determinism:

1. Ask an AI to "Write a function using the Python `quicksort` standard library function"
   - (There is no such function—see what the AI generates)

2. Ask the same prompt 3 times and compare the responses:
   "Write a function to reverse a string in Python"
   - Note differences in approach, naming, and implementation

3. Ask for code using a very specific, niche library:
   "Write code using the `obscure-library-12345` package"
   - Observe how the AI hallucinates a plausible but fake API

These exercises build awareness of AI limitations.

---

## Next Step

[Step 03: Your First AI-Assisted Coding Session](03-first-ai-coding-session.md) — Set up tools and write your first AI-assisted code.

---

## Sources

[^sase]: Guo, P. (2025). "Structured Agentic Software Engineering." arXiv:2509.06216v2. https://arxiv.org/abs/2509.06216

[^nyu]: Pearce, H., et al. (2022). "Asleep at the Keyboard? Assessing the Security of GitHub Copilot's Code Contributions." *IEEE S&P 2022*. NYU Tandon School of Engineering.

[^stanford]: Perry, N., et al. (2023). "Do Users Write More Insecure Code with AI Assistants?" *Stanford University Security Lab*. https://arxiv.org/abs/2211.03622

[^veracode]: Veracode. (2025). *State of Software Security 2025*. Annual Report. https://www.veracode.com/state-of-software-security-report

- Stanford CS146S, Week 1: Introduction to Coding LLMs. Stanford University, 2025.
- Anthropic Documentation: Context Windows and Token Limits. https://docs.anthropic.com/
- Breunig, D. (2024). "How Long Contexts Fail." Research notes on attention degradation in long context windows.

---

## Further Reading

1. **Anthropic: Claude's Character** — Understanding how Claude is designed to be helpful, harmless, and honest. https://www.anthropic.com/research/claudes-character

2. **"Attention Is All You Need" (Vaswani et al., 2017)** — The foundational transformer paper that underlies all modern LLMs. arXiv:1706.03762

3. **OpenAI: Tokenizer Documentation** — Interactive tokenizer for understanding how text becomes tokens. https://platform.openai.com/tokenizer

4. **Drew Breunig: How Long Contexts Fail** — Analysis of attention degradation and context rot in long-running AI sessions. https://www.dbreunig.com/2024/03/14/how-long-contexts-fail.html

5. **NYU Security Studies on AI Code** — Academic research on security vulnerabilities in AI-generated code. https://arxiv.org/abs/2108.09293
